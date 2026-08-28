# 🪟 Commits Git signés avec GPG sous Windows

Ce parcours utilise Gpg4win et Kleopatra. Git for Windows fournit Git Bash ; PowerShell et Command Prompt ont une syntaxe différente.

## Préparation et sécurité

Installez [Gpg4win](https://www.gpg4win.org/download.html) et [Git for Windows](https://git-scm.com/download/win). Préparez une adresse e-mail vérifiée sur GitHub.

> ⚠️ Seule la clé publique est exportée. Ne partagez jamais la clé privée/secrète, la phrase secrète ou le certificat de révocation.

## 1. Installer et repérer GPG

Dans PowerShell, Command Prompt ou Git Bash :

~~~text
gpg --version
~~~

PowerShell :

~~~powershell
Get-Command gpg -All
Get-Command git -All
~~~

Command Prompt ou Git Bash :

~~~text
where gpg
where git
~~~

Utilisez le `gpg.exe` de l'installation Gpg4win que Kleopatra utilise. N'employez pas `/usr/local/bin/gpg` ni les réglages macOS `GPG_TTY`.

## 2. Créer la clé

Dans Kleopatra : **File → New Certificate → OpenPGP**. Ou dans un shell :

~~~text
gpg --full-generate-key
~~~

Choisissez RSA 4096 signable si proposé, une expiration renouvelable, une phrase forte, `YOUR_NAME` et `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub rapproche l'e-mail du committer, le UID et le courrier vérifié du compte.

## 3. Trouver l'empreinte

~~~text
gpg --list-secret-keys --keyid-format=long
~~~

Copiez l'empreinte complète sous `sec` comme `YOUR_GPG_FINGERPRINT`.

## 4. Exporter la clé publique

~~~text
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Copiez entre `-----BEGIN PGP PUBLIC KEY BLOCK-----` et `-----END PGP PUBLIC KEY BLOCK-----`. C'est public ; ne partagez jamais de matériel secret.

## 5. Ajouter à GitHub

Ouvrez **Settings → Access → SSH and GPG keys → New GPG key**, collez la clé publique et cliquez sur **Add GPG key**. Suivez la [page officielle](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 6. Identité globale

~~~text
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` concerne tous les dépôts de cet utilisateur Windows sauf remplacement local.

## 7. Choisir le GPG de Git

Dans PowerShell ou Command Prompt, remplacez le chemin par celui de `Get-Command gpg` ou `where gpg` :

~~~powershell
git config --global gpg.program "C:/Program Files/GnuPG/bin/gpg.exe"
~~~

Les barres obliques fonctionnent avec Git for Windows. Puis :

~~~text
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
git config --global --get gpg.format
~~~

Si la valeur est `ssh` et que vous voulez GPG :

~~~text
git config --global --unset gpg.format
~~~

## 8. Vérifier les remplacements locaux

~~~text
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` est prioritaire. Dans le dépôt :

~~~text
git config --unset user.name
git config --unset user.email
~~~

## 9. Tester

~~~text
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Kleopatra ou pinentry demande la phrase secrète. Recherchez `gpg: Good signature from ...`, faites `git push` et contrôlez `Verified` sur GitHub.

## 🛠️ Dépannage

- **Plusieurs gpg.exe :** comparez `Get-Command gpg -All` / `where gpg` avec `gpg.program` et utilisez la même installation que Kleopatra.
- **Fenêtre pinentry absente :** ouvrez Kleopatra puis `gpgconf --kill gpg-agent`. Windows n'a pas besoin de `GPG_TTY`.
- **IDE :** configurez le même Git et le même GPG que ceux qui fonctionnent dans le terminal.
- **Pas de Verified :** vérifiez clé publique, `user.email`, adresse vérifiée, `gpg.format` et la signature locale.
- **Clé expirée/perdue/révoquée :** renouvelez ou créez une clé. La publique ne restaure pas la privée ; la révocation signale seulement la perte de confiance.

## 🔐 Sauvegarde et note pour l'avenir

Avec Kleopatra ou GnuPG, conservez hors ligne une copie chiffrée de la clé privée et du certificat de révocation. Partagez uniquement la clé publique.

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume) · [Telegram — @syllik](https://t.me/syllik)

<details>
<summary>✨ Une note pour l'avenir</summary>

Vous retrouverez ceci un jour, lorsque vous voudrez apprendre quelque chose de nouveau.

C'est gratuit.

Avec affection,<br>
fireflў
</details>

---

[← Accueil français](README.md) · [🌍 Tous les langages](../../README.md)
