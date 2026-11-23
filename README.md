# proxy-rules

券商 App 的代理分流规则，Shadowrocket 格式。只收券商第一方域名，不含第三方 SDK 与共享 CDN，避免把其他 App 的流量一起送进代理。

## 列表

| 文件 | 对应 App | 条数 |
| --- | --- | --- |
| [`Moomoo.list`](Moomoo.list) | 富途牛牛 / Moomoo | 29 |
| [`ValuableCapital.list`](ValuableCapital.list) | 华盛通 / 华盛证券 / VBrokers | 10 |

## 使用

Shadowrocket → 配置 → 编辑当前配置 → `[Rule]` 段添加（策略组名按自己的改）：

```ini
RULE-SET,https://raw.githubusercontent.com/sleepingzhao/proxy-rules/main/Moomoo.list,Proxy
RULE-SET,https://raw.githubusercontent.com/sleepingzhao/proxy-rules/main/ValuableCapital.list,Proxy
```

raw.githubusercontent.com 不通时换 jsDelivr：

```ini
RULE-SET,https://cdn.jsdelivr.net/gh/sleepingzhao/proxy-rules@main/Moomoo.list,Proxy
RULE-SET,https://cdn.jsdelivr.net/gh/sleepingzhao/proxy-rules@main/ValuableCapital.list,Proxy
```

规则用 `DOMAIN-SUFFIX` 显式匹配，不使用 `DOMAIN-KEYWORD`，避免误伤。添加后首次连接可能需要杀掉 App 重开以刷新 DNS。

**规则顺序很重要**：`RULE-SET` 必须放在 `GEOIP,CN`、`GEOIP,CN,DIRECT` 一类按国家/地区直连的规则之前。富途大量域名解析到腾讯云内地节点（如 `119.91.245.x`），如果 `GEOIP,CN` 在前，会先命中直连，域名规则根本轮不到生效。

Moomoo 额外带一批 `IP-CIDR`：行情/交易客户端会按 IP 直连服务器机群（9595 等非标准端口，没有 SNI），域名规则完全匹配不到。这些网段来自 Moomoo 实际连接 IP，并与社区券商规则集交叉核验，全部是富途的 Tencent Cloud 机群。券商是**在服务端按连接的来源 IP** 判断地区的，所以只要还有一条行情连接直连内地机群，就可能被判为内地——这就是为什么要把这些段一起走了代理。

机群 IP 会轮换。如果以后又有新 IP 漏出去，把 Moomoo 的「网络活动」里靠前的 IP 发出来，或用 Shadowrocket 连接日志确认，按同样方式补 `IP-CIDR` 即可。


## 收录与排除

- Moomoo：基础集来自 [proxyBug/tigertrade-moomoo-rules](https://github.com/proxyBug/tigertrade-moomoo-rules) 的 24 条核验域名，另补 `futuhainan.com`、`futuie.com`、`futuhongkong.com`、`moomooapp.com`、`moomoocn.com`（逐一核对过 TLS 证书主体为富途/Moomoo 实体）。
- 华盛通：`hstong.com`、`hstong.com.cn`、`hstbroker.com`、`vcglg.com`（证书主体为深圳市时代华盛网络科技 / 深圳价值网络科技）、`vbkr.com`、`vbkrhk.com`、`valuable.com.hk`（证书主体为 VALUABLE CAPITAL LIMITED）、`vcl*` 系列品牌域。
- 主动排除：`huashengtong.com`（武汉华圣通电力科技，与华盛证券无关）、`valuablecapital.com` / `futu0-9.com`（停放域名）、`moomoo.com.hk`（证书非官方）、`hszq6.com`（第三方短链平台）、腾讯云 IM / AppsFlyer 等第三方 SDK 域名（多个 App 共用，会误伤）。
