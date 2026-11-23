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

Moomoo 额外带三类补充规则：

- `DOMAIN,shortsgp.im.qcloud.com`：评论区走腾讯云 IM 的新加坡接入点，被判定内地时评论区会隐藏。用精确主机名而非后缀，避免把其他使用腾讯云 IM 的 App 一起代理。
- 两条 `IP-CIDR`：行情长连接按 IP 直连（9595 端口、无 SNI），域名规则匹配不到；这两台已核验为 `futunn.com` 自有服务器。
- 其余行情/交易机群 IP 仍在轮换，暂不写入，避免误伤同一机房的其他服务。如果之后确认有新的判定相关 IP，再单独补。

机群 IP 会轮换。如果 App 又提示内地，用 Shadowrocket 连接日志确认是哪条连接走了直连，再按同样方式补。



## 收录与排除

- Moomoo：基础集来自 [proxyBug/tigertrade-moomoo-rules](https://github.com/proxyBug/tigertrade-moomoo-rules) 的 24 条核验域名，另补 `futuhainan.com`、`futuie.com`、`futuhongkong.com`、`moomooapp.com`、`moomoocn.com`（逐一核对过 TLS 证书主体为富途/Moomoo 实体）。
- 华盛通：`hstong.com`、`hstong.com.cn`、`hstbroker.com`、`vcglg.com`（证书主体为深圳市时代华盛网络科技 / 深圳价值网络科技）、`vbkr.com`、`vbkrhk.com`、`valuable.com.hk`（证书主体为 VALUABLE CAPITAL LIMITED）、`vcl*` 系列品牌域。
- 主动排除：`huashengtong.com`（武汉华圣通电力科技，与华盛证券无关）、`valuablecapital.com` / `futu0-9.com`（停放域名）、`moomoo.com.hk`（证书非官方）、`hszq6.com`（第三方短链平台）、AppsFlyer 等共享 SDK 域名。腾讯云 IM 只精确放行 Moomoo 使用的 `shortsgp.im.qcloud.com`，不收后缀，避免误伤。
