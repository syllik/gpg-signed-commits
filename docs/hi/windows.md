# 🪟 Windows पर GPG-signed Git commits

यह Windows-native workflow Gpg4win और Kleopatra का उपयोग करता है। Git for Windows में Git Bash मिलता है; PowerShell और Command Prompt अलग shells हैं।

## तैयारी और सुरक्षा

[Gpg4win](https://www.gpg4win.org/download.html) और [Git for Windows](https://git-scm.com/download/win) install करें। GitHub में verified email रखें।

> ⚠️ केवल public key upload करें। Private/secret key, passphrase और revocation certificate share न करें।

## 1. GPG install और locate करें

तीनों shells में:

~~~text
gpg --version
~~~

PowerShell:

~~~powershell
Get-Command gpg -All
Get-Command git -All
~~~

Command Prompt या Git Bash:

~~~text
where gpg
where git
~~~

Kleopatra जिस Gpg4win installation का keyring उपयोग करता है, उसी का `gpg.exe` चुनें। Windows में `/usr/local/bin/gpg` या macOS वाला `GPG_TTY` उपयोग न करें।

## 2. Key बनाएँ

Kleopatra में **File → New Certificate → OpenPGP** चुनें, या किसी भी shell में:

~~~text
gpg --full-generate-key
~~~

Signing-capable RSA 4096, renewable expiry, strong passphrase, `YOUR_NAME` और `YOUR_VERIFIED_GITHUB_EMAIL` भरें। GitHub committer email, UID और verified account email को मिलाता है।

## 3. Fingerprint खोजें

~~~text
gpg --list-secret-keys --keyid-format=long
~~~

`sec` के नीचे पूरा fingerprint copy करें और `YOUR_GPG_FINGERPRINT` रखें।

## 4. Public key export करें

~~~text
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

`-----BEGIN PGP PUBLIC KEY BLOCK-----` से `-----END PGP PUBLIC KEY BLOCK-----` तक copy करें। यह public है; secret material न बाँटें।

## 5. GitHub में जोड़ें

**Settings → Access → SSH and GPG keys → New GPG key** में public key paste करके **Add GPG key** चुनें। [GitHub की official guide](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) देखें।

## 6. Global identity

~~~text
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` सभी repositories पर लागू है, local override को छोड़कर।

## 7. Git को सही GPG दें

PowerShell या Command Prompt में actual path से example बदलें:

~~~powershell
git config --global gpg.program "C:/Program Files/GnuPG/bin/gpg.exe"
~~~

Path `Get-Command gpg` या `where gpg` से लें। फिर:

~~~text
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
git config --global --get gpg.format
~~~

यदि `ssh` दिखे और GPG चाहिए:

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
git config --show-origin --get gpg.format
~~~

`file:.git/config` global value को override करता है। Repository के अंदर:

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

Kleopatra/pinentry passphrase पूछेगा और `gpg: Good signature from ...` दिखना चाहिए। `git push` के बाद GitHub पर `Verified` देखें।

## 🛠️ Troubleshooting

- **Multiple gpg.exe:** `Get-Command gpg -All`/`where gpg` को `gpg.program` से compare करें और Kleopatra वाली installation रखें।
- **Pinentry window नहीं:** Kleopatra खोलें और `gpgconf --kill gpg-agent` चलाएँ। Windows को `GPG_TTY` नहीं चाहिए।
- **IDE:** IDE में वही Git और GPG चुनें जो terminal में काम करते हैं।
- **No Verified:** public key, `user.email`, verified email, `gpg.format` और local signature जाँचें।
- **Expired/lost/revoked key:** renew या नई key बनाएं। Public key private key restore नहीं करती; revocation केवल trust हटाने का संकेत है।

## 🔐 Backup और future note

Kleopatra/GnuPG से private key और revocation certificate का encrypted offline backup रखें। केवल public key साझा करें।

🌿 [GitHub](https://github.com/syllik) · [syllik@gmail.com](mailto:syllik@gmail.com)

<details>
<summary>✨ भविष्य के लिए एक नोट</summary>

कभी न कभी, जब आप कुछ नया सीखना चाहेंगे, तो आपको यह फिर मिल जाएगा।

यह मुफ़्त है।

स्नेह सहित,<br>
fireflў
</details>

---

[← हिन्दी home](README.md) · [🌍 सभी भाषाएँ](../../README.md)
