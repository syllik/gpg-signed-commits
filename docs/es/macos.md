# 🍎 Commits Git firmados con GPG en macOS

Configura la firma OpenPGP/GPG para todos los repositorios locales del usuario actual de macOS.

## Antes de empezar

Necesitas Git, Homebrew, una cuenta de GitHub y una dirección de correo añadida y verificada en GitHub. Usa esa misma dirección en el UID de GPG y en `user.email` de Git.

> ⚠️ La clave **pública** se puede subir a GitHub. La clave privada/secreta, la frase de contraseña y el contenido del certificado de revocación nunca se deben compartir.

## 1. Instala y comprueba GnuPG

Homebrew usa prefijos distintos según el Mac. Instala GnuPG y el programa de entrada de contraseña para macOS:

~~~bash
brew install gnupg pinentry-mac
gpg --version
which gpg
which pinentry-mac
~~~

La salida de `gpg --version` debe mostrar GnuPG 2.x. Usa las rutas que devuelvan los comandos; no supongas `/usr/local` ni `/opt/homebrew`.

## 2. Configura pinentry y gpg-agent

`gpg-agent` administra las claves secretas y abre pinentry cuando necesita la frase de contraseña. Su archivo es `~/.gnupg/gpg-agent.conf`.

~~~bash
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

Si el archivo no existe, añade una línea con la ruta real de `which pinentry-mac`:

~~~text
pinentry-program /THE/ACTUAL/PATH/FROM/which-pinentry-mac
~~~

Si ya existe, edítalo y conserva sus demás líneas. Reinicia el agente y configura la terminal:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

Para zsh, añade `export GPG_TTY=$(tty)` a `~/.zshrc` y ejecuta `source ~/.zshrc`. Esto permite que la terminal encuentre el aviso de contraseña.

## 3. Genera la clave

~~~bash
gpg --full-generate-key
~~~

Elige una clave RSA de 4096 bits capaz de firmar si aparece esa opción, una caducidad que puedas renovar, una frase de contraseña fuerte y los valores `YOUR_NAME` y `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub relaciona el correo del committer con un UID de la clave y con un correo verificado de tu cuenta; el UID por sí solo no configura la identidad de Git.

## 4. Busca la huella completa

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

Busca la entrada `sec` y copia la huella completa de la línea `fingerprint` como `YOUR_GPG_FINGERPRINT`. Para firmar hace falta la clave privada.

## 5. Exporta solo la clave pública

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Copia desde `-----BEGIN PGP PUBLIC KEY BLOCK-----` hasta `-----END PGP PUBLIC KEY BLOCK-----`. Es la clave pública. No exportes ni pegues material de clave secreta.

## 6. Añádela a GitHub

En GitHub ve a **Settings → Access → SSH and GPG keys → New GPG key**. Escribe un título, pega la clave pública y pulsa **Add GPG key**. Consulta la [guía oficial de GitHub](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) si cambia la interfaz.

## 7. Configura la identidad global de Git

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` se aplica a todos los repositorios de este usuario de macOS, salvo que uno tenga una configuración local distinta.

## 8. Configura la firma OpenPGP global

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Comprueba si existe un formato anterior:

~~~bash
git config --global --get gpg.format
~~~

Si devuelve `ssh` y quieres GPG, elimina la anulación para volver al formato `openpgp`:

~~~bash
git config --global --unset gpg.format
~~~

## 9. Comprueba las anulaciones locales

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` tiene prioridad sobre la configuración global. Dentro del repositorio, elimina una identidad local no deseada sin `--global`:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Prueba y verifica

En un repositorio de prueba:

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Busca `gpg: Good signature from ...`. Ejecuta `git push` y abre el commit en GitHub: debería mostrar `Verified`. La verificación criptográfica local y la asociación de cuenta/correo de GitHub son comprobaciones relacionadas, pero no idénticas.

## 🛠️ Problemas frecuentes

- **No pinentry:** si aparece `gpg: agent_genkey failed: No pinentry` y `Key generation failed: No pinentry`, instala `pinentry-mac`, localízalo con `which pinentry-mac`, añade `pinentry-program` al archivo existente, reinicia el agente y vuelve a generar la clave.
- **Homebrew sin permisos:** cambia solo el directorio exacto indicado por el error con `sudo chown -R "$(whoami)" "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"` y `chmod u+w "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"`. Nunca hagas chown recursivo de todo `/usr/local` o `/opt/homebrew`.
- **Varios GPG o IDE:** compara `which gpg` con `gpg.program` y con el Git/GPG del IDE; todos deben apuntar al keyring que contiene la clave privada.
- **GitHub no muestra Verified:** revisa la clave pública, `user.email`, el correo verificado, `gpg.format` y la firma con `git log --show-signature -1`. Si la clave caducó, renuévala.
- **Pérdida o revocación:** una clave pública no reconstruye una privada perdida. Recupera una copia segura o crea otra clave. El certificado de revocación anuncia que una clave ya no es confiable; no sirve como copia de recuperación.

## 🔐 Copia de seguridad y futuro

Guarda una copia cifrada y sin conexión de la clave privada y del certificado de revocación. Comparte únicamente la clave pública.

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
