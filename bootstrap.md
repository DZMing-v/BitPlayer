##  `bootstrap.json` 格式

文件为 JSON 数组，每项字段如下：

| 字段               | 类型   | 说明                                                        |
| ------------------ | ------ | ----------------------------------------------------------- |
| `bootstrap_node`   | string | libp2p 多地址，例如 `/dnsaddr/.../p2p/...`                  |
| `sponsor_url`      | string | HTTP(S) 推广页，用于赞助者 Tab 内嵌                         |
| `sponsor_url_web3` | string | Web3 推广页（入库并进入内存；当前 UI 只加载 `sponsor_url`） |
| `algorithm`        | string | 固定 `ed25519`（可空，空则按 ed25519 验签）                 |
| `pub`              | string | 验签公钥，**优先 hex**；也兼容 standard/raw base64          |
| `node_url_sign`    | string | 签名，编码规则同 `pub`                                      |

示例：

```json
[
  {
    "bootstrap_node": "/dnsaddr/example/p2p/Qm...",
    "sponsor_url": "https://sponsor.example/",
    "sponsor_url_web3": "",
    "algorithm": "ed25519",
    "pub": "<32-byte-pubkey-hex>",
    "node_url_sign": "<64-byte-signature-hex>"
  }
]
```

###  签名规则

只对以下三项签名（按此顺序用换行拼接）：

```text
bootstrap_node + "\n" + sponsor_url + "\n" + sponsor_url_web3
```

- 算法：Ed25519
- 验签：用条目内的 `pub` 校验 `node_url_sign`
- 验签失败：`sign_valid = 0`，**仍会插入数据库**，初始分 90
- 验签成功：`sign_valid = 1`

Go 侧等价代码：

```go
msg := []byte(bootstrapNode + "\n" + sponsorURL + "\n" + sponsorURLWeb3)
sig := ed25519.Sign(privateKey, msg)
```

### 