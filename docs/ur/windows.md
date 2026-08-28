# 🪟 Windows پر GPG-signed Git commits

یہ Windows-native workflow Gpg4win اور Kleopatra استعمال کرتا ہے۔ Git for Windows میں Git Bash شامل ہے؛ PowerShell اور Command Prompt الگ shells ہیں۔

## تیاری اور سکیورٹی

[Gpg4win](https://www.gpg4win.org/download.html) اور [Git for Windows](https://git-scm.com/download/win) install کریں۔ GitHub پر verified email رکھیں۔

> ⚠️ صرف public key upload کریں۔ Private/secret key، passphrase اور revocation certificate share نہ کریں۔

## 1. GPG install اور locate کریں

PowerShell، Command Prompt یا Git Bash میں:

~~~text
gpg --version
~~~

PowerShell:

~~~powershell
Get-Command gpg -All
Get-Command git -All
~~~

Command Prompt یا Git Bash:

~~~text
where gpg
where git
~~~

Kleopatra جس Gpg4win installation کا keyring استعمال کرتا ہے، اسی کا `gpg.exe` استعمال کریں۔ Windows میں `/usr/local/bin/gpg` یا macOS کا `GPG_TTY` استعمال نہ کریں۔

## 2. key بنائیں

Kleopatra میں **File → New Certificate → OpenPGP** منتخب کریں، یا shell میں:

~~~text
gpg --full-generate-key
~~~

Signing-capable RSA 4096، renew ہونے والی expiry، مضبوط passphrase، `YOUR_NAME` اور `YOUR_VERIFIED_GITHUB_EMAIL` درج کریں۔

## 3. fingerprint

~~~text
gpg --list-secret-keys --keyid-format=long
~~~

`sec` کے نیچے مکمل fingerprint copy کر کے `YOUR_GPG_FINGERPRINT` رکھیں۔

## 4. public key export

~~~text
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

`-----BEGIN PGP PUBLIC KEY BLOCK-----` سے `-----END PGP PUBLIC KEY BLOCK-----` تک copy کریں۔ یہ public ہے؛ secret material share نہ کریں۔

## 5. GitHub

**Settings → Access → SSH and GPG keys → New GPG key** میں public key paste کریں اور **Add GPG key** منتخب کریں۔ [Official GitHub guide](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) دیکھیں۔

## 6. Global identity

~~~text
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` local override نہ ہونے پر تمام repositories پر لاگو ہے۔

## 7. Git کو درست GPG دیں

PowerShell یا Command Prompt میں example path کو `Get-Command gpg` یا `where gpg` کے اصل path سے بدلیں:

~~~powershell
git config --global gpg.program "C:/Program Files/GnuPG/bin/gpg.exe"
~~~

پھر:

~~~text
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
git config --global --get gpg.format
~~~

اگر value `ssh` ہو اور GPG چاہیے:

~~~text
git config --global --unset gpg.format
~~~

## 8. Local overrides

~~~text
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` global setting کو override کرتا ہے۔ Repository کے اندر:

~~~text
git config --unset user.name
git config --unset user.email
~~~

## 9. Signed commit test

~~~text
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Kleopatra یا pinentry passphrase مانگے گا اور `gpg: Good signature from ...` دکھنا چاہیے۔ `git push` کے بعد GitHub پر `Verified` دیکھیں۔

## 🛠️ مسائل کا حل

- **Multiple gpg.exe:** `Get-Command gpg -All`/`where gpg` کو `gpg.program` سے compare کریں اور Kleopatra والی installation استعمال کریں۔
- **pinentry window نہیں:** Kleopatra کھولیں اور `gpgconf --kill gpg-agent` چلائیں۔ Windows کو `GPG_TTY` کی ضرورت نہیں۔
- **IDE:** وہی Git اور GPG منتخب کریں جو terminal میں کام کرتے ہیں۔
- **Verified نہیں:** public key، `user.email`، verified email، `gpg.format` اور `git log --show-signature -1` چیک کریں۔
- **Expired/lost/revoked key:** renew یا نئی key بنائیں۔ Public key private key بحال نہیں کرتی؛ revocation صرف trust ختم کرنے کا اعلان ہے۔

## 🔐 Backup اور مستقبل

Kleopatra یا GnuPG سے private key اور revocation certificate کا encrypted offline backup رکھیں۔ صرف public key share کریں۔

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume) · [Telegram — @syllik](https://t.me/syllik)

<details>
<summary>✨ مستقبل کے لیے ایک نوٹ</summary>

ایک دن، جب آپ کچھ نیا سیکھنا چاہیں گے، تو آپ کو یہ دوبارہ مل جائے گا۔

یہ مفت ہے۔

محبت کے ساتھ،<br>
fireflў
</details>

---

[← اردو home](README.md) · [🌍 تمام زبانیں](../../README.md)
