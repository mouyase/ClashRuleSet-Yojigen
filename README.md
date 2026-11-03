# ClashRuleSet-Yojigen

Clash / Mihomo 规则集与订阅转换配置，由 GitHub Pages 托管。

## 地址

| 用途 | URL |
| --- | --- |
| 订阅转换配置 | `https://rules.yojigen.eu.org/config.ini` |
| Clash 基础模板 | `https://rules.yojigen.eu.org/base.yaml` |
| 规则集 | `https://rules.yojigen.eu.org/rules/<name>.list` |

## 用法

```
http://127.0.0.1:25500/sub?target=clash&config=https%3A%2F%2Frules.yojigen.eu.org%2Fconfig.ini
```

## 目录结构

```
.
├── CNAME              # GitHub Pages 自定义域名
├── config.ini         # 订阅转换配置
├── base.yaml          # Clash 基础模板（DNS + TUN）
└── rules/             # 35 个规则集，17921 条规则
```

## 规则集来源

- 32 个来自 [ACL4SSR](https://github.com/ACL4SSR/ACL4SSR)
- 3 个自有：`ModifyChinaList` / `ModifyBlackList` / `E-Hentai`

## base.yaml 设计说明

**与机场无关，可直接用于任何订阅。** 不含任何机场专用 DNS、节点域名或认证 token。

### 解析链路

| 用途 | 字段 | 用什么 |
| --- | --- | --- |
| 引导解析 | `default-nameserver` | 阿里 / 腾讯 / 百度 纯 IP |
| 国内域名 | `nameserver-policy: geosite:cn,private` | 阿里 DoH + dnspod DoH |
| 国外域名 | `nameserver` | Cloudflare + Google DoH/DoT |
| 节点域名 | `proxy-server-nameserver` | 国内 DNS（避免鸡生蛋） |
| 直连重解析 | `direct-nameserver` | 国内 DNS |

关键点：`proxy-server-nameserver` 必须用国内 DNS。否则解析节点域名时要先连国外 DNS，
而连国外 DNS 又要先有节点，形成死锁。

### fake-ip 过滤

`fake-ip-filter` 分三类，共 47 条：

- **geosite 规则**：`private` / `connectivity-check` / `category-ntp`
- **连通性检测**：`msftconnecttest` / `msftncsi`
- **NTP 授时**：Windows / Apple / NTP 池 / OpenWrt
- **音乐与流媒体**：网易云 / QQ音乐 / 酷狗 / 酷我 / Joox / 咪咕
  （这些服务会校验解析 IP，fake-ip 会导致版权区域判断错误）
- **游戏主机**：Switch / PlayStation / Xbox 的 NAT 检测

### 其它要点

- `respect-rules: true` — DNS 连接本身遵守分流规则，且已配 `proxy-server-nameserver`
- `fake-ip-ttl: 1` — 强制客户端频繁重解析，保持映射新鲜
- `cache-algorithm: arc` — 突发访问下命中率优于默认 lru
- `sniffer` — 开启 TLS/QUIC/HTTP 嗅探，纯 IP 连接也能正确分流
- `ipv6: false` — 避免 AAAA 泄漏；需要时改 true 并同步 `fake-ip-range6`
- `find-process-mode: strict` — 注意不要写成 `off`，YAML 会解析成布尔 `false` 导致启动失败

## 更新规则

覆盖 `rules/` 下对应文件即可，URL 结构不变，客户端无需改动。
