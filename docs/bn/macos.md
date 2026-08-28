# 🍎 macOS-এ GPG-স্বাক্ষরিত Git commit

বর্তমান macOS user-এর সব local Git repository-র জন্য OpenPGP/GPG signing চালু করুন।

## শুরু করার আগে

Git, Homebrew, GitHub account এবং GitHub-এ যোগ করা ও verified email address দরকার। GPG UID এবং Git-এর `user.email`—দুই জায়গাতেই একই email ব্যবহার করুন।

> ⚠️ **Public key** GitHub-এ upload করা নিরাপদ। Private/secret key, passphrase এবং revocation certificate-এর content কখনও share করবেন না।

## ১. GnuPG install ও যাচাই

Mac-এর ধরন অনুযায়ী Homebrew prefix বদলাতে পারে। GnuPG ও macOS pinentry install করে আসল path দেখুন:

~~~bash
brew install gnupg pinentry-mac
gpg --version
which gpg
which pinentry-mac
~~~

`which`-এর output ব্যবহার করুন; `/usr/local` বা `/opt/homebrew` অনুমান করবেন না।

## ২. pinentry ও gpg-agent configure

`gpg-agent` secret key manage করে এবং passphrase নেওয়ার জন্য pinentry চালায়। Configuration file হল `~/.gnupg/gpg-agent.conf`।

~~~bash
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

File না থাকলে `which pinentry-mac` থেকে পাওয়া path দিয়ে যোগ করুন:

~~~text
pinentry-program /THE/ACTUAL/PATH/FROM/which-pinentry-mac
~~~

File থাকলে শুধু এই line edit করুন, বাকি settings রেখে দিন। Agent restart করে current terminal জানিয়ে দিন:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

zsh ব্যবহার করলে `export GPG_TTY=$(tty)`-কে `~/.zshrc`-এ যোগ করে `source ~/.zshrc` চালান। এতে terminal prompt সঠিক TTY পায়।

## ৩. Key তৈরি

~~~bash
gpg --full-generate-key
~~~

Option থাকলে signing-capable RSA 4096 বেছে নিন, renew করা যাবে এমন expiry, শক্তিশালী passphrase, `YOUR_NAME` এবং `YOUR_VERIFIED_GITHUB_EMAIL` দিন। GitHub commit-এর committer email, key UID এবং account-এর verified email মিলিয়ে দেখে; UID একা Git identity নির্ধারণ করে না।

## ৪. সম্পূর্ণ fingerprint খুঁজুন

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

`sec` record-এর নিচের সম্পূর্ণ fingerprint copy করে `YOUR_GPG_FINGERPRINT` হিসেবে ব্যবহার করুন। Sign করার জন্য private key দরকার।

## ৫. শুধু public key export

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

`-----BEGIN PGP PUBLIC KEY BLOCK-----` থেকে `-----END PGP PUBLIC KEY BLOCK-----` পর্যন্ত copy করুন। এটি public key; secret material কখনও share করবেন না।

## ৬. GitHub-এ যোগ করুন

GitHub-এ **Settings → Access → SSH and GPG keys → New GPG key** খুলুন, title দিন, public key paste করে **Add GPG key** চাপুন। UI বদলালে [GitHub-এর official guide](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) দেখুন।

## ৭. Global Git identity

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` বর্তমান macOS user-এর সব repository-তে প্রযোজ্য, যদি local override না থাকে।

## ৮. Global OpenPGP signing

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

আগের signing format দেখুন:

~~~bash
git config --global --get gpg.format
~~~

Value `ssh` হলে এবং GPG ব্যবহার করতে চাইলে default `openpgp`-এ ফিরুন:

~~~bash
git config --global --unset gpg.format
~~~

## ৯. Local override পরীক্ষা

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` global config-কে override করে। Repository-র ভিতরে local identity সরাতে `--global` ছাড়া চালান:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## ১০. Signed commit test

Test repository-তে:

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

`gpg: Good signature from ...` দেখা উচিত। `git push` করার পর GitHub-এ commit খুলে `Verified` দেখুন। Local cryptographic verification এবং GitHub-এর account/email association আলাদা পরীক্ষা।

## 🛠️ সমস্যা সমাধান

- **No pinentry:** `gpg: agent_genkey failed: No pinentry` এবং `Key generation failed: No pinentry` এলে `pinentry-mac` install করুন, `which pinentry-mac` দিয়ে path নিন, existing config-এ `pinentry-program` যোগ করুন এবং agent restart করুন।
- **Homebrew permission:** Error-এ দেখানো exact directory-তেই `sudo chown -R "$(whoami)" "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"` ও `chmod u+w "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"` চালান। পুরো `/usr/local` বা `/opt/homebrew`-তে chown করবেন না।
- **Multiple GPG বা IDE:** `which gpg`, `gpg.program`, PATH এবং IDE-র Git/GPG তুলনা করুন; private key থাকা একই keyring ব্যবহার করতে হবে।
- **GitHub-এ Verified নেই:** public key, `user.email`, verified email, `gpg.format` এবং `git log --show-signature -1` যাচাই করুন। Expired key renew করুন।
- **Private key হারালে:** GitHub-এর public key থেকে private key পুনরুদ্ধার হয় না। Backup restore করুন বা নতুন key তৈরি করুন; revocation certificate শুধু key-কে আর বিশ্বাস না করার ঘোষণা।

## 🔐 Backup ও ভবিষ্যৎ

Private key এবং revocation certificate-এর encrypted offline backup রাখুন। শুধু public key share করুন।

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume) · [Telegram — @syllik](https://t.me/syllik)

<details>
<summary>✨ ভবিষ্যতের জন্য একটি নোট</summary>

কখনও নতুন কিছু শিখতে চাইলে একদিন আপনি আবার এটি খুঁজে পাবেন।

এটি বিনামূল্যে।

ভালোবাসাসহ,<br>
fireflў
</details>

---

[← বাংলা home](README.md) · [🌍 সব ভাষা](../../README.md)
