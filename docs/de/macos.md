# 🍎 GPG-signierte Git-Commits unter macOS

Richte OpenPGP/GPG-Signaturen für alle lokalen Git-Repositories des aktuellen macOS-Benutzers ein.

## Vorbereitung und Sicherheit

Du brauchst Git, Homebrew, ein GitHub-Konto und eine bei GitHub hinzugefügte und bestätigte E-Mail-Adresse. Verwende diese Adresse sowohl in der GPG-UID als auch in `user.email`.

> ⚠️ Der **öffentliche** Schlüssel darf zu GitHub hochgeladen werden. Privater/geheimer Schlüssel, Passphrase und Inhalt des Widerrufszertifikats dürfen niemals geteilt werden.

## 1. GnuPG installieren und prüfen

Das Homebrew-Präfix hängt vom Mac ab. Installiere GnuPG und pinentry-mac und ermittle die echten Pfade:

~~~bash
brew install gnupg pinentry-mac
gpg --version
which gpg
which pinentry-mac
~~~

Vertraue auf die Ausgabe von `which`, nicht auf vermutete Pfade wie `/usr/local` oder `/opt/homebrew`.

## 2. pinentry und gpg-agent einrichten

`gpg-agent` verwaltet geheime Schlüssel und fragt über pinentry nach der Passphrase. Die Konfiguration liegt in `~/.gnupg/gpg-agent.conf`.

~~~bash
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

Wenn die Datei fehlt, füge mit dem Pfad aus `which pinentry-mac` diese Zeile ein:

~~~text
pinentry-program /THE/ACTUAL/PATH/FROM/which-pinentry-mac
~~~

Wenn sie bereits existiert, bearbeite nur diese Zeile und behalte alle anderen Einstellungen. Starte den Agent neu und setze das aktuelle Terminal:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

Füge für zsh `export GPG_TTY=$(tty)` zu `~/.zshrc` hinzu und führe `source ~/.zshrc` aus.

## 3. Schlüssel erzeugen

~~~bash
gpg --full-generate-key
~~~

Wähle, falls angeboten, RSA 4096 mit Signaturfunktion, ein erneuerbares Ablaufdatum, eine starke Passphrase sowie `YOUR_NAME` und `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub verbindet die Committer-E-Mail mit einer UID des Schlüssels und einer bestätigten E-Mail im Konto; die UID allein ersetzt nicht die Git-Identität.

## 4. Vollständigen Fingerabdruck finden

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

Kopiere den vollständigen Fingerabdruck unter dem Eintrag `sec` als `YOUR_GPG_FINGERPRINT`. Zum Signieren ist der private Schlüssel nötig.

## 5. Nur den öffentlichen Schlüssel exportieren

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Kopiere den gesamten Block zwischen `-----BEGIN PGP PUBLIC KEY BLOCK-----` und `-----END PGP PUBLIC KEY BLOCK-----`. Das ist der öffentliche Schlüssel. Teile niemals geheimes Schlüsselmaterial.

## 6. Zu GitHub hinzufügen

Öffne in GitHub **Settings → Access → SSH and GPG keys → New GPG key**, vergib einen Titel, füge den öffentlichen Schlüssel ein und wähle **Add GPG key**. Bei Änderungen hilft die [offizielle GitHub-Anleitung](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 7. Globale Git-Identität festlegen

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` gilt für alle Repositories dieses macOS-Benutzers, sofern kein lokaler Wert ihn überschreibt.

## 8. Globale OpenPGP-Signatur aktivieren

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Prüfe ein eventuell gesetztes Format:

~~~bash
git config --global --get gpg.format
~~~

Wenn der Wert `ssh` ist und du GPG verwenden möchtest, entferne die globale Vorgabe und kehre zu `openpgp` zurück:

~~~bash
git config --global --unset gpg.format
~~~

## 9. Lokale Überschreibungen prüfen

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
git config --show-origin --get gpg.format
~~~

`file:.git/config` hat Vorrang vor der globalen Konfiguration. Entferne im Repository eine unerwünschte lokale Identität ohne `--global`:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Signierten Commit testen

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Erwarte eine Meldung wie `gpg: Good signature from ...`. Führe `git push` aus und öffne den Commit auf GitHub; dort sollte `Verified` erscheinen. Die lokale kryptografische Prüfung und die Zuordnung von Schlüssel/E-Mail zum GitHub-Konto sind zwei verschiedene Prüfungen.

## 🛠️ Fehlerbehebung

- **No pinentry:** Bei `gpg: agent_genkey failed: No pinentry` und `Key generation failed: No pinentry` installiere `pinentry-mac`, finde es mit `which pinentry-mac`, ergänze `pinentry-program` ohne andere Zeilen zu löschen und starte den Agent neu.
- **Homebrew-Berechtigungen:** Ändere nur das exakt im Fehler genannte Verzeichnis mit `sudo chown -R "$(whoami)" "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"` und `chmod u+w "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"`. Niemals pauschal `/usr/local` oder `/opt/homebrew` übernehmen.
- **Mehrere GPGs oder IDE:** Vergleiche `which gpg`, `gpg.program` sowie Git/GPG der IDE; alle müssen den Keyring mit dem privaten Schlüssel verwenden.
- **Kein Verified:** Prüfe den öffentlichen Schlüssel, `user.email`, die bestätigte GitHub-E-Mail, `gpg.format` und `git log --show-signature -1`. Erneuere abgelaufene Schlüssel.
- **Verlust:** Ein öffentlicher Schlüssel kann keinen verlorenen privaten Schlüssel rekonstruieren. Stelle ein sicheres Backup wieder her oder erstelle einen neuen Schlüssel; ein Widerrufszertifikat meldet nur verlorenes Vertrauen.

## 🔐 Backup und Zukunft

Bewahre private Schlüssel und Widerrufszertifikat verschlüsselt und offline auf. Teile nur den öffentlichen Schlüssel.

🌿 [GitHub](https://github.com/syllik) · [syllik@gmail.com](mailto:syllik@gmail.com)

<details>
<summary>✨ Eine Notiz für die Zukunft</summary>

Du wirst dies eines Tages wiederfinden, wenn du etwas Neues lernen möchtest.

Es ist kostenlos.

Mit Liebe,<br>
fireflў
</details>

---

[← Deutsche Startseite](README.md) · [🌍 Alle Sprachen](../../README.md)
