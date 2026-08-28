# 🪟 عمليات Git الموقّعة بـ GPG على Windows

يستخدم هذا المسار Gpg4win وKleopatra. يوفّر Git for Windows برنامج Git Bash، بينما PowerShell وCommand Prompt لهما صياغة مسارات مختلفة.

## التحضير والأمان

ثبّت [Gpg4win](https://www.gpg4win.org/download.html) و[Git for Windows](https://git-scm.com/download/win)، وجهّز بريداً مؤكّداً في GitHub.

> ⚠️ ارفع المفتاح العام فقط. لا تشارك المفتاح الخاص/السري أو عبارة المرور أو شهادة الإلغاء.

## 1. التثبيت ومعرفة المسارات

في PowerShell أو Command Prompt أو Git Bash:

~~~text
gpg --version
~~~

في PowerShell:

~~~powershell
Get-Command gpg -All
Get-Command git -All
~~~

في Command Prompt أو Git Bash:

~~~text
where gpg
where git
~~~

استخدم `gpg.exe` من تثبيت Gpg4win نفسه الذي يستخدمه Kleopatra. لا تستخدم `/usr/local/bin/gpg` ولا تنسخ إعداد `GPG_TTY` الخاص بـmacOS.

## 2. إنشاء المفتاح

في Kleopatra اختر **File → New Certificate → OpenPGP**، أو نفّذ:

~~~text
gpg --full-generate-key
~~~

اختر RSA 4096 للتوقيع إن توفر، وتاريخ انتهاء قابلاً للتجديد، وعبارة مرور قوية، و`YOUR_NAME` و`YOUR_VERIFIED_GITHUB_EMAIL`.

## 3. العثور على البصمة

~~~text
gpg --list-secret-keys --keyid-format=long
~~~

انسخ البصمة الكاملة تحت `sec` باسم `YOUR_GPG_FINGERPRINT`.

## 4. تصدير المفتاح العام

~~~text
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

انسخ الكتلة بين `-----BEGIN PGP PUBLIC KEY BLOCK-----` و`-----END PGP PUBLIC KEY BLOCK-----`. هذه عامة؛ لا تشارك مادة سرية.

## 5. إضافته إلى GitHub

اذهب إلى **Settings → Access → SSH and GPG keys → New GPG key**، ألصق المفتاح العام واختر **Add GPG key**. راجع [التعليمات الرسمية](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 6. ضبط الهوية العامة

~~~text
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

يطبّق `--global` الإعداد على مستودعات مستخدم Windows كلها ما لم يوجد تجاوز محلي.

## 7. توجيه Git إلى GPG الصحيح

في PowerShell أو Command Prompt، استبدل المسار بمسار `Get-Command gpg` أو `where gpg`:

~~~powershell
git config --global gpg.program "C:/Program Files/GnuPG/bin/gpg.exe"
~~~

يقبل Git for Windows هذا المسار بين علامتي اقتباس. ثم:

~~~text
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
git config --global --get gpg.format
~~~

إذا ظهرت `ssh` وتريد GPG:

~~~text
git config --global --unset gpg.format
~~~

## 8. فحص التجاوزات المحلية

~~~text
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
git config --show-origin --get gpg.format
~~~

تتغلب `file:.git/config` على الإعداد العام. داخل المستودع:

~~~text
git config --unset user.name
git config --unset user.email
~~~

## 9. اختبار التوقيع

~~~text
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

سيطلب Kleopatra أو pinentry عبارة المرور، وينبغي رؤية `gpg: Good signature from ...`. نفّذ `git push` وتحقق من `Verified` في GitHub.

## 🛠️ استكشاف الأخطاء

- **عدة gpg.exe:** قارن `Get-Command gpg -All` أو `where gpg` مع `gpg.program` واستخدم تثبيت Gpg4win نفسه الذي يستخدمه Kleopatra.
- **لا تظهر نافذة pinentry:** افتح Kleopatra ثم نفّذ `gpgconf --kill gpg-agent`. لا يحتاج Windows إلى `GPG_TTY`.
- **مشكلات IDE:** اضبط IDE على Git وGPG نفسيهما اللذين يعملان في الطرفية.
- **لا يظهر Verified:** افحص المفتاح العام و`user.email` والبريد المؤكد و`gpg.format` و`git log --show-signature -1`.
- **مفتاح منتهٍ أو مفقود أو ملغى:** جدده أو أنشئ مفتاحاً جديداً. لا يعيد المفتاح العام المفتاح الخاص؛ الإلغاء يعلن فقدان الثقة فقط.

## 🔐 النسخ الاحتياطي والمستقبل

استخدم Kleopatra أو GnuPG لحفظ نسخة مشفرة وغير متصلة من المفتاح الخاص وشهادة الإلغاء. شارك المفتاح العام فقط.

🌿 [GitHub](https://github.com/syllik) · [syllik@gmail.com](mailto:syllik@gmail.com)

<details>
<summary>✨ ملاحظة للمستقبل</summary>

ستجد هذا الدليل مرة أخرى يوماً ما عندما ترغب في تعلم شيء جديد.

إنه مجاني.

مع المحبة،<br>
fireflў
</details>

---

[← الصفحة العربية](README.md) · [🌍 جميع اللغات](../../README.md)
