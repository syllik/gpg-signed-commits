# 🪟 Commits Git firmados con GPG en Windows

Este flujo usa Gpg4win y Kleopatra. Git for Windows aporta Git Bash; PowerShell y Command Prompt son shells distintos.

## Antes de empezar

Instala [Gpg4win](https://www.gpg4win.org/download.html) y [Git for Windows](https://git-scm.com/download/win). Usa un correo añadido y verificado en GitHub.

> ⚠️ Sube únicamente la clave pública. No compartas la clave privada/secreta, la frase de contraseña ni el certificado de revocación.

## 1. Instala y localiza GPG

En PowerShell, Command Prompt o Git Bash:

~~~text
gpg --version
~~~

PowerShell:

~~~powershell
Get-Command gpg -All
Get-Command git -All
~~~

Command Prompt o Git Bash:

~~~text
where gpg
where git
~~~

Usa el `gpg.exe` de la instalación Gpg4win cuyo keyring ve Kleopatra. No uses rutas Unix como `/usr/local/bin/gpg` ni copies `GPG_TTY` de macOS.

## 2. Genera la clave

En Kleopatra: **File → New Certificate → OpenPGP**. O en cualquier shell:

~~~text
gpg --full-generate-key
~~~

Elige RSA 4096 con firma si está disponible, una caducidad renovable, una frase fuerte, `YOUR_NAME` y `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub relaciona el correo del committer con el UID y el correo verificado de la cuenta.

## 3. Busca la huella

~~~text
gpg --list-secret-keys --keyid-format=long
~~~

Copia la huella completa bajo `sec` y úsala como `YOUR_GPG_FINGERPRINT`.

## 4. Exporta la clave pública

~~~text
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Copia el bloque entre `-----BEGIN PGP PUBLIC KEY BLOCK-----` y `-----END PGP PUBLIC KEY BLOCK-----`. Es público; nunca compartas material secreto.

## 5. Añádela a GitHub

Ve a **Settings → Access → SSH and GPG keys → New GPG key**, pega la clave pública y pulsa **Add GPG key**. Consulta la [guía oficial](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 6. Identidad global

~~~text
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` afecta a todos los repositorios de este usuario de Windows salvo anulaciones locales.

## 7. Configura el GPG que usará Git

En PowerShell o Command Prompt, sustituye la ruta por la que devolvió `Get-Command gpg` o `where gpg`:

~~~powershell
git config --global gpg.program "C:/Program Files/GnuPG/bin/gpg.exe"
~~~

Git for Windows acepta esta ruta Windows entre comillas. Luego:

~~~text
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
git config --global --get gpg.format
~~~

Si muestra `ssh` y quieres GPG:

~~~text
git config --global --unset gpg.format
~~~

## 8. Revisa anulaciones locales

~~~text
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
git config --show-origin --get gpg.format
~~~

`file:.git/config` tiene prioridad. Dentro del repositorio:

~~~text
git config --unset user.name
git config --unset user.email
~~~

## 9. Prueba y verifica

~~~text
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Kleopatra o pinentry pedirá la frase. Busca `gpg: Good signature from ...`, ejecuta `git push` y comprueba `Verified` en GitHub.

## 🛠️ Problemas frecuentes

- **Varios gpg.exe:** compara `Get-Command gpg -All` / `where gpg` con `gpg.program` y usa la instalación que comparte keyring con Kleopatra.
- **Ventana pinentry ausente:** abre Kleopatra y ejecuta `gpgconf --kill gpg-agent`. Windows no necesita `GPG_TTY`.
- **IDE:** configura el IDE con el mismo Git y GPG que funcionan en la terminal.
- **GitHub sin Verified:** revisa clave pública, `user.email`, correo verificado, `gpg.format` y `git log --show-signature -1`.
- **Clave caducada, perdida o revocada:** renueva o crea otra. La pública no recupera la privada; la revocación anuncia que la clave ya no es confiable.

## 🔐 Copia de seguridad y futuro

Con Kleopatra o GnuPG, conserva una copia cifrada y sin conexión de la clave privada y del certificado de revocación. Comparte solo la clave pública.

🌿 [GitHub](https://github.com/syllik) · [syllik@gmail.com](mailto:syllik@gmail.com)

<details>
<summary>✨ Una nota para el futuro</summary>

Algún día volverás a encontrar esto, cuando quieras aprender algo nuevo.

Es gratis.

Con cariño,<br>
fireflў
</details>

---

[← Inicio en español](README.md) · [🌍 Todos los idiomas](../../README.md)
