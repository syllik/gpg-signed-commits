# 🍎 macOS پر GPG-signed Git commits

موجودہ macOS user کی تمام local Git repositories کے لیے OpenPGP/GPG signing configure کریں۔

## شروع کرنے سے پہلے

آپ کو Git، Homebrew، GitHub account اور GitHub پر شامل و verified email address چاہیے۔ یہی email GPG UID اور Git کے `user.email` دونوں میں استعمال کریں۔

> ⚠️ **Public key** GitHub پر upload کی جا سکتی ہے۔ Private/secret key، passphrase اور revocation certificate کا content کبھی share نہ کریں۔

## 1. GnuPG install اور verify کریں

Homebrew prefix Mac کے مطابق مختلف ہو سکتا ہے۔ GnuPG اور macOS pinentry install کریں، پھر اصل paths معلوم کریں:

~~~bash
brew install gnupg pinentry-mac
gpg --version
which gpg
which pinentry-mac
~~~

`which` کا output استعمال کریں؛ `/usr/local` یا `/opt/homebrew` کا اندازہ نہ لگائیں۔

## 2. pinentry اور gpg-agent configure کریں

`gpg-agent` secret keys manage کرتا ہے اور passphrase کے لیے pinentry چلاتا ہے۔ اس کی configuration file `~/.gnupg/gpg-agent.conf` ہے۔

~~~bash
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

اگر file موجود نہیں تو `which pinentry-mac` سے ملنے والے اصل path کے ساتھ یہ line شامل کریں:

~~~text
pinentry-program /THE/ACTUAL/PATH/FROM/which-pinentry-mac
~~~

اگر file پہلے سے موجود ہے تو صرف یہی setting edit کریں اور دوسری lines محفوظ رکھیں۔ Agent restart کریں:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

zsh کے لیے `export GPG_TTY=$(tty)` کو `~/.zshrc` میں شامل کر کے `source ~/.zshrc` چلائیں۔ اس سے terminal کو موجودہ TTY معلوم ہوتا ہے۔

## 3. key بنائیں

~~~bash
gpg --full-generate-key
~~~

اگر option موجود ہو تو signing-capable RSA 4096 منتخب کریں، ایسی expiry رکھیں جسے renew کر سکیں، مضبوط passphrase، `YOUR_NAME` اور `YOUR_VERIFIED_GITHUB_EMAIL` درج کریں۔ GitHub committer email کو key UID اور account کے verified email سے ملاتا ہے؛ UID اکیلا Git identity نہیں بناتا۔

## 4. مکمل fingerprint تلاش کریں

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

`sec` record کے نیچے مکمل fingerprint copy کر کے `YOUR_GPG_FINGERPRINT` کے طور پر استعمال کریں۔ Sign کرنے کے لیے private key ضروری ہے۔

## 5. صرف public key export کریں

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

`-----BEGIN PGP PUBLIC KEY BLOCK-----` سے `-----END PGP PUBLIC KEY BLOCK-----` تک پورا block copy کریں۔ یہ public key ہے؛ secret material کبھی share نہ کریں۔

## 6. GitHub میں شامل کریں

GitHub میں **Settings → Access → SSH and GPG keys → New GPG key** کھولیں، title دیں، public key paste کریں اور **Add GPG key** منتخب کریں۔ UI بدلنے پر [GitHub کی official guide](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) دیکھیں۔

## 7. Global Git identity

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` اس macOS user کی تمام repositories پر لاگو ہوتا ہے، جب تک local override موجود نہ ہو۔

## 8. Global OpenPGP signing

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

پرانا format چیک کریں:

~~~bash
git config --global --get gpg.format
~~~

اگر value `ssh` ہو اور آپ GPG استعمال کرنا چاہتے ہوں تو default `openpgp` پر واپس آنے کے لیے اسے ہٹائیں:

~~~bash
git config --global --unset gpg.format
~~~

## 9. Local overrides چیک کریں

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` کی priority global config سے زیادہ ہے۔ Repository کے اندر local identity ہٹائیں، `--global` استعمال نہ کریں:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Signed commit test کریں

Test repository میں:

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

`gpg: Good signature from ...` جیسا پیغام نظر آنا چاہیے۔ `git push` کے بعد GitHub پر commit کھولیں؛ `Verified` دکھائی دینا چاہیے۔ Local cryptographic verification اور GitHub account/email association الگ checks ہیں۔

## 🛠️ مسائل کا حل

- **No pinentry:** اگر `gpg: agent_genkey failed: No pinentry` اور `Key generation failed: No pinentry` آئے تو `pinentry-mac` install کریں، `which pinentry-mac` سے path لیں، موجودہ config میں `pinentry-program` شامل کریں اور agent restart کریں۔
- **Homebrew permissions:** صرف error میں بتائی گئی exact directory پر `sudo chown -R "$(whoami)" "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"` اور `chmod u+w "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"` چلائیں۔ پورے `/usr/local` یا `/opt/homebrew` پر chown نہ کریں۔
- **Multiple GPG یا IDE:** `which gpg`، `gpg.program`، PATH اور IDE کے Git/GPG compare کریں؛ سب کو private key والے keyring کا استعمال کرنا چاہیے۔
- **GitHub پر Verified نہیں:** public key، `user.email`، verified email، `gpg.format` اور `git log --show-signature -1` check کریں۔ Expired key renew کریں۔
- **Private key ضائع ہو:** GitHub کی public key سے private key دوبارہ نہیں بن سکتی۔ محفوظ backup restore کریں یا نئی key بنائیں؛ revocation certificate صرف key کو ناقابلِ اعتماد قرار دیتا ہے۔

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
