---
name: domains
description: 使用实用的 DNS 和安全指导注册、管理和保护 domain names。
---

# Domain Management Rules

## 注册前
- 直接在 registrar 上检查可用性 —— 来自第三方的 WHOIS 查询可能触发 front-running（有人在你之前注册）
- 搜索 domain name + "scam" 或 "controversy" —— 前所有者会留下 reputation baggage
- 在投入 branding 前验证商标冲突 —— 法律纠纷代价高昂

## 选择 Extensions
- .com 对普通受众仍有最高信任度 —— alternatives 可用但需要更多品牌建设
- Country TLDs（.co.uk、.de）在本地搜索中排名更好 —— 用于 geo-targeted businesses
- New TLDs（.io、.ai、.dev）对 tech 受众有效但会混淆主流用户
- Premium domains 有 recurring premium renewal fees，不仅仅是更高的初始成本 —— 检查 yearly price

## 注册实践
- 立即启用 auto-renewal —— 因过期丢失的 domains 会在几小时内被 squatters 抢走
- 购买 WHOIS privacy —— 公开注册数据会导致 endless spam 和 social engineering 尝试
- 如果 domain 很重要，注册多年 —— 向搜索引擎表明你是认真的
- 为 registrar accounts 使用专用 email —— 丢失该 email 的访问权限意味着丢失 domain

## DNS 基础
- DNS 变更需要 24-48 小时才能完全传播 —— 相应规划迁移
- TTL（Time To Live）应在迁移前降低，之后提高 —— 正常操作期间低 TTL 浪费资源
- A records 指向 IP addresses，CNAME 指向另一个 domain —— 永远不要 CNAME root domain
- MX records 用于 email，与 web hosting 分开 —— 如果 MX 保持不变，更换 hosts 不需要更改 email

## 安全
- 启用 registrar lock（clientTransferProhibited）—— 防止未授权转移
- DNSSEC 添加 cryptographic verification —— 值得启用，但配置错误会破坏
- Registrar account 必须启用 Two-factor —— domain hijacking 是常见的攻击向量
- Authorization/EPP code 是 transfers 的密码 —— 像凭证一样对待它

## 转移
- 注册或上次转移后 60 天内锁定 —— 提前规划，不能立即转移
- 转移会延长 registration 一年 —— 不是浪费钱
- 在发起前解锁 domain 并获取 auth code —— 缺少任一都会阻塞转移
- 某些 TLDs 有特殊转移规则 —— .uk、.de 等不同于标准流程

## 过期
- Grace period（通常 30 天）允许以正常价格续费 —— 但有风险，网站会宕机
- Redemption period 成本是正常续费的 10-20 倍 —— 昂贵的错误
- Redemption 之后，domain 进入拍卖或开放注册 —— 你已经失去了它
- 带有 backlinks 的过期 domains 会被 spammers 购买 —— 即使未使用也要保护你品牌的 domains

## 多 Domain 策略
- 注册常见拼写错误并 redirect —— 否则 typosquatters 会从你的流量中获利
- 至少考虑 .com + main country TLD —— 其他仅在品牌有价值时才考虑
- Subdomains 免费且即时 —— 不要为每个项目购买 domains，对 experiments 使用 subdomains
- 将 domains 合并到一个 registrar —— 更容易管理，更少的 credential sprawl

## 常见错误
- 通过 web host 而非 dedicated registrar 注册 —— 以后更难迁移，通常更贵
- 让 domains 过期，假设没人在意 —— competitors 和 squatters 监控过期
- 对 critical accounts 使用 registrar 的 free email forwarding —— 与 domain 续费绑定，单点故障
- 不记录哪些 domains 存在于何处 —— 大型组织会丢失 track 并丢失 domains
