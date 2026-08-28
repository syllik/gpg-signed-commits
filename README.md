# 🔐 GPG Signed Commits

Configure OpenPGP/GPG signing for Git commits on your computer and get the `Verified` badge on GitHub.

🍎 macOS · 🐧 Linux · 🪟 Windows

## 🌍 Choose your language

- [English](docs/en/README.md)
- [中文](docs/zh/README.md)
- [हिन्दी](docs/hi/README.md)
- [Español](docs/es/README.md)
- [العربية](docs/ar/README.md)
- [Français](docs/fr/README.md)
- [বাংলা](docs/bn/README.md)
- [Português](docs/pt/README.md)
- [Bahasa Indonesia](docs/id/README.md)
- [اردو](docs/ur/README.md)
- [Русский](docs/ru/README.md)
- [Deutsch](docs/de/README.md)

## What this guide does

A GPG signature is a cryptographic mark made with a private key. Git stores that signature in a commit, and GitHub can show `Verified` when it can check the signature against the matching public key on your account. This is about OpenPGP/GPG commit signing, not SSH commit signing.

The setup below can be global: `git config --global` applies to all local repositories for your OS user unless a repository has its own override.

> ⚠️ **Security warning:** the **public key** is safe to export and upload to GitHub. The **private/secret key** must never be uploaded, pasted into an issue, committed, or shared. Protect it with a strong passphrase and keep a secure backup.

## Supported systems

- 🍎 [macOS](docs/en/macos.md)
- 🐧 [Linux](docs/en/linux.md)
- 🪟 [Windows](docs/en/windows.md)

The detailed guides are maintained as plain Markdown so they can be copied, translated, forked, and read for years.

## Official references

- [GitHub: Signing commits](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)
- [GitHub: Adding a GPG key to your account](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account)
- [GnuPG manual](https://gnupg.org/documentation/)

## Free to reuse

This guide is dedicated to the public domain under [CC0 1.0](LICENSE). Copy it, translate it, teach from it, or improve it for free.

## 🌿 Quiet footer

- [YouTube — @plainsight37](https://youtube.com/@plainsight37)
- [Instagram — @fly_lume](https://instagram.com/fly_lume)
- [Telegram — @syllik](https://t.me/syllik)
