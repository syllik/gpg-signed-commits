# 🐧 Commits Git firmados con GPG en Linux

Configura la firma OpenPGP/GPG global para tu usuario de Linux. La instalación cambia según la distribución.

## Antes de empezar

Necesitas Git, GitHub y un correo añadido y verificado en GitHub. Usa el mismo correo en GPG y en `user.email`.

> ⚠️ La clave pública se puede publicar; la clave privada/secreta, la frase de contraseña y el certificado de revocación deben permanecer privados.

## 1. Instala GnuPG

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

En otra distribución, sigue su documentación de paquetes actual e instala GnuPG 2 y un pinentry adecuado para tu terminal o escritorio. Comprueba:

~~~bash
gpg --version
which gpg
~~~

Si tu sistema usa `gpg2`, utiliza ese ejecutable de forma coherente y configúralo después en `gpg.program`.

## 2. Configura pinentry y el agente

`gpg-agent` se inicia bajo demanda. Pinentry puede ser de terminal o gráfico; no presupongas GNOME, KDE, Wayland, X11 ni systemd.

~~~bash
which pinentry
which pinentry-curses
which pinentry-gtk-2
which pinentry-qt
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

Si eliges uno concreto, añade su ruta real a `~/.gnupg/gpg-agent.conf`:

~~~text
pinentry-program /THE/ACTUAL/PATH/TO/PINENTRY
~~~

Conserva las líneas existentes y reinicia el agente:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

Añade `export GPG_TTY=$(tty)` a `~/.bashrc` o `~/.zshrc`, según tu shell, y abre una nueva sesión.

## 3. Genera la clave

~~~bash
gpg --full-generate-key
~~~

Elige RSA 4096 con capacidad de firma si está disponible, una caducidad renovable, una frase fuerte, `YOUR_NAME` y `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub debe poder relacionar el correo del committer, el UID GPG y tu correo verificado.

## 4. Busca la huella

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

Copia la huella completa de la entrada `sec` y úsala como `YOUR_GPG_FINGERPRINT`.

## 5. Exporta la clave pública

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Copia todo el bloque público entre `-----BEGIN PGP PUBLIC KEY BLOCK-----` y `-----END PGP PUBLIC KEY BLOCK-----`. No compartas material secreto.

## 6. Añade la clave a GitHub

Abre **Settings → Access → SSH and GPG keys → New GPG key**, pega la clave pública y añádela. Sigue la [documentación oficial](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 7. Identidad global

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` afecta a los repositorios de este usuario salvo anulaciones locales.

## 8. Firma OpenPGP global

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Si usas `gpg2`, cambia la ruta. Comprueba el formato:

~~~bash
git config --global --get gpg.format
~~~

Si es `ssh`, elimina la anulación solo si quieres volver a `openpgp`:

~~~bash
git config --global --unset gpg.format
~~~

## 9. Configuración local

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

Una entrada `file:.git/config` tiene prioridad. Elimina una identidad local con:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Prueba

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Debe aparecer `gpg: Good signature from ...`. Después de `git push`, GitHub debería mostrar `Verified`. La comprobación local no sustituye la asociación de clave y correo de GitHub.

## 🛠️ Problemas frecuentes

- **Pinentry y agent:** comprueba la implementación, `pinentry-program`, `GPG_TTY`, permisos de `~/.gnupg` y reinicia con `gpgconf --kill gpg-agent`.
- **Varias instalaciones:** compara `which gpg`, `which gpg2` y `git config --show-origin --get gpg.program`; Git debe usar el keyring con la clave privada.
- **Terminal, GUI o IDE:** compara PATH, Git y GPG del IDE con la terminal.
- **Sin Verified:** revisa la clave pública cargada, el correo verificado, `user.email` y que `gpg.format` no sea `ssh`.
- **Caducidad, pérdida o revocación:** renueva la clave o crea otra si falta la privada. Una clave pública no puede recuperar la privada; la revocación solo comunica que deja de ser confiable.

## 🔐 Copia de seguridad y futuro

Guarda cifradas y sin conexión la clave privada y el certificado de revocación. Solo comparte la clave pública.

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume) · [Telegram — @syllik](https://t.me/syllik)

<details>
<summary>✨ Una nota para el futuro</summary>

Algún día volverás a encontrar esto, cuando quieras aprender algo nuevo.

Es gratis.

Con cariño,<br>
fireflў
</details>

---

[← Inicio en español](README.md) · [🌍 Todos los idiomas](../../README.md)
