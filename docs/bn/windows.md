# 🪟 Windows-এ GPG-স্বাক্ষরিত Git commit

এই Windows-native workflow-তে Gpg4win ও Kleopatra ব্যবহার করা হয়। Git for Windows-এ Git Bash থাকে; PowerShell ও Command Prompt আলাদা shell।

## প্রস্তুতি ও নিরাপত্তা

[Gpg4win](https://www.gpg4win.org/download.html) এবং [Git for Windows](https://git-scm.com/download/win) install করুন। GitHub-এ verified email রাখুন।

> ⚠️ শুধু public key upload করুন। Private/secret key, passphrase ও revocation certificate share করবেন না।

## ১. GPG install ও locate

PowerShell, Command Prompt বা Git Bash:

~~~text
gpg --version
~~~

PowerShell:

~~~powershell
Get-Command gpg -All
Get-Command git -All
~~~

Command Prompt বা Git Bash:

~~~text
where gpg
where git
~~~

Kleopatra যে Gpg4win installation-এর keyring ব্যবহার করে, সেই installation-এর `gpg.exe` নিন। Windows-এ `/usr/local/bin/gpg` বা macOS-এর `GPG_TTY` ব্যবহার করবেন না।

## ২. Key তৈরি

Kleopatra-তে **File → New Certificate → OpenPGP** বেছে নিন, অথবা:

~~~text
gpg --full-generate-key
~~~

Signing-capable RSA 4096, renew করা যায় এমন expiry, strong passphrase, `YOUR_NAME` ও `YOUR_VERIFIED_GITHUB_EMAIL` দিন। GitHub committer email, UID এবং verified account email মিলিয়ে দেখে।

## ৩. Fingerprint

~~~text
gpg --list-secret-keys --keyid-format=long
~~~

`sec`-এর নিচে সম্পূর্ণ fingerprint copy করে `YOUR_GPG_FINGERPRINT` রাখুন।

## ৪. Public key export

~~~text
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

`-----BEGIN PGP PUBLIC KEY BLOCK-----` থেকে `-----END PGP PUBLIC KEY BLOCK-----` পর্যন্ত copy করুন। এটি public; secret material share করবেন না।

## ৫. GitHub-এ যোগ

**Settings → Access → SSH and GPG keys → New GPG key**-এ public key paste করে **Add GPG key** চাপুন। [GitHub official guide](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) দেখুন।

## ৬. Global identity

~~~text
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` local override না থাকলে সব repository-তে প্রযোজ্য।

## ৭. Git-কে সঠিক GPG দিন

PowerShell বা Command Prompt-এ actual path দিয়ে উদাহরণটি বদলান:

~~~powershell
git config --global gpg.program "C:/Program Files/GnuPG/bin/gpg.exe"
~~~

Path `Get-Command gpg` বা `where gpg` থেকে নিন। এরপর:

~~~text
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
git config --global --get gpg.format
~~~

`ssh` হলে এবং GPG চাইলে:

~~~text
git config --global --unset gpg.format
~~~

## ৮. Local override

~~~text
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
git config --show-origin --get gpg.format
~~~

`file:.git/config` global-এর ওপর priority পায়। Repository-তে:

~~~text
git config --unset user.name
git config --unset user.email
~~~

## ৯. Test

~~~text
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Kleopatra বা pinentry passphrase চাইবে এবং `gpg: Good signature from ...` দেখা উচিত। `git push` করে GitHub-এ `Verified` দেখুন।

## 🛠️ সমস্যা সমাধান

- **একাধিক gpg.exe:** `Get-Command gpg -All`/`where gpg`-এর output `gpg.program`-এর সঙ্গে তুলনা করুন এবং Kleopatra-র একই installation ব্যবহার করুন।
- **pinentry window নেই:** Kleopatra খুলে `gpgconf --kill gpg-agent` চালান। Windows-এ `GPG_TTY` দরকার নেই।
- **IDE:** Terminal-এ কাজ করা একই Git ও GPG IDE-তে নির্বাচন করুন।
- **Verified নেই:** public key, `user.email`, verified email, `gpg.format` ও local signature পরীক্ষা করুন।
- **Expired/lost/revoked:** renew বা নতুন key তৈরি করুন। Public key private key restore করে না; revocation শুধু trust বাতিল জানায়।

## 🔐 Backup ও ভবিষ্যৎ

Kleopatra বা GnuPG দিয়ে private key ও revocation certificate-এর encrypted offline backup রাখুন। শুধু public key share করুন।

🌿 [GitHub](https://github.com/syllik) · [syllik@gmail.com](mailto:syllik@gmail.com)

<details>
<summary>✨ ভবিষ্যতের জন্য একটি নোট</summary>

কখনও নতুন কিছু শিখতে চাইলে একদিন আপনি আবার এটি খুঁজে পাবেন।

এটি বিনামূল্যে।

ভালোবাসাসহ,<br>
fireflў
</details>

---

[← বাংলা home](README.md) · [🌍 সব ভাষা](../../README.md)
