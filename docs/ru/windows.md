# 🪟 Подписанные коммиты Git с GPG в Windows

Этот вариант использует Gpg4win и Kleopatra. Git for Windows предоставляет Git Bash, а PowerShell и Command Prompt используют собственный синтаксис.

## Подготовка и безопасность

Установите [Gpg4win](https://www.gpg4win.org/download.html) и [Git for Windows](https://git-scm.com/download/win). Подготовьте подтверждённый email GitHub.

> ⚠️ Загружайте только открытый ключ. Закрытый/секретный ключ, парольную фразу и сертификат отзыва нельзя передавать.

## 1. Установите и найдите GPG

В PowerShell, Command Prompt или Git Bash:

~~~text
gpg --version
~~~

PowerShell:

~~~powershell
Get-Command gpg -All
Get-Command git -All
~~~

Command Prompt или Git Bash:

~~~text
where gpg
where git
~~~

Используйте `gpg.exe` из той же установки Gpg4win, чей keyring использует Kleopatra. Не применяйте Unix-путь `/usr/local/bin/gpg` и не копируйте macOS-настройку `GPG_TTY`.

## 2. Создайте ключ

В Kleopatra выберите **File → New Certificate → OpenPGP** или выполните:

~~~text
gpg --full-generate-key
~~~

Выберите RSA 4096 с подписью, если доступно, поддерживаемый срок, сильную парольную фразу, `YOUR_NAME` и `YOUR_VERIFIED_GITHUB_EMAIL`.

## 3. Найдите fingerprint

~~~text
gpg --list-secret-keys --keyid-format=long
~~~

Скопируйте полный fingerprint под `sec` как `YOUR_GPG_FINGERPRINT`.

## 4. Экспортируйте открытый ключ

~~~text
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Скопируйте блок между `-----BEGIN PGP PUBLIC KEY BLOCK-----` и `-----END PGP PUBLIC KEY BLOCK-----`. Это открытый ключ; секретные материалы не передавайте.

## 5. Добавьте ключ в GitHub

Откройте **Settings → Access → SSH and GPG keys → New GPG key**, вставьте открытый ключ и нажмите **Add GPG key**. См. [официальную страницу](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 6. Настройте глобальную идентичность

~~~text
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` действует для всех репозиториев этого пользователя Windows, кроме локальных переопределений.

## 7. Укажите Git правильный GPG

В PowerShell или Command Prompt замените пример путем из `Get-Command gpg` или `where gpg`:

~~~powershell
git config --global gpg.program "C:/Program Files/GnuPG/bin/gpg.exe"
~~~

Git for Windows принимает такой путь в кавычках. Затем:

~~~text
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
git config --global --get gpg.format
~~~

Если вывод — `ssh`, а нужен GPG:

~~~text
git config --global --unset gpg.format
~~~

## 8. Проверьте локальные переопределения

~~~text
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` важнее глобальной настройки. В репозитории:

~~~text
git config --unset user.name
git config --unset user.email
~~~

## 9. Проверьте подпись

~~~text
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Kleopatra или pinentry запросит пароль. Ищите `gpg: Good signature from ...`, выполните `git push` и проверьте `Verified` на GitHub.

## 🛠️ Устранение проблем

- **Несколько gpg.exe:** сравните `Get-Command gpg -All` / `where gpg` с `gpg.program` и используйте одну установку Gpg4win вместе с Kleopatra.
- **Не появляется окно pinentry:** запустите Kleopatra и выполните `gpgconf --kill gpg-agent`. В Windows `GPG_TTY` не требуется.
- **IDE:** настройте в IDE тот же Git и GPG, которые работают в терминале.
- **Нет Verified:** проверьте открытый ключ, `user.email`, подтверждённый email, `gpg.format` и `git log --show-signature -1`.
- **Ключ истёк, потерян или отозван:** продлите или создайте новый. Публичный ключ не восстанавливает закрытый; отзыв сообщает только о потере доверия.

## 🔐 Резервная копия и заметка на будущее

В Kleopatra или GnuPG храните зашифрованную офлайн-копию закрытого ключа и сертификата отзыва. Делитесь только открытым ключом.

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume)

<details>
<summary>✨ Заметка на будущее</summary>

Когда-нибудь вы снова найдёте это, если захотите узнать что-то новое.

Это бесплатно.

С любовью,<br>
fireflў
</details>

---

[← Русская главная](README.md) · [🌍 Все языки](../../README.md)
