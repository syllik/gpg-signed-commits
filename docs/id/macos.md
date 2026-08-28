# 🍎 Commit Git bertanda tangan GPG di macOS

Atur penandatanganan OpenPGP/GPG untuk semua repositori Git lokal milik pengguna macOS saat ini.

## Persiapan dan keamanan

Kamu memerlukan Git, Homebrew, akun GitHub, dan alamat email yang sudah ditambahkan serta diverifikasi di GitHub. Gunakan alamat yang sama pada UID GPG dan `user.email` Git.

> ⚠️ Kunci **publik** aman untuk diunggah ke GitHub. Kunci privat/rahasia, passphrase, dan isi sertifikat pencabutan tidak boleh dibagikan.

## 1. Instal dan periksa GnuPG

Prefix Homebrew berbeda menurut jenis Mac. Instal GnuPG dan pinentry macOS, lalu lihat path yang benar:

~~~bash
brew install gnupg pinentry-mac
gpg --version
which gpg
which pinentry-mac
~~~

Jangan menebak `/usr/local` atau `/opt/homebrew`. Gunakan hasil dari `which`.

## 2. Atur pinentry dan gpg-agent

`gpg-agent` mengelola kunci rahasia dan meminta passphrase melalui pinentry. File konfigurasinya adalah `~/.gnupg/gpg-agent.conf`.

~~~bash
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

Jika file belum ada, tambahkan path nyata dari `which pinentry-mac`:

~~~text
pinentry-program /THE/ACTUAL/PATH/FROM/which-pinentry-mac
~~~

Jika sudah ada, edit hanya baris itu dan pertahankan pengaturan lainnya. Mulai ulang agent dan beri tahu terminal TTY yang aktif:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

Untuk zsh, tambahkan `export GPG_TTY=$(tty)` ke `~/.zshrc` lalu jalankan `source ~/.zshrc`.

## 3. Buat kunci

~~~bash
gpg --full-generate-key
~~~

Pilih RSA 4096 yang dapat digunakan untuk tanda tangan bila tersedia, masa berlaku yang dapat diperpanjang, passphrase kuat, `YOUR_NAME`, dan `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub mencocokkan email committer dengan UID kunci dan email terverifikasi akun; UID saja bukan identitas Git.

## 4. Temukan fingerprint lengkap

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

Salin fingerprint lengkap di bawah baris `sec` sebagai `YOUR_GPG_FINGERPRINT`. Penandatanganan memerlukan kunci privat.

## 5. Ekspor hanya kunci publik

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Salin seluruh blok dari `-----BEGIN PGP PUBLIC KEY BLOCK-----` sampai `-----END PGP PUBLIC KEY BLOCK-----`. Itu adalah kunci publik. Jangan mengekspor atau membagikan material kunci rahasia.

## 6. Tambahkan ke GitHub

Di GitHub buka **Settings → Access → SSH and GPG keys → New GPG key**, beri judul, tempel kunci publik, lalu pilih **Add GPG key**. Gunakan [dokumentasi resmi GitHub](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) jika label berubah.

## 7. Atur identitas Git global

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` berlaku untuk semua repositori pengguna macOS ini kecuali ada pengaturan lokal.

## 8. Atur penandatanganan OpenPGP global

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Periksa format lama:

~~~bash
git config --global --get gpg.format
~~~

Jika nilainya `ssh` dan kamu ingin GPG, hapus override tersebut agar kembali ke `openpgp`:

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
~~~

Nilai dari `file:.git/config` mengalahkan konfigurasi global. Dari dalam repositori, hapus identitas lokal yang tidak diinginkan tanpa `--global`:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Uji commit bertanda tangan

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Cari pesan `gpg: Good signature from ...`. Jalankan `git push` dan buka commit di GitHub; lencana `Verified` seharusnya muncul. Verifikasi kriptografis lokal dan pengaitan akun/email GitHub adalah pemeriksaan yang berbeda.

## 🛠️ Pemecahan masalah

- **No pinentry:** jika muncul `gpg: agent_genkey failed: No pinentry` dan `Key generation failed: No pinentry`, instal `pinentry-mac`, cari path dengan `which pinentry-mac`, tambahkan `pinentry-program` tanpa menimpa file, mulai ulang agent, lalu coba lagi.
- **Izin Homebrew:** ubah hanya direktori persis yang disebut error dengan `sudo chown -R "$(whoami)" "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"` dan `chmod u+w "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"`. Jangan melakukan chown rekursif pada seluruh `/usr/local` atau `/opt/homebrew`.
- **Beberapa GPG atau IDE:** bandingkan `which gpg`, `gpg.program`, PATH, Git, dan GPG yang dipakai IDE. Semuanya harus melihat keyring yang berisi kunci privat.
- **GitHub tidak menampilkan Verified:** pastikan kunci publik benar, `user.email` adalah UID kunci dan email GitHub terverifikasi, serta `gpg.format` bukan `ssh`.
- **Kunci kedaluwarsa/hilang:** perpanjang atau buat kunci baru. Kunci publik di GitHub tidak dapat memulihkan kunci privat yang hilang. Sertifikat pencabutan hanya menyatakan bahwa kunci tidak lagi tepercaya.

## 🔐 Cadangan dan catatan masa depan

Simpan cadangan terenkripsi dan offline untuk kunci privat serta sertifikat pencabutan. Bagikan hanya kunci publik.

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
