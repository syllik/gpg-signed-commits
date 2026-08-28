# 🪟 Commits Git assinados com GPG no Windows

Este fluxo usa Gpg4win e Kleopatra. Git for Windows fornece Git Bash; PowerShell e Command Prompt são shells diferentes.

## Antes de começar

Instale [Gpg4win](https://www.gpg4win.org/download.html) e [Git for Windows](https://git-scm.com/download/win). Tenha um e-mail verificado no GitHub.

> ⚠️ Envie somente a chave pública. Nunca compartilhe a chave privada/secreta, a senha ou o certificado de revogação.

## 1. Instale e localize o GPG

Em PowerShell, Command Prompt ou Git Bash:

~~~text
gpg --version
~~~

PowerShell:

~~~powershell
Get-Command gpg -All
Get-Command git -All
~~~

Command Prompt ou Git Bash:

~~~text
where gpg
where git
~~~

Use o `gpg.exe` da instalação Gpg4win cujo keyring é usado pelo Kleopatra. Não use `/usr/local/bin/gpg` nem copie `GPG_TTY` do macOS.

## 2. Gere a chave

No Kleopatra: **File → New Certificate → OpenPGP**. Ou em qualquer shell:

~~~text
gpg --full-generate-key
~~~

Escolha RSA 4096 com assinatura, se disponível, uma expiração renovável, uma senha forte, `YOUR_NAME` e `YOUR_VERIFIED_GITHUB_EMAIL`. O GitHub relaciona o e-mail do committer, o UID e o e-mail verificado.

## 3. Encontre a fingerprint

~~~text
gpg --list-secret-keys --keyid-format=long
~~~

Copie a fingerprint completa sob `sec` como `YOUR_GPG_FINGERPRINT`.

## 4. Exporte a chave pública

~~~text
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Copie entre `-----BEGIN PGP PUBLIC KEY BLOCK-----` e `-----END PGP PUBLIC KEY BLOCK-----`. É pública; não compartilhe material secreto.

## 5. Adicione ao GitHub

Use **Settings → Access → SSH and GPG keys → New GPG key**, cole a chave pública e clique em **Add GPG key**. Consulte a [página oficial](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 6. Identidade global

~~~text
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` vale para todos os repositórios deste usuário do Windows, salvo substituições locais.

## 7. Aponte o Git para o GPG correto

No PowerShell ou Command Prompt, substitua o exemplo pelo caminho de `Get-Command gpg` ou `where gpg`:

~~~powershell
git config --global gpg.program "C:/Program Files/GnuPG/bin/gpg.exe"
~~~

O Git for Windows aceita esse caminho entre aspas. Depois:

~~~text
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
git config --global --get gpg.format
~~~

Se mostrar `ssh` e você quiser GPG:

~~~text
git config --global --unset gpg.format
~~~

## 8. Verifique substituições

~~~text
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` prevalece. No repositório:

~~~text
git config --unset user.name
git config --unset user.email
~~~

## 9. Teste e verifique

~~~text
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Kleopatra ou pinentry pedirá a senha. Procure `gpg: Good signature from ...`, execute `git push` e confira `Verified` no GitHub.

## 🛠️ Solução de problemas

- **Vários gpg.exe:** compare `Get-Command gpg -All` / `where gpg` com `gpg.program` e use a mesma instalação do Kleopatra.
- **Janela pinentry ausente:** abra o Kleopatra e execute `gpgconf --kill gpg-agent`. Windows não precisa de `GPG_TTY`.
- **IDE:** configure o mesmo Git e GPG que funcionam no terminal.
- **Sem Verified:** confira chave pública, `user.email`, e-mail verificado, `gpg.format` e `git log --show-signature -1`.
- **Chave expirada/perdida/revogada:** renove ou crie outra. A pública não recupera a privada; a revogação apenas comunica que não é mais confiável.

## 🔐 Backup e futuro

Mantenha com Kleopatra ou GnuPG uma cópia criptografada e offline da chave privada e do certificado de revogação. Compartilhe somente a chave pública.

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
