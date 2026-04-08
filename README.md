# Lagrange V2 Sign API 使用指南

## 自助申请

> [!important]
>
> 请务必**完整阅读**以下流程后再进行申请，以免因为操作不当导致申请失败。在下面的流程中，你需要使用 GPG 私钥进行解密操作，请自行查阅相关资料以了解如何使用 GPG 工具。
>
> 此外，你所使用的 GPG Key 的算法需要支持加密和解密操作（如 RSA），如果你的 GPG Key 仅支持签名（如 Ed25519），则无法完成申请流程。

Lagrange V2 Sign API 是**白名单制**，可通过特定的 QQ Bot 自助申请使用权限。流程如下：

首先，确保你的 GitHub 账户中添加了 GPG Key。在包含特定 QQ Bot 的群聊中，使用你的主 QQ 号发送如下内容：

```
#signer register <你的 GitHub 用户名> <你的 GPG Key ID>
```

注意这里所需的 Key ID 是主 ID，而非 Subkey 的 ID。它的长度应该是 16 个字符。

> [!tip]
>
> 如果你的请求被成功接收，Bot 会给你的消息贴一个 `/续标识`（手指按红色按钮）的表情。你可以通过有没有这个表情来判断 Bot 是否成功接收了你的消息。
>
> 如果你的请求处理成功，Bot 会给你的消息贴一个 `/喝彩`（派对礼花）的表情。
>
> 如果你的请求有误，Bot 会给你的表情贴一个 `/敲`（大锤敲头）的表情，请检查你的消息格式是否正确。
>
> 对 Bot 的调用有 1 分钟/次的 rate limit。如果你在 1 分钟内重复调用 Bot，Bot 会给你的消息贴一个 `/睡`（小黄脸睡觉）的表情，来表示你触发了 rate limit。遇到这种情况时，请等待至少 1 分钟后再尝试调用 Bot。Rate limit 适用于 `register`、`bind`、`unbind` 和 `profile` 命令。

随后，Bot 会提供一则使用 GPG 公钥加密的消息，你需要**在 5 分钟内**使用对应的 GPG 私钥进行解密，并将解密的字符串内容（通常是 6 位数字或字母组合）按照以下格式发送到群聊中：

```
#signer verify <解密后的字符串内容>
```

如果验证成功，Bot 会回复注册成功的消息，并且将 token 通过相同的 GPG 公钥加密后发送到群聊中。你需要使用 GPG 私钥解密后获取 token，并妥善保管这个 token，因为它是你后续使用签名 API 的唯一凭证。

在 Bot 提示注册成功后，即可开始绑定实际用到签名的 QQ 号。每个 GitHub 账户对应 1 个主 QQ 号，并可以绑定至多 3 个子 QQ 号，即**实际使用签名的 QQ 号**。绑定方式为发送以下内容：

```
#signer bind <要绑定的 QQ 号>
```

主 QQ 号本身不具备签名使用权限，只有绑定的实际使用签名的 QQ 号才具备签名权限。主 QQ 号也可以绑定自身为子 QQ 号，这时主 QQ 号具备签名权限，但也会占用 1 个绑定名额。

如果绑定的子 QQ 号不再需要使用签名权限，可以通过以下命令解绑：

```
#signer unbind <要解绑的 QQ 号>
```

此外，可以通过如下方式查看目前已经绑定的 QQ 号列表：

```
#signer profile
```

## 协议端对接

在成功申请到 token 并绑定了子 QQ 号后，即可在协议端中使用该 token 进行签名。

### 对接 [Lagrange.Milky](https://lagrangedev.github.io/Lagrange.Milky.Document/)

首先，保证你的 Lagrange.Milky 是最新版本。然后，修改以下关键配置项：

```jsonc
{
  "Core": {
    "Signer": {
      "Url": "https://sign.lagrangecore.org", // Lagrange V2 Sign API 的地址
      "Token": "", // 使用 GPG 私钥解密后获取的 token，通常是一个 UUID 字符串
    },
    "Login": {
      "Uin": 0, // 改为你绑定的子 QQ 号
      "Password": null, // 设为 null 以使用二维码登录
    },
  },
}
```

更多相关说明见 [Lagrange.Milky 文档的配置部分](https://lagrangedev.github.io/Lagrange.Milky.Document/configuration/overview)。

### 对接 [Yogurt](https://acidify.ntqqrev.org/yogurt/start)

首先，保证你的 Yogurt 版本在 `v0.1.0-dev.177` 以上，并且 `config.json` 中的 `configVersion` 为 `3`（如果不为 `3`，可以启动一次 Yogurt 后退出，以更新配置文件）。然后，修改以下关键配置项：

```jsonc
{
  "protocol": {
    "uin": 0, // 改为你绑定的子 QQ 号
    "password": "", // 留空以使用二维码登录
    "os": "Linux", // 使用 Linux 协议
    "version": "46494", // 使用 Linux 协议版本 46494
    "signApiUrl": "https://sign.lagrangecore.org/", // Lagrange V2 Sign API 的地址
    "pcLagrangeSignToken": "", // 使用 GPG 私钥解密后获取的 token，通常是一个 UUID 字符串
  },
}
```

更多相关说明见 [Yogurt 文档的配置部分](https://acidify.ntqqrev.org/yogurt/configuration)。
