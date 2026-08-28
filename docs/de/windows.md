# 🪟 GPG-signierte Git-Commits unter Windows

Dieser Ablauf verwendet Gpg4win und Kleopatra. Git for Windows bietet Git Bash; PowerShell und Command Prompt haben eine eigene Pfadsyntax.

## Vorbereitung und Sicherheit

Installiere [Gpg4win](https://www.gpg4win.org/download.html) und [Git for Windows](https://git-scm.com/download/win). Verwende eine bestätigte GitHub-E-Mail-Adresse.

> ⚠️ Lade nur den öffentlichen Schlüssel hoch. Privater/geheimer Schlüssel, Passphrase und Widerrufszertifikat dürfen nie geteilt werden.

## 1. Installieren und Pfade ermitteln

In PowerShell, Command Prompt oder Git Bash:

~~~text
gpg --version
~~~

PowerShell:

~~~powershell
Get-Command gpg -All
Get-Command git -All
~~~

Command Prompt oder Git Bash:

~~~text
where gpg
where git
~~~

Verwende das `gpg.exe` aus derselben Gpg4win-Installation, deren Keyring Kleopatra nutzt. Keine Unix-Pfade wie `/usr/local/bin/gpg` und keine macOS-Konfiguration für `GPG_TTY` verwenden.

## 2. Schlüssel erzeugen

In Kleopatra: **File → New Certificate → OpenPGP**. Alternativ:

~~~text
gpg --full-generate-key
~~~

Wähle RSA 4096 mit Signatur, falls verfügbar, eine erneuerbare Laufzeit, starke Passphrase, `YOUR_NAME` und `YOUR_VERIFIED_GITHUB_EMAIL`.

## 3. Fingerabdruck finden

~~~text
gpg --list-secret-keys --keyid-format=long
~~~

Kopiere den vollständigen Fingerabdruck unter `sec` als `YOUR_GPG_FINGERPRINT`.

## 4. Öffentlichen Schlüssel exportieren

~~~text
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Kopiere den Block zwischen `-----BEGIN PGP PUBLIC KEY BLOCK-----` und `-----END PGP PUBLIC KEY BLOCK-----`. Teile kein geheimes Material.

## 5. Zu GitHub hinzufügen

Öffne **Settings → Access → SSH and GPG keys → New GPG key**, füge den öffentlichen Schlüssel ein und wähle **Add GPG key**. Siehe die [offizielle Anleitung](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 6. Globale Identität

~~~text
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` gilt für die Repositories dieses Windows-Benutzers, außer bei lokalen Überschreibungen.

## 7. Git auf das richtige GPG zeigen lassen

In PowerShell oder Command Prompt den Beispielpfad durch den Wert aus `Get-Command gpg` oder `where gpg` ersetzen:

~~~powershell
git config --global gpg.program "C:/Program Files/GnuPG/bin/gpg.exe"
~~~

Git for Windows akzeptiert diesen Windows-Pfad in Anführungszeichen. Danach:

~~~text
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
git config --global --get gpg.format
~~~

Wenn `ssh` angezeigt wird und du GPG willst:

~~~text
git config --global --unset gpg.format
~~~

## 8. Lokale Überschreibungen

~~~text
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` hat Vorrang. Im Repository:

~~~text
git config --unset user.name
git config --unset user.email
~~~

## 9. Signatur testen

~~~text
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Kleopatra oder pinentry fragt nach der Passphrase. Suche `gpg: Good signature from ...`, führe `git push` aus und prüfe `Verified` bei GitHub.

## 🛠️ Fehlerbehebung

- **Mehrere gpg.exe:** Vergleiche `Get-Command gpg -All`/`where gpg` mit `gpg.program` und verwende die Kleopatra-Installation.
- **Pinentry-Fenster fehlt:** Starte Kleopatra und führe `gpgconf --kill gpg-agent` aus. Windows benötigt `GPG_TTY` nicht.
- **IDE:** Konfiguriere dasselbe Git und GPG wie im funktionierenden Terminal.
- **Kein Verified:** Prüfe öffentlichen Schlüssel, `user.email`, bestätigte E-Mail, `gpg.format` und lokale Signatur.
- **Abgelaufener/verlorener Schlüssel:** Erneuere oder erstelle einen neuen. Der öffentliche Schlüssel stellt den privaten nicht wieder her; Widerruf meldet nur Vertrauensverlust.

## 🔐 Backup und Zukunft

Bewahre mit Kleopatra oder GnuPG private Schlüssel und Widerrufszertifikat verschlüsselt offline auf. Teile nur den öffentlichen Schlüssel.

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume) · [Telegram — @syllik](https://t.me/syllik)

<details>
<summary>✨ Eine Notiz für die Zukunft</summary>

Du wirst dies eines Tages wiederfinden, wenn du etwas Neues lernen möchtest.

Es ist kostenlos.

Mit Liebe,<br>
fireflў
</details>

---

[← Deutsche Startseite](README.md) · [🌍 Alle Sprachen](../../README.md)
