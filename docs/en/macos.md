# 🍎 GPG-signed Git commits on macOS

This page configures OpenPGP/GPG signing for every local Git repository belonging to your macOS user.

## What you will configure

You will install GnuPG and `pinentry-mac`, create a key, upload only its public part to GitHub, then set global Git identity and signing options.

You need a GitHub account, Git, Homebrew, and an email address that is added and verified on GitHub. Use that same address in the GPG key and in Git's committer identity.

> ⚠️ **Public vs private:** `gpg --armor --export` exports a **public** key. Never export, upload, or share secret-key material, a passphrase, or revocation certificate contents.

## 1. Install and verify GnuPG

Homebrew installs command-line tools into a prefix that depends on the Mac. Install both GnuPG and the macOS pinentry program:

~~~bash
brew install gnupg pinentry-mac
~~~

Check what is actually being used. Do not assume Intel or Apple Silicon paths:

~~~bash
gpg --version
which gpg
which pinentry-mac
~~~

`gpg --version` should print a GnuPG 2.x version. The `which` commands should print the executable paths that the rest of this page will use.

## 2. Configure the pinentry program safely

`gpg-agent` manages secret keys and asks a pinentry program for the passphrase. The setting belongs in `~/.gnupg/gpg-agent.conf`.

Create the directory only if needed and protect it:

~~~bash
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

If `~/.gnupg/gpg-agent.conf` does not exist, create it with one line using the path you discovered:

~~~text
pinentry-program /THE/ACTUAL/PATH/FROM/which-pinentry-mac
~~~

If the file already exists, edit it and add or update only the `pinentry-program` line. Do not blindly overwrite existing GnuPG settings. Homebrew normally uses `/opt/homebrew` on Apple Silicon and `/usr/local` on Intel, but discovery is safer than guessing.

Restart the agent and tell terminal-based GPG which TTY to use:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

For persistence in zsh, add this line to `~/.zshrc`, then reload it:

~~~bash
export GPG_TTY=$(tty)
~~~

~~~bash
source ~/.zshrc
~~~

`GPG_TTY` lets GPG find the current terminal when a passphrase prompt is needed. A GUI pinentry can still open its own window.

## 3. Generate a key

Run the interactive wizard:

~~~bash
gpg --full-generate-key
~~~

For a straightforward GitHub setup, choose an RSA signing-capable key at 4096 bits if offered, set an expiration that you can renew, and protect the key with a strong passphrase. Enter `YOUR_NAME` and `YOUR_VERIFIED_GITHUB_EMAIL` as the identity. GitHub associates the signature with your account only when the commit's committer email matches an email identity on the key and a verified email on GitHub.

The wizard should finish with a message that a key was created. Keep the passphrase private.

## 4. Find the full fingerprint

List keys that include a private key:

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

Look for a `sec rsa4096/…` line. The long identifier is useful, but prefer the complete fingerprint shown on the following `fingerprint` line. Use it as `YOUR_GPG_FINGERPRINT` below. A private key is required to sign; the public key alone cannot create signatures.

## 5. Export only the public key

Export an ASCII-armored public key:

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Copy from `-----BEGIN PGP PUBLIC KEY BLOCK-----` through `-----END PGP PUBLIC KEY BLOCK-----`. This is safe to upload to GitHub. Never use a secret-key export and never paste or share secret-key material.

## 6. Add the public key to GitHub

In GitHub:

1. Open your profile menu and choose **Settings**.
2. In **Access**, choose **SSH and GPG keys**.
3. Beside **GPG keys**, choose **New GPG key**.
4. Enter a title, paste the public key, and choose **Add GPG key**.
5. Authenticate if GitHub asks you to confirm.

Use the [official GitHub instructions](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) if the UI wording changes.

## 7. Configure global Git identity

