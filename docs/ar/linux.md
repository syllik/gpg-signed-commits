# 🐧 عمليات Git الموقّعة بـ GPG على Linux

اضبط توقيع OpenPGP/GPG بشكل عام لمستخدم Linux. تختلف أوامر التثبيت حسب التوزيعة.

## التحضير والأمان

ستحتاج إلى Git وحساب GitHub وبريد إلكتروني مؤكّد. استخدم البريد نفسه في GPG و`user.email`. يمكن مشاركة المفتاح العام، أما المفتاح الخاص/السري وعبارة المرور وشهادة الإلغاء فتبقى سرية.

## 1. تثبيت GnuPG

Debian/Ubuntu:

~~~bash
sudo apt update
sudo apt install gnupg
~~~

Fedora:

~~~bash
sudo dnf install gnupg2
~~~

Arch Linux:

~~~bash
sudo pacman -Syu gnupg
~~~

للتوزيعات الأخرى اتبع توثيق الحزم الرسمي الحالي وثبّت GnuPG 2 وpinentry مناسباً للطرفية أو لسطح المكتب. تحقق:

~~~bash
gpg --version
which gpg
~~~

إذا كان النظام يستخدم `gpg2` فاستخدمه باستمرار واضبط `gpg.program` عليه.

## 2. إعداد pinentry والوكيل

يبدأ `gpg-agent` عادة عند الحاجة. اختر pinentry طرفياً أو رسوميّاً وفق بيئتك؛ لا تفترض GNOME أو KDE أو Wayland أو X11 أو systemd.

~~~bash
which pinentry
which pinentry-curses
which pinentry-gtk-2
which pinentry-qt
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

إذا اخترت برنامجاً محدداً، أضف مساره الفعلي إلى `~/.gnupg/gpg-agent.conf`:

~~~text
pinentry-program /THE/ACTUAL/PATH/TO/PINENTRY
~~~

حافظ على الإعدادات الموجودة ثم:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

أضف `export GPG_TTY=$(tty)` إلى `~/.bashrc` أو `~/.zshrc` بحسب shell الذي تستخدمه، ثم افتح shell جديداً.

## 3. إنشاء المفتاح

~~~bash
gpg --full-generate-key
~~~

اختر RSA 4096 قابلاً للتوقيع إن توفر، وتاريخ انتهاء قابلاً للتجديد، وعبارة مرور قوية، و`YOUR_NAME` و`YOUR_VERIFIED_GITHUB_EMAIL`. يجب أن يستطيع GitHub مطابقة بريد الـcommitter مع UID والبريد المؤكّد.

## 4. العثور على البصمة

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

انسخ البصمة الكاملة تحت `sec` باسم `YOUR_GPG_FINGERPRINT`.

## 5. تصدير المفتاح العام

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

انسخ الكتلة العامة بين `-----BEGIN PGP PUBLIC KEY BLOCK-----` و`-----END PGP PUBLIC KEY BLOCK-----`. لا تشارك المادة السرية.

## 6. إضافته إلى GitHub

افتح **Settings → Access → SSH and GPG keys → New GPG key**، ألصق المفتاح العام وأضفه. راجع [وثائق GitHub](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 7. الهوية العامة

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

ينطبق `--global` على مستودعات المستخدم كلها ما لم يوجد تجاوز محلي.

## 8. توقيع OpenPGP العام

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

افحص التنسيق:

~~~bash
git config --global --get gpg.format
~~~

إذا كان `ssh` وتريد OpenPGP:

~~~bash
git config --global --unset gpg.format
~~~

## 9. فحص الإعدادات المحلية

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

لـ`file:.git/config` أولوية أعلى. احذف الهوية المحلية:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. الاختبار

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

ابحث عن `gpg: Good signature from ...`، نفّذ `git push` ثم تحقق من ظهور `Verified` على GitHub. التحقق المحلي وربط الحساب ليسا اختباراً واحداً.

## 🛠️ استكشاف الأخطاء

- **pinentry أو الوكيل:** تحقق من البرنامج و`pinentry-program` و`GPG_TTY` وصلاحيات `~/.gnupg`، ثم شغّل `gpgconf --kill gpg-agent`.
- **تعدد تثبيتات GPG:** قارن `which gpg` و`which gpg2` و`git config --show-origin --get gpg.program`.
- **الطرفية وGUI وIDE:** قارن PATH وGit وGPG التي يستخدمها IDE.
- **غياب Verified:** افحص المفتاح العام و`user.email` والبريد المؤكد و`gpg.format` وانتهاء المفتاح.
- **فقدان المفتاح أو إلغاؤه:** لا يعيد المفتاح العام إنشاء الخاص. استعد نسخة احتياطية أو أنشئ مفتاحاً جديداً؛ الإلغاء يعلن فقدان الثقة فقط.

## 🔐 النسخ الاحتياطي والمستقبل

احفظ المفتاح الخاص وشهادة الإلغاء مشفرين وغير متصلين. شارك المفتاح العام فقط.

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume) · [Telegram — @syllik](https://t.me/syllik)

<details>
<summary>✨ ملاحظة للمستقبل</summary>

ستجد هذا الدليل مرة أخرى يوماً ما عندما ترغب في تعلم شيء جديد.

إنه مجاني.

مع المحبة،<br>
fireflў
</details>

---

[← الصفحة العربية](README.md) · [🌍 جميع اللغات](../../README.md)
