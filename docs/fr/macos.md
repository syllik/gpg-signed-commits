# 🍎 Commits Git signés avec GPG sur macOS

Configurez la signature OpenPGP/GPG pour tous les dépôts Git locaux de l'utilisateur macOS actuel.

## Préparation et sécurité

Vous avez besoin de Git, de Homebrew, d'un compte GitHub et d'une adresse e-mail ajoutée et vérifiée sur GitHub. Utilisez cette adresse dans l'identité GPG et dans `user.email`.

> ⚠️ La clé **publique** peut être exportée vers GitHub. La clé privée/secrète, la phrase secrète et le contenu du certificat de révocation ne doivent jamais être partagés.

## 1. Installer et vérifier GnuPG

Le préfixe Homebrew dépend du Mac. Installez GnuPG et pinentry pour macOS, puis découvrez les chemins réels :

~~~bash
brew install gnupg pinentry-mac
gpg --version
which gpg
which pinentry-mac
~~~

La sortie de `gpg --version` doit indiquer GnuPG 2.x. Ne supposez ni `/usr/local` ni `/opt/homebrew`.

## 2. Configurer pinentry et gpg-agent

`gpg-agent` gère les clés secrètes et demande la phrase secrète via pinentry. Le fichier est `~/.gnupg/gpg-agent.conf`.

~~~bash
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

Dans un fichier nouveau, ajoutez le chemin renvoyé par `which pinentry-mac` :

~~~text
pinentry-program /THE/ACTUAL/PATH/FROM/which-pinentry-mac
~~~

Si le fichier existe déjà, modifiez uniquement cette ligne et conservez les autres réglages. Redémarrez l'agent et indiquez le terminal courant :

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

Pour zsh, ajoutez `export GPG_TTY=$(tty)` à `~/.zshrc` puis exécutez `source ~/.zshrc`.

## 3. Créer la clé

~~~bash
gpg --full-generate-key
~~~

Choisissez RSA 4096 avec capacité de signature si proposé, une expiration que vous pourrez renouveler, une phrase secrète forte, `YOUR_NAME` et `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub associe l'e-mail du committer à un UID de la clé et à un e-mail vérifié du compte ; le UID seul ne remplace pas l'identité Git.

## 4. Trouver l'empreinte complète

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

Sous l'entrée `sec`, copiez l'empreinte complète de la ligne `fingerprint` et utilisez `YOUR_GPG_FINGERPRINT`.

## 5. Exporter uniquement la clé publique

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Copiez tout le bloc entre `-----BEGIN PGP PUBLIC KEY BLOCK-----` et `-----END PGP PUBLIC KEY BLOCK-----`. C'est la clé publique ; ne partagez jamais du matériel de clé secrète.

## 6. Ajouter la clé à GitHub

Dans GitHub, ouvrez **Settings → Access → SSH and GPG keys → New GPG key**, donnez un titre à la clé, collez la clé publique et cliquez sur **Add GPG key**. Consultez la [documentation officielle](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) si l'interface change.

## 7. Définir l'identité Git globale

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` s'applique à tous les dépôts de cet utilisateur macOS, sauf remplacement local.

## 8. Activer la signature OpenPGP globale

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Vérifiez un ancien format :

~~~bash
git config --global --get gpg.format
~~~

Si la valeur est `ssh` et que vous voulez GPG, supprimez ce remplacement pour revenir à `openpgp` :

~~~bash
git config --global --unset gpg.format
~~~

## 9. Vérifier les remplacements locaux

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

Une valeur provenant de `file:.git/config` remplace la valeur globale. Dans le dépôt, retirez une identité locale avec :

~~~bash
git config --unset user.name
git config --unset user.email
~~~

N'ajoutez pas `--global` à ces commandes.

## 10. Tester et vérifier

Dans un dépôt de test :

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Vous devez voir `gpg: Good signature from ...`. Faites `git push` puis ouvrez le commit sur GitHub : le badge devrait être `Verified`. La vérification cryptographique locale et l'association du compte GitHub sont liées, mais différentes.

## 🛠️ Dépannage

- **No pinentry :** si vous voyez `gpg: agent_genkey failed: No pinentry` et `Key generation failed: No pinentry`, installez `pinentry-mac`, utilisez `which pinentry-mac`, ajoutez `pinentry-program` sans écraser le fichier, redémarrez l'agent et réessayez.
- **Homebrew et permissions :** ne faites jamais de chown récursif sur `/usr/local` ou `/opt/homebrew`. Modifiez uniquement le dossier exact signalé, avec `sudo chown -R "$(whoami)" "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"` puis `chmod u+w "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"`.
- **Plusieurs GPG ou IDE :** comparez `which gpg`, `gpg.program` et le Git/GPG de l'IDE ; le même keyring doit contenir la clé privée.
- **Pas de Verified :** contrôlez la clé publique, `user.email`, l'e-mail vérifié, `gpg.format` et `git log --show-signature -1`. Renouvelez une clé expirée.
- **Clé perdue ou révoquée :** une clé publique ne restaure pas une clé privée. Récupérez une sauvegarde ou créez une nouvelle clé ; le certificat de révocation annonce seulement qu'une clé n'est plus fiable.

## 🔐 Sauvegarde et note pour l'avenir

Conservez hors ligne une copie chiffrée de la clé privée et du certificat de révocation. Ne partagez que la clé publique.

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume)

<details>
<summary>✨ Une note pour l'avenir</summary>

Vous retrouverez ceci un jour, lorsque vous voudrez apprendre quelque chose de nouveau.

C'est gratuit.

Avec affection,<br>
fireflў
</details>

---

[← Accueil français](README.md) · [🌍 Tous les langages](../../README.md)
