# 🐧 Commit Git bertanda tangan GPG di Linux

Atur penandatanganan OpenPGP/GPG global untuk pengguna Linux. Perintah instalasi bergantung pada distro.

## Persiapan dan keamanan

Siapkan Git, akun GitHub, serta email yang sudah diverifikasi. Gunakan email itu di GPG dan `user.email`. Kunci publik boleh dibagikan; kunci privat/rahasia, passphrase, dan sertifikat pencabutan harus tetap rahasia.

## 1. Instal GnuPG

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

Untuk distro lain, ikuti dokumentasi paket resminya dan instal GnuPG 2 serta pinentry yang sesuai dengan terminal atau desktop. Periksa:

~~~bash
gpg --version
which gpg
~~~

Jika distro memakai `gpg2`, gunakan program itu secara konsisten dan arahkan `gpg.program` kepadanya.

## 2. Atur pinentry dan agent

`gpg-agent` biasanya berjalan saat dibutuhkan. Pilih pinentry terminal atau GUI sesuai lingkunganmu; jangan mengasumsikan GNOME, KDE, Wayland, X11, atau systemd.

~~~bash
which pinentry
which pinentry-curses
which pinentry-gtk-2
which pinentry-qt
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

Jika perlu memilih program tertentu, tambahkan path nyatanya ke `~/.gnupg/gpg-agent.conf`:

~~~text
pinentry-program /THE/ACTUAL/PATH/TO/PINENTRY
~~~

Pertahankan baris yang sudah ada, lalu:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

Tambahkan `export GPG_TTY=$(tty)` ke `~/.bashrc` atau `~/.zshrc` dan buka shell baru.

## 3. Buat kunci

~~~bash
gpg --full-generate-key
~~~

Pilih RSA 4096 untuk tanda tangan bila tersedia, masa berlaku yang bisa diperpanjang, passphrase kuat, `YOUR_NAME`, dan `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub harus dapat mengaitkan email committer, UID GPG, dan email akun yang terverifikasi.

## 4. Temukan fingerprint

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

Salin fingerprint lengkap dari entri `sec` sebagai `YOUR_GPG_FINGERPRINT`.

## 5. Ekspor kunci publik

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Salin blok publik antara `-----BEGIN PGP PUBLIC KEY BLOCK-----` dan `-----END PGP PUBLIC KEY BLOCK-----`. Jangan membagikan material kunci rahasia.

## 6. Tambahkan ke GitHub

Gunakan **Settings → Access → SSH and GPG keys → New GPG key**, tempel kunci publik, lalu tambahkan. Lihat [dokumentasi GitHub](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 7. Identitas global

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` berlaku untuk repositori pengguna ini kecuali ada override lokal.

## 8. Penandatanganan global

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Periksa format:

~~~bash
git config --global --get gpg.format
~~~

Jika `ssh` dan kamu ingin OpenPGP, hapus override agar kembali ke `openpgp`:

~~~bash
git config --global --unset gpg.format
~~~

## 9. Periksa override lokal

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
git config --show-origin --get gpg.format
~~~

`file:.git/config` lebih tinggi prioritasnya. Hapus identitas lokal dengan:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Uji dan verifikasi

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Cari `gpg: Good signature from ...`, jalankan `git push`, lalu periksa `Verified` di GitHub. Verifikasi lokal dan pengaitan akun GitHub bukan hal yang sama persis.

## 🛠️ Pemecahan masalah

- **Pinentry atau agent:** periksa program, `pinentry-program`, `GPG_TTY`, izin `~/.gnupg`, lalu jalankan `gpgconf --kill gpg-agent`.
- **Beberapa instalasi GPG:** bandingkan `which gpg`, `which gpg2`, dan `git config --show-origin --get gpg.program`.
- **Terminal/GUI/IDE:** bandingkan PATH, Git, dan GPG yang digunakan IDE.
- **Tidak ada Verified:** periksa kunci publik, `user.email`, email terverifikasi, `gpg.format`, dan keyring; perpanjang kunci yang kedaluwarsa.
- **Kunci hilang atau dicabut:** kunci publik tidak dapat membangun ulang kunci privat. Pulihkan cadangan atau buat kunci baru; pencabutan hanya mengumumkan hilangnya kepercayaan.

## 🔐 Cadangan dan catatan masa depan

Simpan cadangan terenkripsi offline untuk kunci privat dan sertifikat pencabutan. Bagikan hanya kunci publik.

🌿 [GitHub](https://github.com/syllik) · [syllik@gmail.com](mailto:syllik@gmail.com)

<details>
<summary>✨ Catatan untuk masa depan</summary>

Suatu hari kamu akan menemukan ini lagi ketika ingin mempelajari sesuatu yang baru.

Ini gratis.

Dengan kasih,<br>
fireflў
</details>

---

[← Beranda Indonesia](README.md) · [🌍 Semua bahasa](../../README.md)
