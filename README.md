# Muse 走代理 · Egern 规则集

让 **Muse（Meta 的 AI Agent，muse.ai）** 的流量全部走代理，其它流量不受影响。

## 为什么需要

- `muse.ai`、`api.muse.ai`、`platform.muse.ai`、`auth.muse.ai` 全部 CNAME 到 `star.c10r.facebook.com`（Meta 基础设施），国内直连不通。
- 域名清单不是猜的：取自 `https://muse.ai` 线上响应的 **CSP 头**（`connect-src` / `img-src` / `frame-src`）＋实测 DNS 解析。

## 文件

| 文件 | 用途 |
|---|---|
| `muse.list` | Surge / Shadowrocket / Egern 通用规则集（不带策略，引用时指定） |
| `muse.yaml` | Egern 原生规则集格式 |

## Egern 用法

在配置的 `rules` 里加一条（**放在 `geoip: CN → DIRECT` 之前**，否则会被兜底规则先吃掉）：

```yaml
rules:
  - rule_set:
      match: "https://cdn.jsdelivr.net/gh/bluee1111/egern-muse-proxy@main/muse.list"
      policy: 你的代理组名
      update_interval: 86400
```

或者用 Egern 原生 YAML 版：

```yaml
rules:
  - rule_set:
      match: "https://cdn.jsdelivr.net/gh/bluee1111/egern-muse-proxy@main/muse.yaml"
      policy: 你的代理组名
      update_interval: 86400
```

`policy` 填你配置里已有的策略组或节点名（例如「手动选择」「Proxy」），不要原样抄。

## Surge / Shadowrocket

```
RULE-SET,https://cdn.jsdelivr.net/gh/bluee1111/egern-muse-proxy@main/muse.list,PROXY
```

## 覆盖范围

| 分类 | 域名 |
|---|---|
| 核心 | `muse.ai`、`meta.ai` |
| 内容 CDN | `metaaiusercontent.com`、`ecto1usercontent.com` |
| 登录 / 账号 | `facebook.com`、`fbsbx.com` |
| 媒体 CDN | `fbcdn.net`、`cdninstagram.com`、`instagram.com`、`cdn.whatsapp.net` |
| 内嵌小应用 | `meta-agents-apps.workers.dev` |
| 可选支付 | `api.stripe.com` 等（默认注释） |

## 注意事项

- **没有**把 `meta.com`、`whatsapp.com` 全量放进来，避免顺带代理 Meta 官网和 WhatsApp 本体。
- Muse 的实时聊天用 WebSocket（`wss://gateway.meta.ai`、`wss://edge-chat.meta.ai`、`wss://shortwave.meta.ai`），已包含在 `meta.ai` 后缀内；**节点必须支持 WS**，否则对话会一直转圈。
- App 若走 QUIC/UDP 443 而节点不支持 UDP，让 Egern 的节点保持 TCP，或加一条 `protocol: quic → REJECT` 强制回落。
- 登录 Muse 用 Meta 账号，`facebook.com` 走代理是必要的；如果你的 Facebook App 另有分流规则，注意本条顺序不要冲突。
