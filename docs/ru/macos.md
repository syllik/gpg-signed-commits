# 🍎 Подписанные коммиты Git с GPG в macOS

Настройте подпись OpenPGP/GPG для всех локальных Git-репозиториев текущего пользователя macOS.

## Что понадобится

Нужны Git, Homebrew, аккаунт GitHub и добавленный и подтверждённый в GitHub адрес электронной почты. Используйте этот адрес в UID GPG и в `user.email` Git.

> ⚠️ **Открытый ключ** можно экспортировать и добавить в GitHub. Закрытый/секретный ключ, парольная фраза и содержимое сертификата отзыва должны оставаться в секрете.

## 1. Установите и проверьте GnuPG

Префикс Homebrew зависит от Mac. Установите GnuPG и pinentry для macOS, затем найдите реальные пути:

~~~bash
brew install gnupg pinentry-mac
gpg --version
which gpg
which pinentry-mac
~~~

Не угадывайте `/usr/local` или `/opt/homebrew` — используйте вывод `which`.

## 2. Настройте pinentry и gpg-agent

`gpg-agent` управляет секретными ключами и вызывает pinentry для ввода парольной фразы. Файл настроек — `~/.gnupg/gpg-agent.conf`.

~~~bash
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

Если файла нет, добавьте в него путь из `which pinentry-mac`:

~~~text
pinentry-program /THE/ACTUAL/PATH/FROM/which-pinentry-mac
~~~

Если файл существует, отредактируйте только эту строку и сохраните остальные настройки. Перезапустите агент и укажите текущий терминал:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

Для zsh добавьте `export GPG_TTY=$(tty)` в `~/.zshrc` и выполните `source ~/.zshrc`. `GPG_TTY` помогает терминалу найти окно или запрос pinentry.

## 3. Создайте ключ

~~~bash
gpg --full-generate-key
~~~

Если такой вариант предложен, выберите RSA 4096 с возможностью подписи, срок действия, который сможете продлевать, сильную парольную фразу, `YOUR_NAME` и `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub связывает email коммитера с UID ключа и подтверждённым email аккаунта; один UID не заменяет Git-идентичность.

## 4. Найдите полный fingerprint

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

Под записью `sec` скопируйте полный fingerprint и используйте его как `YOUR_GPG_FINGERPRINT`. Для подписи нужен закрытый ключ.

## 5. Экспортируйте только открытый ключ

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Скопируйте весь блок от `-----BEGIN PGP PUBLIC KEY BLOCK-----` до `-----END PGP PUBLIC KEY BLOCK-----`. Это открытый ключ. Не экспортируйте и не передавайте секретный материал.

## 6. Добавьте ключ в GitHub

Откройте **Settings → Access → SSH and GPG keys → New GPG key**, укажите название, вставьте открытый ключ и нажмите **Add GPG key**. При изменении интерфейса сверяйтесь с [официальной инструкцией GitHub](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 7. Настройте глобальную Git-идентичность

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` применяется ко всем репозиториям текущего пользователя macOS, если репозиторий не задаёт собственное значение.

## 8. Включите глобальную подпись OpenPGP

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Проверьте, не задан ли другой формат:

~~~bash
git config --global --get gpg.format
~~~

Если вывод — `ssh`, а вы хотите GPG, удалите переопределение, чтобы вернуться к формату `openpgp`:

~~~bash
git config --global --unset gpg.format
~~~

## 9. Проверьте локальные переопределения

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
git config --show-origin --get gpg.format
~~~

Значение из `file:.git/config` важнее глобального. Внутри репозитория удалите ненужную локальную идентичность без `--global`:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Проверьте подписанный коммит

В тестовом репозитории:

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Ищите сообщение `gpg: Good signature from ...`. Выполните `git push` и откройте коммит в GitHub: должен появиться `Verified`. Локальная криптографическая проверка и связывание ключа с аккаунтом GitHub — не одно и то же.

## 🛠️ Устранение проблем

- **No pinentry:** при `gpg: agent_genkey failed: No pinentry` и `Key generation failed: No pinentry` установите `pinentry-mac`, найдите путь через `which pinentry-mac`, добавьте `pinentry-program` в существующий файл, перезапустите агент и повторите попытку.
- **Homebrew не может записать в каталог:** изменяйте только точный каталог из ошибки командами `sudo chown -R "$(whoami)" "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"` и `chmod u+w "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"`. Не выполняйте рекурсивный chown для всего `/usr/local` или `/opt/homebrew`.
- **Несколько GPG или проблема IDE:** сравните `which gpg`, `gpg.program`, PATH, Git и GPG в IDE. Они должны видеть keyring с закрытым ключом.
- **Нет Verified на GitHub:** проверьте открытый ключ, `user.email`, подтверждённый email, `gpg.format` и вывод `git log --show-signature -1`. Просроченный ключ нужно продлить.
- **Ключ потерян или отозван:** открытый ключ не восстанавливает потерянный закрытый. Используйте защищённую копию или создайте новый ключ; сертификат отзыва только сообщает, что ключ больше нельзя считать доверенным.

## 🔐 Резервная копия и заметка на будущее

Храните зашифрованную офлайн-копию закрытого ключа и сертификата отзыва. Передавайте только открытый ключ.

🌿 [GitHub](https://github.com/syllik) · [syllik@gmail.com](mailto:syllik@gmail.com)

<details>
<summary>✨ Заметка на будущее</summary>

Когда-нибудь вы снова найдёте это, если захотите узнать что-то новое.

Это бесплатно.

С любовью,<br>
fireflў
</details>

---

[← Русская главная](README.md) · [🌍 Все языки](../../README.md)
