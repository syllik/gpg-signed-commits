# 🪟 在 Windows 上配置 GPG 签名提交

这是 Windows 原生流程：Gpg4win 提供 GnuPG、Kleopatra、gpg-agent 和 pinentry；Git for Windows 提供 Git Bash；PowerShell 与 Command Prompt 是不同的 shell。

## 准备工作与安全

从 [Gpg4win 官网](https://www.gpg4win.org/download.html) 安装 Gpg4win，并安装 [Git for Windows](https://git-scm.com/download/win)。准备一个已验证的 GitHub 邮箱。

> ⚠️ 只上传公钥。私钥/秘密密钥、密码短语和撤销证书内容永远不要上传或分享。

## 1. 检查实际程序

在 PowerShell、Command Prompt 或 Git Bash 中执行：

~~~text
gpg --version
~~~

PowerShell：

~~~powershell
Get-Command gpg -All
Get-Command git -All
~~~

Command Prompt 或 Git Bash：

~~~text
where gpg
where git
~~~

可能会看到 `C:\Program Files\GnuPG\bin\gpg.exe` 或 Gpg4win 目录。使用与 Kleopatra keyring 相同的安装。Windows 不要使用 `/usr/local/bin/gpg` 或照搬 Unix 的 `GPG_TTY` 配置。

## 2. 创建密钥

可以在 Kleopatra 中选择 **File → New Certificate → OpenPGP**，也可以在任一 shell 执行：

~~~text
gpg --full-generate-key
~~~

选择可签名的 RSA 4096 位密钥（如果可选）、可维护的过期时间和强密码短语，并输入 `YOUR_NAME` 与 `YOUR_VERIFIED_GITHUB_EMAIL`。GitHub 会匹配提交者邮箱、密钥 UID 和已验证账户邮箱。

## 3. 找到指纹

~~~text
gpg --list-secret-keys --keyid-format=long
~~~

从 `sec` 记录下的 fingerprint 复制完整值，作为 `YOUR_GPG_FINGERPRINT`。签名必须有私钥。

## 4. 导出公钥

PowerShell、Command Prompt 和 Git Bash 都可以执行：

~~~text
gpg --armor --export YOUR_GPG_FINGERPRINT
~~~

复制从 `-----BEGIN PGP PUBLIC KEY BLOCK-----` 到 `-----END PGP PUBLIC KEY BLOCK-----` 的内容。这是公钥，能安全添加到 GitHub；不要导出或分享秘密密钥材料。

## 5. 添加到 GitHub

进入 **Settings → Access → SSH and GPG keys → New GPG key**，填写标题、粘贴公钥并添加。参考 [GitHub 官方页面](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account)。

## 6. 设置全局身份

以下命令在三种 shell 中都可用：

~~~text
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_VERIFIED_GITHUB_EMAIL"
~~~

`--global` 作用于当前 Windows 用户的所有仓库，除非有本地覆盖。

## 7. 指定 Git 使用的 GPG

如果 shell 中的 GPG 正确而 Git 使用了另一个安装，PowerShell 或 Command Prompt 中可以这样设置：

~~~powershell
git config --global gpg.program "C:/Program Files/GnuPG/bin/gpg.exe"
~~~

把示例路径替换为 `Get-Command gpg` 或 `where gpg` 找到的实际路径。Git for Windows 接受带引号的正斜杠 Windows 路径。

~~~text
git config --global user.signingkey YOUR_GPG_FINGERPRINT
git config --global commit.gpgsign true
git config --global tag.gpgSign true
git config --global --get gpg.format
~~~

如果旧格式是 `ssh`，并且你要使用 OpenPGP，再执行：

~~~text
git config --global --unset gpg.format
~~~

## 8. 检查本地覆盖

~~~text
git config --show-origin --get user.name
git config --show-origin --get user.email
git config --show-origin --get user.signingkey
git config --show-origin --get commit.gpgsign
git config --show-origin --get gpg.program
~~~

`file:.git/config` 会覆盖全局设置。在仓库中用以下命令移除本地身份，不要加 `--global`：

~~~text
git config --unset user.name
git config --unset user.email
~~~

## 9. 测试签名

在测试仓库的 PowerShell、Command Prompt 或 Git Bash 中执行：

~~~text
git commit --allow-empty -m "test: signed commit"
git log --show-signature -1
git verify-commit HEAD
~~~

Kleopatra 或 pinentry 应请求密码，并出现 `gpg: Good signature from ...`。执行 `git push` 后，在 GitHub 提交页面看 `Verified`。本地成功还不代表 GitHub 的公钥和邮箱关联一定正确。

## 🛠️ 排查

- **多个 gpg.exe：** 对比 `Get-Command gpg -All` / `where gpg` 与 `git config --show-origin --get gpg.program`，确保 Git 和 Kleopatra 来自同一 Gpg4win 安装。
- **pinentry 窗口不出现：** 启动 Kleopatra，确认当前用户已安装 Gpg4win，然后 `gpgconf --kill gpg-agent`。Windows 不需要 `GPG_TTY`。
- **IDE 失败而命令行成功：** IDE 可能使用另一个 Git、PATH 或 gpg.program；设置为同一套可执行文件。反过来同理。
- **GitHub 没有 Verified：** 检查上传的公钥、`user.email` 与已验证邮箱、`gpg.format`，并用 `git log --show-signature -1` 查看本地签名。
- **密钥过期或有多个密钥：** 续期或用完整指纹设置 `user.signingkey`。公钥不能恢复丢失的私钥；撤销证书只用于宣布密钥失效。

## 🔐 备份与未来

用 Kleopatra 或 GnuPG 加密保存私钥和撤销证书的离线备份，只分享公钥。

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
