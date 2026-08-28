# 🍎 在 macOS 上配置 GPG 签名提交

本页将 OpenPGP/GPG 签名配置为对当前 macOS 用户的所有本地 Git 仓库生效。

## 你将配置什么

你会使用 Homebrew 安装 GnuPG 和 `pinentry-mac`，创建签名密钥，只上传公钥到 GitHub，然后设置全局 Git 身份和签名选项。请先确认 GitHub 账户已有一个已添加并验证的邮箱，并在密钥和 Git 中使用同一个邮箱。

> ⚠️ **公钥与私钥：** `gpg --armor --export` 导出的是公钥，可以上传到 GitHub。私钥/秘密密钥、密码短语和撤销证书内容绝不能上传或分享。

## 1. 安装并检查 GnuPG

Homebrew 在 Intel 和 Apple Silicon Mac 上的安装前缀可能不同，因此不要猜路径：

~~~bash
brew install gnupg pinentry-mac
gpg --version
which gpg
which pinentry-mac
~~~

`gpg --version` 应显示 GnuPG 2.x。`which` 输出的路径就是你实际使用的程序路径。

## 2. 安全配置 pinentry 和 agent

`gpg-agent` 管理秘密密钥，并调用 pinentry 请求密码。配置文件是 `~/.gnupg/gpg-agent.conf`。

~~~bash
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
~~~

如果配置文件不存在，创建它并写入使用 `which pinentry-mac` 找到的真实路径：

~~~text
pinentry-program /THE/ACTUAL/PATH/FROM/which-pinentry-mac
~~~

如果文件已经存在，请编辑并只添加或修改 `pinentry-program` 行，不要覆盖其他配置。常见前缀是 Apple Silicon 的 `/opt/homebrew` 和 Intel 的 `/usr/local`，但应以实际发现的路径为准。

重启 agent，并让终端知道当前 TTY：

~~~bash
gpgconf --kill gpg-agent
export GPG_TTY=$(tty)
~~~

如需在 zsh 中永久生效，把下面一行加入 `~/.zshrc`，再执行 `source ~/.zshrc`：

~~~bash
export GPG_TTY=$(tty)
~~~

`GPG_TTY` 帮助 GPG 在终端中找到密码提示；GUI pinentry 仍可能弹出独立窗口。

## 3. 创建密钥

~~~bash
gpg --full-generate-key
~~~

按向导选择可用于签名的 RSA 4096 位密钥（如果界面提供）、设置你能维护的过期时间和强密码短语，并输入 `YOUR_NAME` 与 `YOUR_VERIFIED_GITHUB_EMAIL`。GitHub 会将提交者邮箱与密钥 UID 和账户中已验证邮箱关联；UID 本身不会替代 Git 的提交者身份。

## 4. 找到完整指纹

~~~bash
gpg --list-secret-keys --keyid-format=long
~~~

在 `sec` 记录下面找到完整的 fingerprint，保存为 `YOUR_GPG_FINGERPRINT`。签名需要私钥，只有公钥不能创建签名。

## 5. 只导出公钥

~~~bash
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

复制从 `-----BEGIN PGP PUBLIC KEY BLOCK-----` 到 `-----END PGP PUBLIC KEY BLOCK-----` 的全部内容。它是公钥。不要导出或分享任何秘密密钥材料。

## 6. 添加到 GitHub

在 GitHub 中打开 **Settings → Access → SSH and GPG keys → New GPG key**，填写标题，粘贴公钥并点击 **Add GPG key**。如界面变化，请参考 [GitHub 官方说明](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account)。

## 7. 设置全局 Git 身份

~~~bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` 对当前 macOS 用户的所有仓库生效，除非仓库有本地覆盖。GPG UID 邮箱和 Git 的 `user.email` 都要正确。

## 8. 设置全局 OpenPGP 签名

~~~bash
git config --global gpg.program "$(which gpg)"
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
~~~

检查是否曾设置 SSH 签名格式：

~~~bash
git config --global --get gpg.format
~~~

若输出 `ssh` 且你要使用 GPG，可删除该全局覆盖，使默认格式回到 `openpgp`：

~~~bash
git config --global --unset gpg.format
~~~

## 9. 检查仓库本地覆盖

仓库的 `.git/config` 会覆盖全局配置：

~~~bash
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

看到 `file:.git/config` 就表示该值来自本地配置。进入该仓库后，用下面命令移除不需要的本地身份；不要加 `--global`：

~~~bash
git config --unset user.name
git config --unset user.email
~~~

## 10. 测试签名提交

在测试仓库中：

~~~bash
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

成功时应看到类似 `gpg: Good signature from ...` 的信息。然后 `git push`，在 GitHub 的提交页面查看 `Verified`。本地密码学验证和 GitHub 的密钥/邮箱账户关联是两个相关但不同的检查。

## 🛠️ 常见问题

- **No pinentry 或无法输入密码：** 检查 `which gpg`、`which pinentry-mac`、`~/.gnupg/gpg-agent.conf` 和 `GPG_TTY`，再运行 `gpgconf --kill gpg-agent`。不要为了添加一行而覆盖已有配置。
- **macOS 生成密钥时报错：** 若出现 `gpg: agent_genkey failed: No pinentry` 和 `Key generation failed: No pinentry`，安装 `pinentry-mac`，用 `which pinentry-mac` 找路径，配置 `pinentry-program`，重启 agent 后重试。
- **Homebrew 无写权限：** 只处理错误中明确指出的目录：`sudo chown -R "$(whoami)" "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"` 和 `chmod u+w "/THE/EXACT/DIRECTORY/FROM/THE/ERROR"`。不要对整个 `/usr/local` 或 `/opt/homebrew` 执行递归 chown。
- **多个 GPG、终端/IDE 不一致：** 对比 `which gpg`、Git 的 `gpg.program` 和 IDE 使用的 PATH；它们必须使用包含该私钥的同一 keyring。
- **GitHub 没有 Verified：** 确认上传的是对应公钥，`user.email` 同时是密钥 UID 和 GitHub 已验证邮箱，并确认 `gpg.format` 不是 `ssh`。
- **密钥过期、多个密钥或丢失私钥：** 续期并更新 GitHub，或用完整指纹设置 `user.signingkey`。GitHub 上的公钥不能恢复丢失的私钥；从安全备份恢复或创建新密钥。撤销证书用于声明密钥不再可信，不是恢复文件。

## 🔐 备份与未来

离线、加密备份私钥和撤销证书，不要放进此仓库。公钥可以分享，秘密密钥不可以。

🌿 [YouTube — @plainsight37](https://youtube.com/@plainsight37) · [Instagram — @fly_lume](https://instagram.com/fly_lume) · [Telegram — @syllik](https://t.me/syllik)

<details>
<summary>✨ 给未来的一句话</summary>

有一天，当你想学点新东西时，你会再次找到这里。

它是免费的。

爱你，<br>
fireflў
</details>

---

[← 中文首页](README.md) · [🌍 全部语言](../../README.md)
