# 🐧 Linux پر GPG-signed Git commits

Linux user کے لیے global OpenPGP/GPG signing configure کریں۔ Installation command distribution کے مطابق بدلتی ہے۔

## تیاری اور سکیورٹی

GitHub account، verified email اور Git چاہیے۔ یہی email GPG اور `user.email` میں رکھیں۔ Public key share کی جا سکتی ہے؛ private/secret key، passphrase اور revocation certificate خفیہ رہیں۔

## 1. GnuPG install کریں

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

دوسری distribution کے لیے اس کی current official package documentation دیکھیں اور GnuPG 2 کے ساتھ مناسب pinentry install کریں۔ چیک کریں:

~~~bash
gpg --version
which gpg
~~~

اگر system `gpg2` استعمال کرتا ہے تو اسی executable کو مسلسل استعمال کریں اور `gpg.program` میں set کریں۔

## 2. pinentry اور agent configure کریں

`gpg-agent` ضرورت کے وقت خود start ہوتا ہے۔ اپنے ماحول کے مطابق terminal/curses یا GUI pinentry منتخب کریں؛ GNOME، KDE، Wayland، X11 یا systemd فرض نہ کریں۔

~~~bash
which pinentry
which pinentry-curses
which pinentry-gtk-2
which pinentry-qt
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

مخصوص program کے لیے اس کا اصل path `~/.gnupg/gpg-agent.conf` میں شامل کریں:

~~~text
pinentry-program /THE/ACTUAL/PATH/TO/PINENTRY
~~~

پہلے سے موجود lines محفوظ رکھیں، پھر:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

`export GPG_TTY=$(tty)` کو اپنے shell کے `~/.bashrc` یا `~/.zshrc` میں شامل کر کے نیا shell کھولیں۔

## 3. key بنائیں

~~~bash
gpg --full-generate-key
~~~

Signing-capable RSA 4096، renew ہونے والی expiry، مضبوط passphrase، `YOUR_NAME` اور `YOUR_VERIFIED_GITHUB_EMAIL` درج کریں۔ GitHub committer email، GPG UID اور verified email کا تعلق چیک کرتا ہے۔

## 4. fingerprint

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

`sec` کے نیچے مکمل fingerprint copy کر کے `YOUR_GPG_FINGERPRINT` بنائیں۔

## 5. public key export

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

`-----BEGIN PGP PUBLIC KEY BLOCK-----` سے `-----END PGP PUBLIC KEY BLOCK-----` تک public block copy کریں۔ Secret material share نہ کریں۔

## 6. GitHub میں شامل کریں

**Settings → Access → SSH and GPG keys → New GPG key** کھولیں، public key paste کریں اور add کریں۔ [GitHub documentation](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) دیکھیں۔

## 7. Global identity

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` local override نہ ہونے پر اس user کی تمام repositories پر لاگو ہے۔

## 8. Global signing

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Format دیکھیں:

~~~bash
git config --global --get gpg.format
~~~

اگر `ssh` ہو اور OpenPGP چاہیے:

~~~bash
git config --global --unset gpg.format
~~~

## 9. Local override

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` زیادہ priority رکھتا ہے۔ Local identity ہٹائیں:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Test اور verification

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

`gpg: Good signature from ...` تلاش کریں، `git push` کریں اور GitHub پر `Verified` دیکھیں۔ Local verification اور GitHub association ایک ہی check نہیں ہیں۔

## 🛠️ مسائل کا حل

- **pinentry/agent:** program، `pinentry-program`، `GPG_TTY` اور `~/.gnupg` permissions چیک کریں، پھر `gpgconf --kill gpg-agent` چلائیں۔
- **Multiple GPG:** `which gpg`، `which gpg2` اور `git config --show-origin --get gpg.program` کا موازنہ کریں۔
- **Terminal/GUI/IDE:** IDE کا PATH، Git اور GPG terminal کے ساتھ match کریں۔
- **Verified نہیں:** public key، `user.email`، verified email، `gpg.format` اور key expiry چیک کریں۔
- **Key ضائع یا revoke:** Public key private key بحال نہیں کرتی۔ Backup restore کریں یا نئی key بنائیں؛ revocation صرف trust ختم کرنے کا اعلان ہے۔

## 🔐 Backup اور مستقبل

Private key اور revocation certificate کا encrypted offline backup رکھیں۔ صرف public key share کریں۔

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume)

<details>
<summary>✨ مستقبل کے لیے ایک نوٹ</summary>

ایک دن، جب آپ کچھ نیا سیکھنا چاہیں گے، تو آپ کو یہ دوبارہ مل جائے گا۔

یہ مفت ہے۔

محبت کے ساتھ،<br>
fireflў
</details>

---

[← اردو home](README.md) · [🌍 تمام زبانیں](../../README.md)