Set the name and verified email Git should write into commits:

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` applies to repositories for this macOS user unless a repository overrides the value. The GPG UID email alone does not replace Git's committer identity; both need to line up for GitHub verification.

## 8. Configure global OpenPGP signing

Point Git at the GPG executable you verified and enable signed commits and tags:

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

If you previously configured SSH signing, inspect the format:

~~~bash
git config --global --get gpg.format
~~~

If it prints `ssh` and you want OpenPGP/GPG, remove that global override before testing:

~~~bash
git config --global --unset gpg.format
~~~

This lets Git use its default `openpgp` format. The command may print an error when no such setting exists; that is harmless.

## 9. Check local overrides

Repository configuration in `.git/config` has higher priority than `~/.gitconfig`. From a repository, inspect the origin of each value:

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

If a line begins with `file:.git/config`, it overrides the global value. Remove an unwanted local identity from inside that repository with:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

Do not add `--global` here: that would remove the global setting instead.

## 10. Test a signed commit

In a disposable or existing test repository, create an empty signed commit:

~~~bash
git commit --allow-empty -m "test: signed commit"
~~~

Enter the passphrase in the pinentry prompt. Then inspect the signature locally:

~~~bash
git log --show-signature -1
git verify-commit HEAD
~~~

Successful output contains a good-signature message such as `gpg: Good signature from ...` and `git verify-commit` exits successfully. This proves local cryptographic verification; GitHub additionally checks that the public key is on your account and the committer email is verified there.

Push the commit:

~~~bash
git push
~~~

Open the commit on GitHub. It should display `Verified`. Select the badge to inspect details.

## 🛠️ Troubleshooting

### `gpg: agent_genkey failed: No pinentry`

If key generation ends with:

~~~text
gpg: agent_genkey failed: No pinentry
Key generation failed: No pinentry
~~~

Install the GUI pinentry, discover its path, edit the existing agent configuration without deleting other lines, restart the agent, and restore the TTY variable:

~~~bash
brew install pinentry-mac
which pinentry-mac
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

Then add `pinentry-program /THE/ACTUAL/PATH/FROM/which-pinentry-mac` to `~/.gnupg/gpg-agent.conf` and run `gpg --full-generate-key` again.

### Homebrew says a directory is not writable

Do not run a broad ownership command such as `sudo chown -R $(whoami) /usr/local` or `sudo chown -R $(whoami) /opt/homebrew`. Read the error and change only the exact directory Homebrew names:

~~~bash
sudo chown -R "$(whoami)" "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"
chmod u+w "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"
~~~

These commands change ownership and user write permission for that one reported directory. Substitute the exact path from your own error.

### GPG cannot ask for the passphrase

Check `which gpg`, `which pinentry-mac`, `~/.gnupg/gpg-agent.conf`, and `GPG_TTY`. Run `gpgconf --kill gpg-agent`, open a new terminal, and retry. If multiple GPG installations exist, make `gpg.program` match the same `gpg` that owns the keyring.

### Terminal works but an IDE does not (or the reverse)

The IDE may use a different `PATH`, environment, or GPG executable. Compare its configured Git/GPG path with `which gpg` and check the repository's `gpg.program` and local overrides. Test a commit in the same repository from the terminal.

### GitHub does not show `Verified`

Confirm that the public key was added to the correct GitHub account, the commit uses the private key whose public half you uploaded, and `user.email` is both a UID on that key and a verified email on GitHub. Check `git log --show-signature -1` locally. Also inspect `gpg.format`: `ssh` means Git is not using OpenPGP.

### The key is expired, there are multiple keys, or the private key is lost

Renew an expiring key before it expires and update GitHub when appropriate. With multiple keys, set `user.signingkey` to the full fingerprint of the intended key. A public key on GitHub cannot reconstruct a lost private key; restore the private key from a secure backup or create a new key and add its public key. A revocation certificate is a prepared way to publish that a key should no longer be trusted; store it securely and use it only when the key is genuinely compromised or permanently lost.

## 🔐 Backup and future notes

Back up the private key and revocation certificate offline, encrypted and separately from your computer. Never put them in this repository. The public key may be shared; the secret key may not.

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume)

<details>
<summary>✨ A note for the future</summary>

You will find this again someday, when you want to learn something new.

It is free.

With love,<br>
fireflў
</details>

---

[← English home](README.md) · [🌍 All languages](../../README.md)
