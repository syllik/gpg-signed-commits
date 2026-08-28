# 🐧 Linux-এ GPG-স্বাক্ষরিত Git commit

Linux user-এর জন্য global OpenPGP/GPG signing configure করুন। Installation command distro অনুযায়ী বদলায়।

## প্রস্তুতি ও নিরাপত্তা

GitHub account, verified email এবং Git দরকার। একই email GPG ও `user.email`-এ ব্যবহার করুন। Public key share করা যায়; private/secret key, passphrase এবং revocation certificate গোপন রাখুন।

## ১. GnuPG install

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

অন্য distro হলে তার current official package documentation অনুসরণ করে GnuPG 2 ও উপযুক্ত pinentry install করুন। তারপর:

~~~bash
gpg --version
which gpg
~~~

যদি `gpg2` ব্যবহৃত হয়, একই executable ব্যবহার করে `gpg.program`-এ সেট করুন।

## ২. pinentry ও agent

`gpg-agent` প্রয়োজন হলে নিজে start হয়। Terminal/curses বা GUI pinentry আপনার environment অনুযায়ী নিন; GNOME, KDE, Wayland, X11 বা systemd ধরে নেবেন না।

~~~bash
which pinentry
which pinentry-curses
which pinentry-gtk-2
which pinentry-qt
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

নির্দিষ্ট program দরকার হলে `~/.gnupg/gpg-agent.conf`-এ আসল path যোগ করুন:

~~~text
pinentry-program /THE/ACTUAL/PATH/TO/PINENTRY
~~~

আগের lines রেখে agent restart করুন:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

`export GPG_TTY=$(tty)`-কে `~/.bashrc` বা `~/.zshrc`-এ যোগ করে নতুন shell খুলুন।

## ৩. Key তৈরি

~~~bash
gpg --full-generate-key
~~~

Signing-capable RSA 4096, renew করা যায় এমন expiry, শক্ত passphrase, `YOUR_NAME` এবং `YOUR_VERIFIED_GITHUB_EMAIL` দিন। GitHub committer email, GPG UID এবং verified email মিলিয়ে দেখে।

## ৪. Fingerprint

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

`sec`-এর নিচের সম্পূর্ণ fingerprint `YOUR_GPG_FINGERPRINT` হিসেবে copy করুন।

## ৫. Public key export

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

`-----BEGIN PGP PUBLIC KEY BLOCK-----` থেকে `-----END PGP PUBLIC KEY BLOCK-----` পর্যন্ত public block copy করুন। Secret material share করবেন না।

## ৬. GitHub

**Settings → Access → SSH and GPG keys → New GPG key**-এ public key paste করে যোগ করুন। [Official GitHub documentation](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) দেখুন।

## ৭. Global identity

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` local override না থাকলে এই user-এর সব repository-তে প্রযোজ্য।

## ৮. Global signing

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Format পরীক্ষা করুন:

~~~bash
git config --global --get gpg.format
~~~

`ssh` হলে এবং OpenPGP চাইলে:

~~~bash
git config --global --unset gpg.format
~~~

## ৯. Local override

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` বেশি priority পায়। Local identity সরান:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## ১০. Test

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

`gpg: Good signature from ...` খুঁজুন, `git push` করুন এবং GitHub-এ `Verified` দেখুন। Local check এবং GitHub association একই জিনিস নয়।

## 🛠️ সমস্যা সমাধান

- **pinentry/agent:** program, `pinentry-program`, `GPG_TTY` ও `~/.gnupg` permissions দেখুন; `gpgconf --kill gpg-agent` চালান।
- **Multiple GPG:** `which gpg`, `which gpg2` এবং `git config --show-origin --get gpg.program` তুলনা করুন।
- **Terminal/GUI/IDE:** IDE-র PATH, Git এবং GPG terminal-এর সঙ্গে মিলিয়ে নিন।
- **Verified নেই:** public key, `user.email`, verified email, `gpg.format` ও expiry পরীক্ষা করুন।
- **Key হারানো বা revoke:** Public key private key ফিরিয়ে আনতে পারে না। Backup restore বা নতুন key তৈরি করুন; revocation শুধু trust বাতিলের ঘোষণা।

## 🔐 Backup ও ভবিষ্যৎ

Private key ও revocation certificate encrypted offline backup হিসেবে রাখুন। শুধু public key share করুন।

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume)

<details>
<summary>✨ ভবিষ্যতের জন্য একটি নোট</summary>

কখনও নতুন কিছু শিখতে চাইলে একদিন আপনি আবার এটি খুঁজে পাবেন।

এটি বিনামূল্যে।

ভালোবাসাসহ,<br>
fireflў
</details>

---

[← বাংলা home](README.md) · [🌍 সব ভাষা](../../README.md)
