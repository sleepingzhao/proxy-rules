# proxy-rules

券商 App 的代理分流规则，Shadowrocket 格式。尽量只收券商自己的域名，避免把其他 App 的流量一起送进代理。

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
