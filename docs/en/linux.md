# 🐧 GPG-signed Git commits on Linux

This page configures OpenPGP/GPG signing for every local Git repository belonging to your Linux user. Linux distributions differ, so installation is the one part that branches.

## What you need

You need Git, a GitHub account, and an email address that is added and verified on GitHub. Use that address in both the GPG key identity and Git's committer configuration.

> ⚠️ The public key can be uploaded to GitHub. The private/secret key, passphrase, and revocation certificate contents must stay private.

## 1. Install GnuPG

Use the command for your distribution family:

### Debian or Ubuntu

~~~bash
sudo apt update
sudo apt install gnupg
~~~

### Fedora

Fedora publishes the package as `gnupg2`:

~~~bash
sudo dnf install gnupg2
~~~

### Arch Linux

Arch's official package is `gnupg` and it includes a `pinentry` dependency:

~~~bash
sudo pacman -Syu gnupg
~~~

If you use another distribution, follow its current package documentation and install GnuPG 2 plus a pinentry implementation suitable for your desktop or terminal.

Verify the executable and version:

~~~bash
gpg --version
which gpg
~~~

Some older installations expose `gpg2`. If that is the executable that owns your keys, use it consistently and set `gpg.program` to its path later.

## 2. Configure a pinentry and the agent

`gpg-agent` is normally started on demand. Pinentry may be a terminal/curses program or a desktop program such as GTK or Qt; choose one that matches how you work. Do not assume GNOME, KDE, Wayland, X11, or systemd unless your own environment uses it.

Find available programs:

~~~bash
which pinentry
which pinentry-curses
which pinentry-gtk-2
which pinentry-qt
~~~

If your distribution provides a generic `pinentry`, try it first. If a GUI is not available, a terminal-friendly option such as `pinentry-curses` is appropriate. If you need to set a specific program, create the directory safely and edit the agent file:

~~~bash
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

Add one setting to `~/.gnupg/gpg-agent.conf`, using an actual path from `which`:

~~~text
pinentry-program /THE/ACTUAL/PATH/TO/PINENTRY
~~~

If the file exists, edit it and preserve its other lines. Restart the agent:

~~~bash
gpgconf --kill gpg-agent
~~~

For terminal prompts, set the current TTY:

~~~bash
export GPG_TTY=$(tty)
~~~

Add that line to the startup file for the shell you actually use, commonly `~/.bashrc` or `~/.zshrc`, then start a new shell or run `source ~/.bashrc` / `source ~/.zshrc` as appropriate. `GPG_TTY` directs pinentry to the current terminal.

## 3. Generate a key

Run:

~~~bash
gpg --full-generate-key
~~~

Choose a straightforward signing-capable RSA 4096-bit key if offered, an expiration you can maintain, a strong passphrase, `YOUR_NAME`, and `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub uses the relationship between the commit committer email, a matching GPG UID, and your verified GitHub email; the UID alone does not set the Git identity.

## 4. Find the full fingerprint

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

Find the `sec` record and copy the complete fingerprint on its `fingerprint` line. Use that as `YOUR_GPG_FINGERPRINT`. A private key is required for signing.

## 5. Export the public key

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Copy the complete block from `-----BEGIN PGP PUBLIC KEY BLOCK-----` to `-----END PGP PUBLIC KEY BLOCK-----`. This exports only the public key. Never upload or share any private/secret key material.

## 6. Add it to GitHub

Go to **Settings → Access → SSH and GPG keys → New GPG key**, give the key a title, paste the public block, and choose **Add GPG key**. See [GitHub's current instructions](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) if labels move.

## 7. Configure global Git identity

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` applies to all repositories for this Linux user unless a repository has a local override.

## 8. Configure global OpenPGP signing

Use the path you verified. `command -v` is a POSIX-friendly alternative to `which`; the example below uses `which` for readability:

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

If your system uses `gpg2`, use `"$(which gpg2)"` instead.

Check whether a previous SSH signing override exists:

~~~bash
git config --global --get gpg.format
~~~

If it returns `ssh`, remove that setting to return Git to its default OpenPGP format:

~~~bash
git config --global --unset gpg.format
~~~

Removing the override matters because otherwise Git may invoke SSH signing even though this guide configured GPG.

## 9. Check local repository overrides

From the repository where signing fails, inspect where each value came from:

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
git config --show-origin --get gpg.format
~~~

`file:.git/config` is local and overrides global config. Remove an unwanted local identity with no `--global`:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Test and verify

In a disposable/test repository:

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Look for `gpg: Good signature from ...`. Push with `git push` and open the commit on GitHub; it should show `Verified`. Local cryptographic verification and GitHub's account/email association are related checks, but not identical.

## 🛠️ Troubleshooting

- **No pinentry / no passphrase prompt:** verify the installed pinentry path, the `pinentry-program` line, `GPG_TTY`, and then run `gpgconf --kill gpg-agent`. Do not overwrite an existing config file just to add one setting.
- **Agent problems:** `gpg-agent` starts on demand; killing it lets GnuPG start a fresh instance. Check permissions on `~/.gnupg` and retry in a new terminal.
- **Multiple GPG installations:** compare `which gpg`, `which gpg2`, and `git config --show-origin --get gpg.program`. Git must use the executable connected to the keyring containing your private key.
- **Terminal vs GUI/IDE:** an IDE may have a different `PATH`, environment, or Git binary. Compare its GPG path with the terminal and test in the same repository.
- **GitHub lacks `Verified`:** confirm the uploaded public key belongs to the signing private key, `user.email` is a verified GitHub email and a UID on the key, and `gpg.format` is not `ssh`.
- **Expired or multiple signing keys:** renew the key or set `user.signingkey` to the intended full fingerprint. A lost private key cannot be rebuilt from the public key on GitHub; restore a secure backup or create a new key. A revocation certificate is used to announce that a key is no longer trustworthy, not to recover it.

## 🔐 Backup and future notes

Keep an encrypted offline backup of the private key and revocation certificate. The public key is shareable; the secret key is not.

🌿 [GitHub](https://github.com/syllik) · [syllik@gmail.com](mailto:syllik@gmail.com)

<details>
<summary>✨ A note for the future</summary>

You will find this again someday, when you want to learn something new.

It is free.

With love,<br>
fireflў
</details>

---

[← English home](README.md) · [🌍 All languages](../../README.md)
