---
name: falcon
description: 通过 TwexAPI 搜索、阅读和与 Twitter/X 互动。触发词：搜索(search)、阅读(read)、推文(tweet)、用户(user)、关注(follow)
user-invocable: True
command-dispatch: tool
command-tool: Bash
command-arg-mode: raw
metadata: None
openclaw: None
emoji: 🦅
os:
- darwin
- linux
primaryEnv: TWEXAPI_KEY
requires: None
bins:
- curl
- jq
env:
- TWEXAPI_KEY
tags:
- 搜索
---

falcon

使用 falcon 阅读、搜索和与 Twitter/X 互动。

快速开始

    falcon check
    falcon user elonmusk
    falcon tweets elonmusk 5
    falcon read <url-or-id>
    falcon search "bitcoin" 10

阅读用户

    falcon user <username>               单个用户的个人资料信息
    falcon users <u1,u2,...>             查找多个用户（逗号分隔）
    falcon find <keyword> [count]        按关键词搜索用户 (默认: 5)
    falcon followers <username> [count]  列出关注者 (默认: 20)
    falcon following <username> [count]  列出关注 (默认: 20)

阅读推文

    falcon tweets <username> [count]     用户的推文和回复 (默认: 20)
    falcon read <id-or-url> [...]        通过 ID 或 URL 阅读一条或多条推文
    falcon replies <id-or-url> [count]   推文的回复 (默认: 20)
    falcon similar <id-or-url>           查找相似推文
    falcon retweeters <id-or-url> [cnt]  谁转推了这条推文 (默认: 20)

搜索

    falcon search <query> [count]        高级搜索 (默认: 10)
    falcon hashtag <tag> [count]         按标签搜索 (默认: 20)
    falcon cashtag <tag> [count]         按现金标签搜索 (默认: 20)
    falcon trending [country]            热门话题 (默认: 全球)

发帖（先与用户确认）

    falcon tweet "text"
    falcon reply <id-or-url> "text"
    falcon quote <tweet-url> "text"

互动（先与用户确认）

    falcon like <id-or-url>
    falcon unlike <id-or-url>
    falcon retweet <id-or-url>
    falcon bookmark <id-or-url>
    falcon follow <username>
    falcon unfollow <username>

账户

    falcon check                         验证 API key 和 cookie 是否已设置
    falcon balance                       检查剩余 API 积分

认证来源

    TWEXAPI_KEY 环境变量: TwexAPI bearer token (所有命令必需)
    TWITTER_COOKIE 环境变量: Twitter 认证 cookie (写/互动命令必需)

重要说明

    - falcon 脚本位于 {baseDir}/falcon.sh
    - 所有命令接受推文 URL (x.com 或 twitter.com) 或裸推文 ID
    - 在执行任何写或互动命令之前始终与用户确认
    - 搜索接受任何 Twitter 高级搜索语法
    - 标签可以带或不带 # 前缀传递
    - 现金标签可以带或不带 $ 前缀传递
    - 趋势的国家使用 slug 格式: united-states, united-kingdom, japan, 等。
