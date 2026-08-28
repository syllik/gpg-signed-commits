# 🐧 Linux पर GPG-signed Git commits

यह guide Linux user के लिए global OpenPGP/GPG signing configure करती है। Installation command distro पर निर्भर है।

## तैयारी और सुरक्षा

GitHub account, verified email, और Git चाहिए। वही email GPG और `user.email` में रखें। Public key share की जा सकती है; private/secret key, passphrase और revocation certificate private रखें।

## 1. GnuPG install करें

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

दूसरी distro के लिए उसकी current official package documentation देखें और GnuPG 2 तथा suitable pinentry install करें। जाँचें:

~~~bash
gpg --version
which gpg
~~~

यदि system `gpg2` देता है तो उसी को consistently use करें और बाद में `gpg.program` में वही path रखें।

## 2. pinentry और agent configure करें

`gpg-agent` जरूरत पड़ने पर start होता है। Terminal/curses या GUI pinentry अपने environment के अनुसार चुनें; GNOME, KDE, Wayland, X11 या systemd assume न करें।

~~~bash
which pinentry
which pinentry-curses
which pinentry-gtk-2
which pinentry-qt
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

Specific program चुनने पर उसका actual path `~/.gnupg/gpg-agent.conf` में जोड़ें:

~~~text
pinentry-program /THE/ACTUAL/PATH/TO/PINENTRY
~~~

मौजूदा lines बचाएँ, फिर:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

अपने shell के अनुसार `export GPG_TTY=$(tty)` को `~/.bashrc` या `~/.zshrc` में जोड़ें और नया shell खोलें।

## 3. key बनाएँ

~~~bash
gpg --full-generate-key
~~~

Signing-capable RSA 4096, renew की जा सकने वाली expiry, strong passphrase, `YOUR_NAME` और `YOUR_VERIFIED_GITHUB_EMAIL` चुनें। GitHub committer email, GPG UID और verified account email का संबंध जाँचता है।

## 4. fingerprint खोजें

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

`sec` के नीचे पूरा fingerprint copy करके `YOUR_GPG_FINGERPRINT` रखें।

## 5. public key export करें

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Public block को `-----BEGIN PGP PUBLIC KEY BLOCK-----` से `-----END PGP PUBLIC KEY BLOCK-----` तक copy करें। Secret material share न करें।

## 6. GitHub में जोड़ें

**Settings → Access → SSH and GPG keys → New GPG key** पर जाकर public key paste करें। [Official GitHub instructions](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) देखें।

## 7. Global identity सेट करें

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` इस user के repositories पर लागू होता है, local override को छोड़कर।

## 8. Global signing enable करें

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Format देखें:

~~~bash
git config --global --get gpg.format
~~~

यदि `ssh` हो और OpenPGP चाहिए, तो:

~~~bash
git config --global --unset gpg.format
~~~

## 9. Local overrides जाँचें

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` की priority अधिक है। Local identity हटाने के लिए:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Test करें

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

`gpg: Good signature from ...` देखें, फिर `git push` करें और GitHub पर `Verified` जाँचें। Local verification और GitHub association बिल्कुल एक ही check नहीं हैं।

## 🛠️ Troubleshooting

- **Pinentry/agent:** program, `pinentry-program`, `GPG_TTY` और `~/.gnupg` permissions जाँचें; फिर `gpgconf --kill gpg-agent` चलाएँ।
- **Multiple GPG:** `which gpg`, `which gpg2` और `git config --show-origin --get gpg.program` compare करें।
- **Terminal/GUI/IDE:** IDE के PATH, Git और GPG की तुलना terminal से करें।
- **No Verified:** public key, `user.email`, verified email, `gpg.format` और key expiry जाँचें।
- **Key lost/revoked:** Public key private key वापस नहीं बना सकती। Backup restore करें या नई key बनाएं; revocation सिर्फ trust हटाने की घोषणा है।

## 🔐 Backup और future note

Private key और revocation certificate का encrypted offline backup रखें। केवल public key साझा करें।

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume) · [Telegram — @syllik](https://t.me/syllik)

<details>
<summary>✨ भविष्य के लिए एक नोट</summary>

कभी न कभी, जब आप कुछ नया सीखना चाहेंगे, तो आपको यह फिर मिल जाएगा।

यह मुफ़्त है।

स्नेह सहित,<br>
fireflў
</details>

---

[← हिन्दी home](README.md) · [🌍 सभी भाषाएँ](../../README.md)
