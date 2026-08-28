# 🪟 Commit Git bertanda tangan GPG di Windows

Alur ini memakai Gpg4win dan Kleopatra. Git for Windows menyediakan Git Bash; PowerShell dan Command Prompt memiliki sintaks yang berbeda.

## Persiapan dan keamanan

Instal [Gpg4win](https://www.gpg4win.org/download.html) dan [Git for Windows](https://git-scm.com/download/win). Siapkan email GitHub yang sudah diverifikasi.

> ⚠️ Unggah hanya kunci publik. Jangan membagikan kunci privat/rahasia, passphrase, atau sertifikat pencabutan.

## 1. Instal dan temukan GPG

Di PowerShell, Command Prompt, atau Git Bash:

~~~text
gpg --version
~~~

PowerShell:

~~~powershell
Get-Command gpg -All
Get-Command git -All
~~~

Command Prompt atau Git Bash:

~~~text
where gpg
where git
~~~

Gunakan `gpg.exe` dari instalasi Gpg4win yang keyring-nya dipakai Kleopatra. Jangan gunakan `/usr/local/bin/gpg` atau menyalin `GPG_TTY` dari macOS.

## 2. Buat kunci

Di Kleopatra pilih **File → New Certificate → OpenPGP**, atau jalankan:

~~~text
gpg --full-generate-key
~~~

Pilih RSA 4096 untuk tanda tangan bila tersedia, masa berlaku yang dapat diperpanjang, passphrase kuat, `YOUR_NAME`, dan `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub mengaitkan email committer, UID, dan email akun yang terverifikasi.

## 3. Temukan fingerprint

~~~text
gpg --list-secret-keys --keyid-format=long
~~~

Salin fingerprint lengkap di bawah `sec` sebagai `YOUR_GPG_FINGERPRINT`.

## 4. Ekspor kunci publik

~~~text
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Salin blok antara `-----BEGIN PGP PUBLIC KEY BLOCK-----` dan `-----END PGP PUBLIC KEY BLOCK-----`. Itu publik; jangan bagikan material rahasia.

## 5. Tambahkan ke GitHub

Buka **Settings → Access → SSH and GPG keys → New GPG key**, tempel kunci publik, dan pilih **Add GPG key**. Lihat [panduan resmi](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 6. Identitas global

~~~text
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` berlaku untuk semua repositori pengguna Windows ini kecuali override lokal.

## 7. Arahkan Git ke GPG yang benar

Di PowerShell atau Command Prompt, ganti contoh path dengan hasil `Get-Command gpg` atau `where gpg`:

~~~powershell
git config --global gpg.program "C:/Program Files/GnuPG/bin/gpg.exe"
~~~

Git for Windows menerima path Windows dengan tanda kutip. Lalu:

~~~text
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
git config --global --get gpg.format
~~~

Jika hasilnya `ssh` dan kamu ingin GPG:

~~~text
git config --global --unset gpg.format
~~~

## 8. Periksa override lokal

~~~text
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` lebih tinggi prioritasnya. Di dalam repositori:

~~~text
git config --unset user.name
git config --unset user.email
~~~

## 9. Uji

~~~text
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Kleopatra atau pinentry akan meminta passphrase. Cari `gpg: Good signature from ...`, jalankan `git push`, lalu periksa `Verified` di GitHub.

## 🛠️ Pemecahan masalah

- **Beberapa gpg.exe:** bandingkan `Get-Command gpg -All` / `where gpg` dengan `gpg.program` dan pakai instalasi yang sama dengan Kleopatra.
- **Jendela pinentry tidak muncul:** buka Kleopatra lalu jalankan `gpgconf --kill gpg-agent`. Windows tidak memerlukan `GPG_TTY`.
- **IDE gagal:** atur IDE agar menggunakan Git dan GPG yang sama dengan terminal.
- **Tidak ada Verified:** periksa kunci publik, `user.email`, email terverifikasi, `gpg.format`, dan output `git log --show-signature -1`.
- **Kunci kedaluwarsa/hilang/dicabut:** perpanjang atau buat baru. Kunci publik tidak dapat memulihkan privat; sertifikat pencabutan hanya menyatakan kunci tidak tepercaya.

## 🔐 Cadangan dan catatan masa depan

Dengan Kleopatra atau GnuPG, simpan cadangan terenkripsi offline untuk kunci privat dan sertifikat pencabutan. Bagikan hanya kunci publik.

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume) · [Telegram — @syllik](https://t.me/syllik)

<details>
<summary>✨ Catatan untuk masa depan</summary>

Suatu hari kamu akan menemukan ini lagi ketika ingin mempelajari sesuatu yang baru.

Ini gratis.

Dengan kasih,<br>
fireflў
</details>

---

[← Beranda Indonesia](README.md) · [🌍 Semua bahasa](../../README.md)
