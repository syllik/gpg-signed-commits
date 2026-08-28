# 🍎 عمليات Git الموقّعة بـ GPG على macOS

اضبط توقيعات OpenPGP/GPG لجميع مستودعات Git المحلية الخاصة بالمستخدم الحالي على macOS.

## قبل البدء والأمان

ستحتاج إلى Git وHomebrew وحساب GitHub وعنوان بريد إلكتروني مضاف ومؤكّد في GitHub. استخدم العنوان نفسه في UID الخاص بـ GPG وفي `user.email` في Git.

> ⚠️ يمكن رفع **المفتاح العام** إلى GitHub. لا ترفع المفتاح الخاص/السري أو عبارة المرور أو محتوى شهادة الإلغاء ولا تشاركها.

## 1. تثبيت GnuPG والتحقق منه

قد يختلف مسار Homebrew بين أجهزة Mac. ثبّت GnuPG وpinentry الخاص بـ macOS ثم اكتشف المسارات الفعلية:

~~~bash
brew install gnupg pinentry-mac
gpg --version
which gpg
which pinentry-mac
~~~

استخدم نتيجة `which`؛ لا تفترض مسبقاً `/usr/local` أو `/opt/homebrew`.

## 2. إعداد pinentry وgpg-agent

يدير `gpg-agent` المفاتيح السرية ويستدعي pinentry لطلب عبارة المرور. يوجد الإعداد في `~/.gnupg/gpg-agent.conf`.

~~~bash
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

إذا لم يوجد الملف، أضف إليه المسار الذي أظهره `which pinentry-mac`:

~~~text
pinentry-program /THE/ACTUAL/PATH/FROM/which-pinentry-mac
~~~

إذا كان الملف موجوداً، أضف أو عدّل هذا السطر فقط وحافظ على بقية الإعدادات. أعد تشغيل الوكيل واضبط الطرفية:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

في zsh، أضف `export GPG_TTY=$(tty)` إلى `~/.zshrc` ثم نفّذ `source ~/.zshrc`. يساعد ذلك pinentry الطرفي على الوصول إلى TTY الحالي.

## 3. إنشاء المفتاح

~~~bash
gpg --full-generate-key
~~~

اختر RSA بحجم 4096 وقابلاً للتوقيع إن ظهر الخيار، وحدد تاريخ انتهاء يمكنك تجديده، وعبارة مرور قوية، ثم أدخل `YOUR_NAME` و`YOUR_VERIFIED_GITHUB_EMAIL`. يطابق GitHub بريد الـcommitter مع UID المفتاح وبريد مؤكّد في الحساب؛ لا يحدد UID وحده هوية Git.

## 4. العثور على البصمة الكاملة

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

انسخ البصمة الكاملة تحت السطر `sec` واستخدمها باسم `YOUR_GPG_FINGERPRINT`. يلزم المفتاح الخاص لإنشاء التوقيع.

## 5. تصدير المفتاح العام فقط

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

انسخ الكتلة من `-----BEGIN PGP PUBLIC KEY BLOCK-----` إلى `-----END PGP PUBLIC KEY BLOCK-----`. هذه كتلة عامة؛ لا تصدّر أو تشارك أي مادة سرية.

## 6. إضافة المفتاح إلى GitHub

في GitHub افتح **Settings → Access → SSH and GPG keys → New GPG key**، اكتب عنواناً، ألصق المفتاح العام ثم اختر **Add GPG key**. راجع [تعليمات GitHub الرسمية](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) إذا تغيرت الواجهة.

## 7. ضبط هوية Git العامة

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

يطبّق `--global` الإعداد على مستودعات هذا المستخدم كلها، ما لم يوجد إعداد محلي يتغلب عليه.

## 8. تفعيل توقيع OpenPGP بشكل عام

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

تحقق من وجود تنسيق سابق:

~~~bash
git config --global --get gpg.format
~~~

إذا كانت القيمة `ssh` وتريد استخدام GPG، احذف التجاوز للعودة إلى `openpgp`:

~~~bash
git config --global --unset gpg.format
~~~

## 9. فحص التجاوزات المحلية

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

تتغلب القيم الموجودة في `file:.git/config` على الإعداد العام. داخل المستودع احذف الهوية المحلية غير المرغوبة من دون `--global`:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. اختبار عملية موقّعة

في مستودع اختبار:

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

ابحث عن رسالة مثل `gpg: Good signature from ...`. نفّذ `git push` ثم افتح العملية في GitHub؛ ينبغي أن يظهر `Verified`. التحقق المحلي والتأكد من ربط المفتاح والبريد بحساب GitHub عمليتان مرتبطتان لكنهما ليستا الشيء نفسه.

## 🛠️ استكشاف الأخطاء

- **No pinentry:** عند ظهور `gpg: agent_genkey failed: No pinentry` و`Key generation failed: No pinentry` ثبّت `pinentry-mac`، اعثر عليه بواسطة `which pinentry-mac`، أضف `pinentry-program` إلى الملف الموجود، ثم أعد تشغيل الوكيل.
- **صلاحيات Homebrew:** غيّر الدليل المحدد في رسالة الخطأ فقط باستخدام `sudo chown -R "$(whoami)" "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"` ثم `chmod u+w "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"`. لا تطبق chown على كامل `/usr/local` أو `/opt/homebrew`.
- **أكثر من GPG أو مشكلة IDE:** قارن `which gpg` و`gpg.program` ومسارات Git/GPG في IDE؛ يجب أن تستخدم جميعها keyring الذي يحوي المفتاح الخاص.
- **لا يظهر Verified:** تحقق من المفتاح العام و`user.email` والبريد المؤكّد و`gpg.format` ونتيجة `git log --show-signature -1`. جدّد المفتاح المنتهي.
- **فقدان المفتاح:** لا يمكن للمفتاح العام في GitHub إعادة بناء المفتاح الخاص. استعد نسخة احتياطية آمنة أو أنشئ مفتاحاً جديداً؛ شهادة الإلغاء تعلن فقط أن المفتاح لم يعد موثوقاً.

## 🔐 النسخ الاحتياطي والمستقبل

احتفظ بنسخة مشفرة وغير متصلة من المفتاح الخاص وشهادة الإلغاء. شارك المفتاح العام فقط.

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume)

<details>
<summary>✨ ملاحظة للمستقبل</summary>

ستجد هذا الدليل مرة أخرى يوماً ما عندما ترغب في تعلم شيء جديد.

إنه مجاني.

مع المحبة،<br>
fireflў
</details>

---

[← الصفحة العربية](README.md) · [🌍 جميع اللغات](../../README.md)
