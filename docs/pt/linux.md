# 🐧 Commits Git assinados com GPG no Linux

Configure a assinatura OpenPGP/GPG global para seu usuário Linux. A instalação depende da distribuição.

## Antes de começar

Tenha Git, uma conta no GitHub e um e-mail verificado. Use esse endereço no GPG e em `user.email`.

> ⚠️ A chave pública pode ser publicada; a chave privada/secreta, a senha e o certificado de revogação devem permanecer privados.

## 1. Instale o GnuPG

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

Em outra distribuição, consulte a documentação atual e instale GnuPG 2 e um pinentry adequado ao seu terminal ou desktop. Verifique:

~~~bash
gpg --version
which gpg
~~~

Se o sistema usar `gpg2`, use-o de forma consistente e configure-o em `gpg.program`.

## 2. Configure pinentry e o agente

`gpg-agent` normalmente inicia sob demanda. Escolha um pinentry de terminal ou gráfico; não presuma GNOME, KDE, Wayland, X11 ou systemd.

~~~bash
which pinentry
which pinentry-curses
which pinentry-gtk-2
which pinentry-qt
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

Se precisar fixar um programa, adicione seu caminho real a `~/.gnupg/gpg-agent.conf`:

~~~text
pinentry-program /THE/ACTUAL/PATH/TO/PINENTRY
~~~

Preserve as linhas existentes e reinicie:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

Adicione `export GPG_TTY=$(tty)` a `~/.bashrc` ou `~/.zshrc` e abra um novo shell.

## 3. Gere a chave

~~~bash
gpg --full-generate-key
~~~

Escolha RSA 4096 com assinatura, se oferecido, uma expiração renovável, uma senha forte, `YOUR_NAME` e `YOUR_VERIFIED_GITHUB_EMAIL`. O GitHub precisa relacionar o e-mail do committer, o UID GPG e o e-mail verificado.

## 4. Encontre a fingerprint

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

Copie a fingerprint completa da entrada `sec` como `YOUR_GPG_FINGERPRINT`.

## 5. Exporte a chave pública

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Copie o bloco entre `-----BEGIN PGP PUBLIC KEY BLOCK-----` e `-----END PGP PUBLIC KEY BLOCK-----`. Não compartilhe material secreto.

## 6. Adicione ao GitHub

Abra **Settings → Access → SSH and GPG keys → New GPG key**, cole a chave pública e adicione-a. Veja a [documentação do GitHub](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 7. Identidade global

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` se aplica aos repositórios deste usuário, salvo substituições locais.

## 8. Assinatura OpenPGP global

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Confira `gpg.format`:

~~~bash
git config --global --get gpg.format
~~~

Se for `ssh`, remova-o somente para voltar a `openpgp`:

~~~bash
git config --global --unset gpg.format
~~~

## 9. Substituições locais

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` prevalece. Remova uma identidade local com:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Teste

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Procure `gpg: Good signature from ...`, faça `git push` e confirme `Verified` no GitHub. A validação local não substitui a associação da chave e do e-mail na conta.

## 🛠️ Solução de problemas

- **Sem pinentry/agente:** verifique o programa, `pinentry-program`, `GPG_TTY` e as permissões de `~/.gnupg`; depois execute `gpgconf --kill gpg-agent`.
- **Várias instalações:** compare `which gpg`, `which gpg2` e `git config --show-origin --get gpg.program`.
- **Terminal e IDE diferentes:** compare PATH, Git e GPG usados pelo IDE.
- **Sem Verified:** confira a chave pública, `user.email`, o e-mail verificado e `gpg.format`; renove chaves expiradas.
- **Chave perdida/revogada:** a pública não recupera a privada. Restaure um backup ou crie outra; a revogação só comunica perda de confiança.

## 🔐 Backup e futuro

Guarde uma cópia offline e criptografada da chave privada e do certificado de revogação. Compartilhe somente a chave pública.

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume)

<details>
<summary>✨ Uma nota para o futuro</summary>

Você encontrará isto novamente algum dia, quando quiser aprender algo novo.

É gratuito.

Com carinho,<br>
fireflў
</details>

---

[← Início em português](README.md) · [🌍 Todos os idiomas](../../README.md)
