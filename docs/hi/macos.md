# 🍎 macOS पर GPG-signed Git commits

यहाँ OpenPGP/GPG signing को वर्तमान macOS user के सभी local Git repositories के लिए configure किया जाएगा।

## शुरू करने से पहले

आपको Git, Homebrew, GitHub account और GitHub में verified email address चाहिए। उसी email को GPG UID और Git के `user.email` में रखें।

> ⚠️ **Public key** GitHub पर upload की जा सकती है। Private/secret key, passphrase और revocation certificate का content कभी share न करें।

## 1. GnuPG install और verify करें

Homebrew prefix Mac के अनुसार बदल सकता है। GnuPG और macOS pinentry install करें और असली paths देखें:

~~~bash
brew install gnupg pinentry-mac
gpg --version
which gpg
which pinentry-mac
~~~

`which` के output पर भरोसा करें; `/usr/local` या `/opt/homebrew` का path अनुमान से न लिखें।

## 2. pinentry और gpg-agent configure करें

`gpg-agent` secret keys manage करता है और passphrase के लिए pinentry बुलाता है। इसकी file `~/.gnupg/gpg-agent.conf` है।

~~~bash
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

अगर file नहीं है, तो `which pinentry-mac` से मिले path के साथ यह line जोड़ें:

~~~text
pinentry-program /THE/ACTUAL/PATH/FROM/which-pinentry-mac
~~~

File पहले से हो तो सिर्फ यह setting edit करें और बाकी lines बचाएँ। फिर:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

zsh के लिए `export GPG_TTY=$(tty)` को `~/.zshrc` में जोड़ें और `source ~/.zshrc` चलाएँ। इससे terminal को current TTY मिलती है।

## 3. key बनाइए

~~~bash
gpg --full-generate-key
~~~

यदि option उपलब्ध हो तो signing-capable RSA 4096 चुनें, ऐसा expiry रखें जिसे renew कर सकें, strong passphrase रखें, और `YOUR_NAME` तथा `YOUR_VERIFIED_GITHUB_EMAIL` भरें। GitHub commit के committer email को key UID और account के verified email से मिलाता है; UID अकेले Git identity नहीं बनाता।

## 4. पूरा fingerprint खोजें

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

`sec` entry के नीचे पूरा fingerprint copy करें और उसे `YOUR_GPG_FINGERPRINT` मानें। Sign करने के लिए private key जरूरी है।

## 5. केवल public key export करें

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

`-----BEGIN PGP PUBLIC KEY BLOCK-----` से `-----END PGP PUBLIC KEY BLOCK-----` तक पूरा block copy करें। यह public key है; secret key material कभी share न करें।

## 6. GitHub में key जोड़ें

GitHub में **Settings → Access → SSH and GPG keys → New GPG key** खोलें, title दें, public key paste करें और **Add GPG key** चुनें। UI बदले तो [GitHub की official guide](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) देखें।

## 7. Git identity global रूप से सेट करें

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` इस macOS user के सभी repositories पर लागू होता है, जब तक कोई local override न हो।

## 8. OpenPGP signing global रूप से enable करें

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

पुराना signing format देखें:

~~~bash
git config --global --get gpg.format
~~~

यदि value `ssh` है और आपको GPG चाहिए, तो default `openpgp` पर लौटने के लिए हटाएँ:

~~~bash
git config --global --unset gpg.format
~~~

## 9. local overrides जाँचें

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
git config --show-origin --get gpg.format
~~~

`file:.git/config` global config पर प्राथमिकता रखता है। उसी repository में local identity हटाएँ, `--global` न लगाएँ:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. signed commit test करें

Test repository में:

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

`gpg: Good signature from ...` जैसा संदेश दिखना चाहिए। `git push` के बाद GitHub पर commit खोलें; `Verified` दिखना चाहिए। Local cryptographic check और GitHub account/email association अलग checks हैं।

## 🛠️ समस्याएँ और समाधान

- **No pinentry:** `gpg: agent_genkey failed: No pinentry` और `Key generation failed: No pinentry` पर `pinentry-mac` install करें, `which pinentry-mac` से path लें, मौजूदा config में `pinentry-program` जोड़ें और agent restart करें।
- **Homebrew permissions:** केवल error में बताए गए exact directory पर `sudo chown -R "$(whoami)" "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"` और `chmod u+w "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"` चलाएँ। पूरे `/usr/local` या `/opt/homebrew` पर chown न करें।
- **Multiple GPG या IDE:** `which gpg`, `gpg.program`, PATH और IDE के Git/GPG की तुलना करें; सभी को private key वाले keyring का उपयोग करना चाहिए।
- **GitHub पर Verified नहीं:** public key, `user.email`, verified email, `gpg.format` और `git log --show-signature -1` जाँचें। Expired key renew करें।
- **Private key खो गई:** GitHub की public key से private key वापस नहीं बनती। सुरक्षित backup restore करें या नई key बनाएं। Revocation certificate केवल key को अविश्वसनीय घोषित करता है।

## 🔐 Backup और future note

Private key और revocation certificate का encrypted offline backup रखें। केवल public key share करें।

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
