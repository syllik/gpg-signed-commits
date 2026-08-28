# 🐧 在 Linux 上配置 GPG 签名提交

本页适用于 Linux 用户的全局 OpenPGP/GPG Git 签名设置。Linux 发行版不同，安装命令也不同。

## 准备工作与安全

请准备 Git、GitHub 账户和一个已添加并验证的 GitHub 邮箱，并在 GPG UID 与 Git 的 `user.email` 中使用它。

> ⚠️ 可以上传公钥；私钥/秘密密钥、密码短语和撤销证书内容必须保密。

## 1. 安装 GnuPG

根据发行版执行：

### Debian / Ubuntu

~~~bash
sudo apt update
sudo apt install gnupg
~~~

### Fedora

~~~bash
sudo dnf install gnupg2
~~~

### Arch Linux

~~~bash
sudo pacman -Syu gnupg
~~~

其他发行版请查看当前官方软件包文档，安装 GnuPG 2 和适合你的终端或桌面的 pinentry。然后检查：

~~~bash
gpg --version
which gpg
~~~

有些旧系统使用 `gpg2`；如果它拥有你的 keyring，后面就把 `gpg.program` 指向它。

## 2. 配置 pinentry、agent 和终端

`gpg-agent` 通常按需启动。pinentry 可以是终端 curses 程序，也可以是 GTK/Qt GUI；不要假定必须使用某个桌面环境。

~~~bash
which pinentry
which pinentry-curses
which pinentry-gtk-2
which pinentry-qt
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

如需指定程序，在 `~/.gnupg/gpg-agent.conf` 中添加真实路径：

~~~text
pinentry-program /THE/ACTUAL/PATH/TO/PINENTRY
~~~

已有配置文件时只编辑这一行并保留其他内容，然后执行：

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

将 `export GPG_TTY=$(tty)` 放入你实际使用的 `~/.bashrc` 或 `~/.zshrc`，再重新打开 shell。它让终端 pinentry 能找到当前 TTY。

## 3. 创建密钥

~~~bash
gpg --full-generate-key
~~~

选择可签名的 RSA 4096 位密钥（如果可选）、可维护的过期时间和强密码短语，输入 `YOUR_NAME` 与 `YOUR_VERIFIED_GITHUB_EMAIL`。GitHub 需要提交者邮箱、GPG UID 和已验证账户邮箱相互匹配。

## 4. 找到指纹

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

复制 `sec` 记录下的完整 fingerprint，作为 `YOUR_GPG_FINGERPRINT`。

## 5. 导出公钥

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

复制完整的 public key block（从 `-----BEGIN PGP PUBLIC KEY BLOCK-----` 到 `-----END PGP PUBLIC KEY BLOCK-----`）。这里只导出公钥，绝不要分享秘密密钥材料。

## 6. 添加到 GitHub

打开 **Settings → Access → SSH and GPG keys → New GPG key**，粘贴公钥并添加。参考 [GitHub 官方说明](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account)。

## 7. 设置全局身份

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` 对本 Linux 用户的所有仓库生效，除非仓库有覆盖。

## 8. 设置全局签名

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

如使用 `gpg2`，改用 `"$(which gpg2)"`。检查旧的签名格式：

~~~bash
git config --global --get gpg.format
~~~

如果是 `ssh`，确认要回到 OpenPGP 后再执行：

~~~bash
git config --global --unset gpg.format
~~~

## 9. 检查本地覆盖

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` 的值会覆盖全局值。用不带 `--global` 的命令删除本地覆盖：

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. 测试并验证

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

应看到 `gpg: Good signature from ...`。然后执行 `git push`，在 GitHub 提交页检查 `Verified`。本地签名检查与 GitHub 的账户验证检查并不完全相同。

## 🛠️ 排查

- **No pinentry / 无法提示密码：** 检查 pinentry 实现、`pinentry-program`、`GPG_TTY` 和 `gpg-agent.conf`，然后 `gpgconf --kill gpg-agent`。
- **agent 问题：** agent 会按需启动；停止它可以让 GnuPG 创建新实例。检查 `~/.gnupg` 权限。
- **多个 GPG 或错误 gpg.program：** 对比 `which gpg`、`which gpg2` 与 `git config --show-origin --get gpg.program`，使用拥有私钥的那一个。
- **终端、GUI、IDE 行为不同：** IDE 可能有不同 PATH、Git 或 GPG；对比实际路径并在同一仓库测试。
- **GitHub 没有 Verified：** 检查公钥、`user.email`、GitHub 已验证邮箱和 `gpg.format`；过期密钥需要续期。
- **私钥丢失或撤销：** 公钥不能恢复私钥；从安全备份恢复或创建新密钥。撤销证书用于发布“不再信任”，不是备份。

## 🔐 备份与未来

加密保存私钥和撤销证书的离线备份，只分享公钥。

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume)

<details>
<summary>✨ 给未来的一句话</summary>

有一天，当你想学点新东西时，你会再次找到这里。

它是免费的。

爱你，<br>
fireflў
</details>

---

[← 中文首页](README.md) · [🌍 全部语言](../../README.md)
