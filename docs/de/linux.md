# 🐧 GPG-signierte Git-Commits unter Linux

Richte globale OpenPGP/GPG-Signaturen für deinen Linux-Benutzer ein. Der Installationsbefehl hängt von der Distribution ab.

## Vorbereitung und Sicherheit

Du brauchst Git, ein GitHub-Konto und eine bestätigte E-Mail-Adresse. Verwende sie in GPG und `user.email`. Der öffentliche Schlüssel ist teilbar; privater Schlüssel, Passphrase und Widerrufszertifikat bleiben geheim.

## 1. GnuPG installieren

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

Für andere Distributionen gilt deren aktuelle Paketdokumentation. Installiere GnuPG 2 und ein zu Terminal oder Desktop passendes pinentry. Prüfe:

~~~bash
gpg --version
which gpg
~~~

Falls deine Installation `gpg2` verwendet, nutze es konsequent und setze `gpg.program` entsprechend.

## 2. pinentry und Agent konfigurieren

`gpg-agent` startet normalerweise bei Bedarf. Wähle Terminal- oder GUI-pinentry passend zu deiner Umgebung; setze GNOME, KDE, Wayland, X11 oder systemd nicht voraus.

~~~bash
which pinentry
which pinentry-curses
which pinentry-gtk-2
which pinentry-qt
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

Bei einer festen Auswahl ergänze den echten Pfad in `~/.gnupg/gpg-agent.conf`:

~~~text
pinentry-program /THE/ACTUAL/PATH/TO/PINENTRY
~~~

Vorhandene Zeilen bleiben erhalten. Danach:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

Füge `export GPG_TTY=$(tty)` zu `~/.bashrc` oder `~/.zshrc` hinzu und öffne eine neue Shell.

## 3. Schlüssel erzeugen

~~~bash
gpg --full-generate-key
~~~

Wähle RSA 4096 mit Signaturfunktion, falls verfügbar, eine erneuerbare Laufzeit, eine starke Passphrase, `YOUR_NAME` und `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub muss Committer-E-Mail, GPG-UID und bestätigte Konto-E-Mail verbinden können.

## 4. Fingerabdruck finden

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

Kopiere den vollständigen Fingerabdruck unter `sec` als `YOUR_GPG_FINGERPRINT`.

## 5. Öffentlichen Schlüssel exportieren

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Kopiere den öffentlichen Block zwischen `-----BEGIN PGP PUBLIC KEY BLOCK-----` und `-----END PGP PUBLIC KEY BLOCK-----`. Teile kein geheimes Schlüsselmaterial.

## 6. Zu GitHub hinzufügen

Wähle **Settings → Access → SSH and GPG keys → New GPG key**, füge den öffentlichen Schlüssel ein und bestätige. Siehe die [GitHub-Dokumentation](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 7. Globale Identität

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` gilt für die Repositories dieses Benutzers, lokale Überschreibungen ausgenommen.

## 8. Globale OpenPGP-Signatur

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Prüfe:

~~~bash
git config --global --get gpg.format
~~~

Bei `ssh` entferne die Einstellung nur, wenn du zu `openpgp` zurückkehren willst:

~~~bash
git config --global --unset gpg.format
~~~

## 9. Lokale Werte prüfen

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` hat Vorrang. Entferne lokale Identitätswerte mit:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Test

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Suche `gpg: Good signature from ...`, führe `git push` aus und prüfe `Verified` bei GitHub. Die lokale Prüfung und die GitHub-Kontozuordnung sind nicht identisch.

## 🛠️ Fehlerbehebung

- **Pinentry/Agent:** Prüfe Programm, `pinentry-program`, `GPG_TTY` und die Rechte von `~/.gnupg`, dann `gpgconf --kill gpg-agent`.
- **Mehrere GPG-Installationen:** Vergleiche `which gpg`, `which gpg2` und `git config --show-origin --get gpg.program`.
- **Terminal, GUI, IDE:** Vergleiche PATH, Git und GPG der IDE mit dem Terminal.
- **Kein Verified:** Prüfe öffentlichen Schlüssel, `user.email`, bestätigte E-Mail und `gpg.format`; erneuere abgelaufene Schlüssel.
- **Privater Schlüssel verloren:** Der öffentliche Schlüssel kann ihn nicht wiederherstellen. Nutze ein Backup oder erstelle einen neuen; Widerruf meldet nur den Vertrauensverlust.

## 🔐 Backup und Zukunft

Bewahre privaten Schlüssel und Widerrufszertifikat verschlüsselt offline auf. Teile nur den öffentlichen Schlüssel.

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume)

<details>
<summary>✨ Eine Notiz für die Zukunft</summary>

Du wirst dies eines Tages wiederfinden, wenn du etwas Neues lernen möchtest.

Es ist kostenlos.

Mit Liebe,<br>
fireflў
</details>

---

[← Deutsche Startseite](README.md) · [🌍 Alle Sprachen](../../README.md)
