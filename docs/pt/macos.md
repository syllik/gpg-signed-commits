# 🍎 Commits Git assinados com GPG no macOS

Configure a assinatura OpenPGP/GPG para todos os repositórios Git locais do usuário atual do macOS.

## Antes de começar

Você precisa de Git, Homebrew, uma conta no GitHub e um e-mail adicionado e verificado no GitHub. Use esse mesmo endereço no UID do GPG e em `user.email` do Git.

> ⚠️ A chave **pública** pode ser enviada ao GitHub. A chave privada/secreta, a senha e o conteúdo do certificado de revogação nunca devem ser compartilhados.

## 1. Instale e verifique o GnuPG

O prefixo do Homebrew varia entre Macs. Instale GnuPG e o pinentry do macOS:

~~~bash
brew install gnupg pinentry-mac
gpg --version
which gpg
which pinentry-mac
~~~

Use os caminhos exibidos por `which`; não presuma `/usr/local` nem `/opt/homebrew`.

## 2. Configure pinentry e gpg-agent

`gpg-agent` gerencia as chaves secretas e chama o pinentry quando precisa da senha. O arquivo é `~/.gnupg/gpg-agent.conf`.

~~~bash
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

Se o arquivo não existir, adicione uma linha usando o caminho real de `which pinentry-mac`:

~~~text
pinentry-program /THE/ACTUAL/PATH/FROM/which-pinentry-mac
~~~

Se já existir, edite apenas essa linha e preserve as demais configurações. Reinicie o agente e configure o terminal:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

No zsh, coloque `export GPG_TTY=$(tty)` em `~/.zshrc` e execute `source ~/.zshrc`. Isso ajuda o terminal a encontrar o prompt de senha.

## 3. Gere a chave

~~~bash
gpg --full-generate-key
~~~

Escolha RSA de 4096 bits com capacidade de assinatura, se disponível, uma expiração que possa renovar, uma senha forte, `YOUR_NAME` e `YOUR_VERIFIED_GITHUB_EMAIL`. O GitHub associa o e-mail do committer a um UID da chave e a um e-mail verificado na conta; o UID não substitui a identidade do Git.

## 4. Encontre a impressão digital completa

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

Na entrada `sec`, copie a fingerprint completa e use `YOUR_GPG_FINGERPRINT`. É necessária uma chave privada para assinar.

## 5. Exporte somente a chave pública

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Copie do marcador `-----BEGIN PGP PUBLIC KEY BLOCK-----` até `-----END PGP PUBLIC KEY BLOCK-----`. Isso é a chave pública; não exporte nem compartilhe material secreto.

## 6. Adicione a chave ao GitHub

No GitHub, abra **Settings → Access → SSH and GPG keys → New GPG key**, informe um título, cole a chave pública e clique em **Add GPG key**. Consulte a [documentação oficial](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) se os rótulos mudarem.

## 7. Configure a identidade global do Git

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` vale para todos os repositórios deste usuário do macOS, exceto quando há uma configuração local.

## 8. Configure a assinatura OpenPGP global

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Confira o formato configurado:

~~~bash
git config --global --get gpg.format
~~~

Se aparecer `ssh` e você quer GPG, remova a substituição para voltar a `openpgp`:

~~~bash
git config --global --unset gpg.format
~~~

## 9. Verifique substituições locais

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
git config --show-origin --get gpg.format
~~~

`file:.git/config` tem prioridade sobre o global. Dentro do repositório, remova uma identidade local indesejada sem `--global`:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Teste a assinatura

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Procure `gpg: Good signature from ...`. Execute `git push` e abra o commit no GitHub: ele deve exibir `Verified`. A verificação local e a associação de conta/e-mail do GitHub são verificações relacionadas, mas diferentes.

## 🛠️ Solução de problemas

- **No pinentry:** se aparecer `gpg: agent_genkey failed: No pinentry` e `Key generation failed: No pinentry`, instale `pinentry-mac`, use `which pinentry-mac`, adicione `pinentry-program` sem apagar o arquivo e reinicie o agente.
- **Permissões do Homebrew:** altere somente a pasta exata informada pelo erro com `sudo chown -R "$(whoami)" "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"` e `chmod u+w "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"`. Nunca faça chown de todo `/usr/local` ou `/opt/homebrew`.
- **Vários GPG ou IDE:** compare `which gpg`, `gpg.program` e o Git/GPG do IDE; todos devem usar o keyring que contém a chave privada.
- **Sem Verified:** confirme a chave pública, `user.email`, o e-mail verificado, `gpg.format` e a saída de `git log --show-signature -1`. Renove chaves expiradas.
- **Chave perdida:** a chave pública não recupera a privada. Restaure um backup ou gere outra; o certificado de revogação apenas anuncia que uma chave deixou de ser confiável.

## 🔐 Backup e futuro

Mantenha uma cópia criptografada e offline da chave privada e do certificado de revogação. Compartilhe apenas a chave pública.

🌿 [GitHub](https://github.com/syllik) · [syllik@gmail.com](mailto:syllik@gmail.com)

<details>
<summary>✨ Uma nota para o futuro</summary>

Você encontrará isto novamente algum dia, quando quiser aprender algo novo.

É gratuito.

Com carinho,<br>
fireflў
</details>

---

[← Início em português](README.md) · [🌍 Todos os idiomas](../../README.md)
