# 🪟 GPG-signed Git commits on Windows

This page uses a Windows-native GnuPG setup. Gpg4win provides GnuPG and Kleopatra; Git for Windows supplies Git Bash, while PowerShell and Command Prompt are separate shells with different path syntax.

## What you need

Install [Gpg4win](https://www.gpg4win.org/download.html) from its official site. It includes GnuPG, `gpg-agent`, pinentry, and Kleopatra. Install [Git for Windows](https://git-scm.com/download/win) if Git is not already present.

> ⚠️ Export and upload only a **public** key. Never upload or share a private/secret key, passphrase, or revocation certificate contents.

## 1. Install and identify the tools

Run `gpg --version` in **PowerShell**, **Command Prompt**, or **Git Bash**. The command is the same in these shells:

~~~text
gpg --version
~~~

Locate every GPG executable so Git and your shell do not silently use different installations.

In **PowerShell**:

~~~powershell
Get-Command gpg -All
Get-Command git -All
~~~

In **Command Prompt** or **Git Bash**:

~~~text
where gpg
where git
~~~

You may see a path under `C:\Program Files\GnuPG\bin\gpg.exe` or the Gpg4win installation directory. Use the path that belongs to the installation whose Kleopatra keyring contains your key. Gpg4win normally starts its own pinentry and agent; you do not need Unix `~/.gnupg` or `/usr/local/bin` instructions on Windows.

Open **Kleopatra** from the Start menu when you want a graphical certificate manager. It and the command-line `gpg.exe` use the same GnuPG home when they belong to the same Gpg4win installation.

## 2. Generate a key

You can use Kleopatra's **File → New Certificate**, select **OpenPGP**, and create a personal certificate. Or use the same interactive command in **PowerShell**, **Command Prompt**, or **Git Bash**:

~~~text
gpg --full-generate-key
~~~

Choose a signing-capable RSA 4096-bit key if offered, an expiration you can maintain, a strong passphrase, `YOUR_NAME`, and `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub matches the commit's committer email to an email identity on the GPG key and a verified address on your account. The GPG UID alone is not a replacement for Git's `user.email`.

## 3. Find the full fingerprint

Run in any of the three shells:

~~~text
gpg --list-secret-keys --keyid-format=long
~~~

Copy the complete fingerprint from the `fingerprint` line under the `sec` record and use it as `YOUR_GPG_FINGERPRINT`. A private key is required to sign.

## 4. Export the public key

In **PowerShell**, print the ASCII-armored public key:

~~~powershell
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

In **Command Prompt** or **Git Bash**, the same command works:

~~~text
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Copy from `-----BEGIN PGP PUBLIC KEY BLOCK-----` through `-----END PGP PUBLIC KEY BLOCK-----`. The output is public and safe to upload. Never run a secret-key export and never share a private-key block.

## 5. Add the public key to GitHub

In GitHub choose **Settings → Access → SSH and GPG keys → New GPG key**, enter a title, paste the public block, choose **Add GPG key**, and authenticate if asked. See [GitHub's current UI instructions](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 6. Configure global Git identity

These commands work in **PowerShell**, **Command Prompt**, and **Git Bash**:

~~~text
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` applies to all repositories for this Windows user unless a repository overrides it.

## 7. Point Git to the correct GPG executable

If `gpg --version` works but Git signs with another installation, set an explicit Windows path. In **PowerShell** or **Command Prompt**, use a quoted Windows path:

~~~powershell
git config --global gpg.program "C:/Program Files/GnuPG/bin/gpg.exe"
~~~

Forward slashes avoid extra escaping and are accepted by Git for Windows. Replace the example with the actual path returned by `Get-Command gpg` or `where gpg`. In **Git Bash**, the same Git setting can use a Windows path in quotes; do not replace it with a guessed `/usr/local/bin/gpg` path.

Set the key and enable OpenPGP signing:

~~~text
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

If you previously configured SSH signing, inspect and, if needed, remove the override from any of the three shells:

~~~text
git config --global --get gpg.format
git config --global --unset gpg.format
~~~

The second command returns Git to its default `openpgp` format. Only run it when you intend to stop using SSH signing.

## 8. Check local overrides

From the repository that will be signed:

~~~text
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

An entry from `file:.git/config` overrides the global Windows config. Remove an unwanted local identity from inside that repository without `--global`:

~~~text
git config --unset user.name
git config --unset user.email
~~~

## 9. Test a signed commit

In a disposable/test repository, run this in **PowerShell**, **Command Prompt**, or **Git Bash**:

~~~text
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Kleopatra or the configured pinentry should ask for the passphrase. Look for `gpg: Good signature from ...`. Then run `git push` and open the commit on GitHub; it should show `Verified`. GitHub also checks the uploaded public key and verified committer email, so local success alone does not guarantee the badge.

## 🛠️ Troubleshooting

### Multiple `gpg.exe` installations

Compare `Get-Command gpg -All` or `where gpg` with `git config --show-origin --get gpg.program`. Remove stale PATH entries or set `gpg.program` to the Gpg4win executable that can see the key in Kleopatra.

### Git uses a different GPG than the shell

Run `gpg --list-secret-keys --keyid-format=long` in the shell and compare it with the same command invoked by Git's configured path. An explicit path such as `C:/Program Files/GnuPG/bin/gpg.exe` avoids ambiguity.

### Pinentry window does not appear

Start Kleopatra, confirm Gpg4win is installed for the current user, and retry after restarting the agent. In a terminal, run `gpgconf --kill gpg-agent`. Do not copy macOS `GPG_TTY` or Unix path instructions into Windows; W32 pinentry does not require `GPG_TTY`.

### Kleopatra and Gpg4win disagree

You may have installed more than one GnuPG suite. Use Kleopatra and `gpg.exe` from the same Gpg4win installation, then set Git's `gpg.program` to that executable. Check the path instead of trusting a shortcut.

### Passphrase or IDE problems

An IDE can use its own Git binary, PATH, or environment. First make a signed commit from Git Bash or PowerShell, then configure the IDE to use the same Git and GPG executables. If terminal signing works but the IDE does not, compare those paths. If the terminal fails too, fix Gpg4win/pinentry first.

### GitHub does not show `Verified`

Confirm the public key on GitHub matches the private signing key, `user.email` is a verified GitHub email and a UID on the key, and `gpg.format` is not `ssh`. Inspect the local signature with `git log --show-signature -1`.

### Expired, multiple, or lost keys

Renew an expiring key and update GitHub if necessary. Set `user.signingkey` to the intended full fingerprint when multiple keys exist. A public key on GitHub cannot recreate a lost private key; restore a secure backup or create a new key. A revocation certificate announces that a key should no longer be trusted; it is not a recovery file.

## 🔐 Backup and future notes

Use Kleopatra or GnuPG to keep an encrypted offline backup of your private key and revocation certificate. Share only the public key.

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume) · [Telegram — @syllik](https://t.me/syllik)

<details>
<summary>✨ A note for the future</summary>

You will find this again someday, when you want to learn something new.

It is free.

With love,<br>
fireflў
</details>

---

[← English home](README.md) · [🌍 All languages](../../README.md)
