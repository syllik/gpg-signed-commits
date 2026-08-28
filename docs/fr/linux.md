# 🐧 Commits Git signés avec GPG sous Linux

Configurez la signature OpenPGP/GPG globale pour votre utilisateur Linux. La commande d'installation dépend de la distribution.

## Préparation et sécurité

Préparez Git, GitHub et une adresse e-mail vérifiée sur GitHub. La clé publique est partageable ; la clé privée/secrète, la phrase secrète et le certificat de révocation restent privés.

## 1. Installer GnuPG

Debian/Ubuntu :

~~~bash
sudo apt update
sudo apt install gnupg
~~~

Fedora :

~~~bash
sudo dnf install gnupg2
~~~

Arch Linux :

~~~bash
sudo pacman -Syu gnupg
~~~

Pour une autre distribution, suivez sa documentation actuelle et installez GnuPG 2 avec un pinentry adapté. Vérifiez :

~~~bash
gpg --version
which gpg
~~~

Si seul `gpg2` contient vos clés, utilisez-le et pointez `gpg.program` vers ce programme.

## 2. Configurer pinentry et l'agent

`gpg-agent` démarre normalement à la demande. Choisissez un pinentry de terminal ou graphique selon votre environnement ; ne supposez pas GNOME, KDE, Wayland, X11 ou systemd.

~~~bash
which pinentry
which pinentry-curses
which pinentry-gtk-2
which pinentry-qt
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

Si vous fixez un programme, ajoutez son chemin réel dans `~/.gnupg/gpg-agent.conf` :

~~~text
pinentry-program /THE/ACTUAL/PATH/TO/PINENTRY
~~~

Préservez les réglages existants, puis :

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

Ajoutez `export GPG_TTY=$(tty)` à `~/.bashrc` ou `~/.zshrc` selon votre shell, puis ouvrez un nouveau shell.

## 3. Créer la clé

~~~bash
gpg --full-generate-key
~~~

Choisissez RSA 4096 signable si proposé, une expiration renouvelable, une phrase secrète forte, `YOUR_NAME` et `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub doit relier l'e-mail du committer, le UID GPG et l'e-mail vérifié.

## 4. Trouver l'empreinte

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

Copiez l'empreinte complète sous `sec` comme `YOUR_GPG_FINGERPRINT`.

## 5. Exporter la clé publique

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Copiez le bloc public entre `-----BEGIN PGP PUBLIC KEY BLOCK-----` et `-----END PGP PUBLIC KEY BLOCK-----`. Ne partagez aucun matériel secret.

## 6. Ajouter à GitHub

Utilisez **Settings → Access → SSH and GPG keys → New GPG key**, collez la clé publique et ajoutez-la. Voir la [documentation GitHub](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 7. Identité globale

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` s'applique aux dépôts de cet utilisateur sauf remplacement local.

## 8. Signature globale OpenPGP

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Avec `gpg2`, adaptez le chemin. Contrôlez le format :

~~~bash
git config --global --get gpg.format
~~~

Si c'est `ssh`, retirez-le seulement pour revenir à `openpgp` :

~~~bash
git config --global --unset gpg.format
~~~

## 9. Vérifier les remplacements

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
git config --show-origin --get gpg.format
~~~

`file:.git/config` est prioritaire. Supprimez une identité locale avec :

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Tester et vérifier

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Recherchez `gpg: Good signature from ...`, puis faites `git push`. Le commit GitHub devrait porter `Verified` ; le contrôle local et l'association GitHub ne sont pas exactement la même chose.

## 🛠️ Dépannage

- **Pas de pinentry :** vérifiez le programme, `pinentry-program`, `GPG_TTY`, les permissions de `~/.gnupg`, puis `gpgconf --kill gpg-agent`.
- **Plusieurs GPG :** comparez `which gpg`, `which gpg2` et `git config --show-origin --get gpg.program`.
- **Terminal/GUI/IDE :** comparez PATH, Git et GPG utilisés par l'IDE.
- **Pas de Verified :** contrôlez la clé publique, `user.email`, l'adresse vérifiée et `gpg.format` ; renouvelez une clé expirée.
- **Clé perdue :** la clé publique ne peut pas recréer la privée. Restaurez une sauvegarde ou créez une nouvelle clé ; la révocation signale la perte de confiance.

## 🔐 Sauvegarde et note pour l'avenir

Gardez une sauvegarde chiffrée hors ligne de la clé privée et du certificat de révocation. Partagez seulement la clé publique.

🌿 [GitHub](https://github.com/syllik) · [syllik@gmail.com](mailto:syllik@gmail.com)

<details>
<summary>✨ Une note pour l'avenir</summary>

Vous retrouverez ceci un jour, lorsque vous voudrez apprendre quelque chose de nouveau.

C'est gratuit.

Avec affection,<br>
fireflў
</details>

---

[← Accueil français](README.md) · [🌍 Tous les langages](../../README.md)
