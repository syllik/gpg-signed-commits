# 🐧 Подписанные коммиты Git с GPG в Linux

Настройте глобальную подпись OpenPGP/GPG для пользователя Linux. Команда установки зависит от дистрибутива.

## Подготовка и безопасность

Нужны Git, аккаунт GitHub и подтверждённый email. Используйте его в GPG и в `user.email`. Открытый ключ можно публиковать; закрытый ключ, парольную фразу и сертификат отзыва — нельзя.

## 1. Установите GnuPG

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

Для другого дистрибутива смотрите его актуальную документацию и установите GnuPG 2 с подходящим pinentry. Проверьте:

~~~bash
gpg --version
which gpg
~~~

Если ключи находятся в `gpg2`, используйте его последовательно и укажите его в `gpg.program`.

## 2. Настройте pinentry и агент

`gpg-agent` обычно запускается по требованию. Выберите терминальный или графический pinentry под свою среду; не предполагайте GNOME, KDE, Wayland, X11 или systemd.

~~~bash
which pinentry
which pinentry-curses
which pinentry-gtk-2
which pinentry-qt
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

При необходимости добавьте реальный путь в `~/.gnupg/gpg-agent.conf`:

~~~text
pinentry-program /THE/ACTUAL/PATH/TO/PINENTRY
~~~

Сохраните существующие строки и выполните:

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

Добавьте `export GPG_TTY=$(tty)` в используемый вами `~/.bashrc` или `~/.zshrc` и откройте новый shell.

## 3. Создайте ключ

~~~bash
gpg --full-generate-key
~~~

Выберите RSA 4096 с подписью, если доступно, поддерживаемый срок действия, сильную парольную фразу, `YOUR_NAME` и `YOUR_VERIFIED_GITHUB_EMAIL`. GitHub должен сопоставить email коммитера, UID GPG и подтверждённый email.

## 4. Найдите fingerprint

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

Скопируйте полный fingerprint под `sec` как `YOUR_GPG_FINGERPRINT`.

## 5. Экспортируйте открытый ключ

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

Скопируйте публичный блок от `-----BEGIN PGP PUBLIC KEY BLOCK-----` до `-----END PGP PUBLIC KEY BLOCK-----`. Секретный материал не передавайте.

## 6. Добавьте его в GitHub

Откройте **Settings → Access → SSH and GPG keys → New GPG key**, вставьте открытый ключ и добавьте его. См. [официальную документацию GitHub](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

## 7. Задайте глобальную идентичность

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` действует для репозиториев пользователя, если нет локального переопределения.

## 8. Включите глобальную подпись

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

Проверьте формат:

~~~bash
git config --global --get gpg.format
~~~

Если это `ssh`, удалите значение только при намерении вернуться к `openpgp`:

~~~bash
git config --global --unset gpg.format
~~~

## 9. Проверьте локальные настройки

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` имеет приоритет. Удалите локальную идентичность:

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. Тест

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Ищите `gpg: Good signature from ...`, затем выполните `git push` и проверьте `Verified` на GitHub. Локальная проверка и привязка к аккаунту — разные проверки.

## 🛠️ Устранение проблем

- **Нет pinentry или запроса пароля:** проверьте программу, `pinentry-program`, `GPG_TTY` и права `~/.gnupg`; затем выполните `gpgconf --kill gpg-agent`.
- **Несколько GPG:** сравните `which gpg`, `which gpg2` и `git config --show-origin --get gpg.program`.
- **Terminal/GUI/IDE:** сравните PATH, Git и GPG, которыми пользуется IDE.
- **Нет Verified:** проверьте публичный ключ, `user.email`, подтверждённый email, `gpg.format` и срок действия ключа.
- **Закрытый ключ потерян:** публичный ключ не может его восстановить. Используйте резервную копию или создайте новый ключ; сертификат отзыва только объявляет потерю доверия.

## 🔐 Резервная копия и заметка на будущее

Храните закрытый ключ и сертификат отзыва зашифрованными и офлайн. Передавайте только открытый ключ.

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume) · [Telegram — @syllik](https://t.me/syllik)

<details>
<summary>✨ Заметка на будущее</summary>

Когда-нибудь вы снова найдёте это, если захотите узнать что-то новое.

Это бесплатно.

С любовью,<br>
fireflў
</details>

---

[← Русская главная](README.md) · [🌍 Все языки](../../README.md)
