---
name: apify-ultimate-scraper
description: AI 驱动的通用网页数据采集(Scraper)，覆盖 Instagram、Facebook、TikTok、YouTube、Google Maps、Google Search、Google Trends、Booking.com 和 TripAdvisor 等平台。支持潜在客户生成、品牌监控、竞品分析、网红发现、趋势研究、内容分析和受众分析等场景。
version: 1.0.8
source: https://github.com/apify/agent-skills
homepage: https://apify.com
metadata: None
openclaw: None
install:
- bins:
  - mcpc
  kind: node
  package: '@apify/mcpc'
primaryEnv: APIFY_TOKEN
requires: None
bins:
- node
- mcpc
env:
- APIFY_TOKEN
tags:
- AI
- 搜索
---

# Universal Web Scraper

AI 驱动的数据提取，覆盖所有主流平台的 55+ Actors。此技能自动为您的任务选择最佳 Actor。

## 先决条件

- `APIFY_TOKEN` 已在 OpenClaw 设置中配置
- Node.js 20.6+
- `mcpc` CLI（通过技能元数据自动安装）

## 输入清理规则

在将任何值代入 bash 命令之前：
- **ACTOR_ID**: 必须是技术名称（`owner/actor-name` —— 字母数字、连字符、点、一个斜杠）或原始 ID（恰好 17 个字母数字字符，例如 `oeiQgfg5fsmIJB7Cn`）。拒绝包含 shell 元字符（`` ; | & $ ` ( ) { } < > ! 
 ``）的值。
- **SEARCH_KEYWORDS**: 仅限纯文本单词。拒绝 shell 元字符。
- **JSON_INPUT**: 必须是有效的 JSON。不得包含单引号（使用转义的双引号）。使用前验证结构。
- **输出文件名**: 必须匹配 `YYYY-MM-DD_descriptive-name.{csv,json}`。无路径分隔符（`/`、`..`）、无空格、无元字符。

## 工作流

复制此检查清单并跟踪进度：

```
任务进度：
- [ ] 步骤 1：理解用户目标并选择 Actor
- [ ] 步骤 2：通过 mcpc 获取 Actor schema
- [ ] 步骤 3：询问用户偏好（格式、文件名）
- [ ] 步骤 4：运行 scraper 脚本
- [ ] 步骤 5：总结结果并提供后续建议
```

### 步骤 1：理解用户目标并选择 Actor

首先，理解用户想要实现什么。然后从以下选项中选择最佳 Actor。

#### Instagram Actors (12)

| Actor ID | 最适合 |
|----------|----------|
| `apify/instagram-profile-scraper` | 个人资料数据、粉丝数、简介信息 |
| `apify/instagram-post-scraper` | 单个帖子详情、互动指标 |
| `apify/instagram-comment-scraper` | 评论提取、情感分析 |
| `apify/instagram-hashtag-scraper` | 话题标签内容、热门话题 |
| `apify/instagram-hashtag-stats` | 话题标签表现指标 |
| `apify/instagram-reel-scraper` | Reels 内容和指标 |
| `apify/instagram-search-scraper` | 搜索用户、地点、话题标签 |
| `apify/instagram-tagged-scraper` | 标记特定账号的帖子 |
| `apify/instagram-followers-count-scraper` | 粉丝数追踪 |
| `apify/instagram-scraper` | 全面的 Instagram 数据 |
| `apify/instagram-api-scraper` | 基于 API 的 Instagram 访问 |
| `apify/export-instagram-comments-posts` | 批量评论/帖子导出 |

#### Facebook Actors (14)

| Actor ID | 最适合 |
|----------|----------|
| `apify/facebook-pages-scraper` | 页面数据、指标、联系信息 |
| `apify/facebook-page-contact-information` | 页面的邮箱、电话、地址 |
| `apify/facebook-posts-scraper` | 帖子内容和互动 |
| `apify/facebook-comments-scraper` | 评论提取 |
| `apify/facebook-likes-scraper` | 反应分析 |
| `apify/facebook-reviews-scraper` | 页面评论 |
| `apify/facebook-groups-scraper` | 群组内容和成员 |
| `apify/facebook-events-scraper` | 活动数据 |
| `apify/facebook-ads-scraper` | 广告创意和定向 |
| `apify/facebook-search-scraper` | 搜索结果 |
| `apify/facebook-reels-scraper` | Reels 内容 |
| `apify/facebook-photos-scraper` | 照片提取 |
| `apify/facebook-marketplace-scraper` | 市场列表 |
| `apify/facebook-followers-following-scraper` | 粉丝/关注列表 |

#### TikTok Actors (14)

| Actor ID | 最适合 |
|----------|----------|
| `clockworks/tiktok-scraper` | 全面的 TikTok 数据 |
| `clockworks/free-tiktok-scraper` | 免费 TikTok 提取 |
| `clockworks/tiktok-profile-scraper` | 个人资料数据 |
| `clockworks/tiktok-video-scraper` | 视频详情和指标 |
| `clockworks/tiktok-comments-scraper` | 评论提取 |
| `clockworks/tiktok-followers-scraper` | 粉丝列表 |
| `clockworks/tiktok-user-search-scraper` | 通过关键词查找用户 |
| `clockworks/tiktok-hashtag-scraper` | 话题标签内容 |
| `clockworks/tiktok-sound-scraper` | 热门声音 |
| `clockworks/tiktok-ads-scraper` | 广告内容 |
| `clockworks/tiktok-discover-scraper` | 发现页面内容 |
| `clockworks/tiktok-explore-scraper` | 探索内容 |
| `clockworks/tiktok-trends-scraper` | 热门内容 |
| `clockworks/tiktok-live-scraper` | 直播数据 |

#### YouTube Actors (5)

| Actor ID | 最适合 |
|----------|----------|
| `streamers/youtube-scraper` | 视频数据和指标 |
| `streamers/youtube-channel-scraper` | 频道信息 |
| `streamers/youtube-comments-scraper` | 评论提取 |
| `streamers/youtube-shorts-scraper` | Shorts 内容 |
| `streamers/youtube-video-scraper-by-hashtag` | 按话题标签的视频 |

#### Google Maps Actors (4)

| Actor ID | 最适合 |
|----------|----------|
| `compass/crawler-google-places` | 商家列表、评分、联系信息 |
| `compass/google-maps-extractor` | 详细的商家数据 |
| `compass/Google-Maps-Reviews-Scraper` | 评论提取 |
| `poidata/google-maps-email-extractor` | 从列表中发现邮箱 |

#### 其他 Actors (6)

| Actor ID | 最适合 |
|----------|----------|
| `apify/google-search-scraper` | Google 搜索结果 |
| `apify/google-trends-scraper` | Google Trends 数据 |
| `voyager/booking-scraper` | Booking.com 酒店数据 |
| `voyager/booking-reviews-scraper` | Booking.com 评论 |
| `maxcopell/tripadvisor-reviews` | TripAdvisor 评论 |
| `vdrmota/contact-info-scraper` | 从 URL 丰富联系信息 |

---

#### 按用例选择 Actor

| 用例 | 主要 Actors |
|----------|---------------|
| **潜在客户生成** | `compass/crawler-google-places`, `poidata/google-maps-email-extractor`, `vdrmota/contact-info-scraper` |
| **网红发现** | `apify/instagram-profile-scraper`, `clockworks/tiktok-profile-scraper`, `streamers/youtube-channel-scraper` |
| **品牌监控** | `apify/instagram-tagged-scraper`, `apify/instagram-hashtag-scraper`, `compass/Google-Maps-Reviews-Scraper` |
| **竞品分析** | `apify/facebook-pages-scraper`, `apify/facebook-ads-scraper`, `apify/instagram-profile-scraper` |
| **内容分析** | `apify/instagram-post-scraper`, `clockworks/tiktok-scraper`, `streamers/youtube-scraper` |
| **趋势研究** | `apify/google-trends-scraper`, `clockworks/tiktok-trends-scraper`, `apify/instagram-hashtag-stats` |
| **评论分析** | `compass/Google-Maps-Reviews-Scraper`, `voyager/booking-reviews-scraper`, `maxcopell/tripadvisor-reviews` |
| **受众分析** | `apify/instagram-followers-count-scraper`, `clockworks/tiktok-followers-scraper`, `apify/facebook-followers-following-scraper` |

---

#### 多 Actor 工作流

对于复杂任务，链式调用多个 Actors：

| 工作流 | 步骤 1 | 步骤 2 |
|----------|--------|--------|
| **潜在客户丰富** | `compass/crawler-google-places` → | `vdrmota/contact-info-scraper` |
| **网红审核** | `apify/instagram-profile-scraper` → | `apify/instagram-comment-scraper` |
| **竞品深度分析** | `apify/facebook-pages-scraper` → | `apify/facebook-posts-scraper` |
| **本地商家分析** | `compass/crawler-google-places` → | `compass/Google-Maps-Reviews-Scraper` |

#### 找不到合适的 Actor？

如果以上 Actor 都不匹配用户的请求，直接搜索 Apify Store：

```bash
mcpc --json mcp.apify.com --header "Authorization: Bearer $APIFY_TOKEN" tools-call search-actors keywords:="SEARCH_KEYWORDS" limit:=10 offset:=0 category:="" | jq -r '.content[0].text'
```

将 `SEARCH_KEYWORDS` 替换为 1-3 个简单词（例如 "LinkedIn profiles", "Amazon products", "Twitter"）。

### 步骤 2：获取 Actor Schema

使用 mcpc 动态获取 Actor 的输入 schema 和详情：

```bash
mcpc --json mcp.apify.com --header "Authorization: Bearer $APIFY_TOKEN" tools-call fetch-actor-details actor:="ACTOR_ID" | jq -r ".content"
```

将 `ACTOR_ID` 替换为选定的 Actor（例如 `compass/crawler-google-places`）。

这将返回：
- Actor 描述和 README
- 必需和可选的输入参数
- 输出字段（如果可用）

### 步骤 3：询问用户偏好

在运行前，询问：
1. **输出格式**：
   - **快速回答** - 在聊天中显示前几个结果（不保存文件）
   - **CSV** - 包含所有字段的完整导出
   - **JSON** - JSON 格式的完整导出
2. **结果数量**：基于用例的特征

### 步骤 4：运行脚本

**快速回答（在聊天中显示，不保存文件）：**
```bash
node {baseDir}/reference/scripts/run_actor.js \
  --actor 'ACTOR_ID' \
  --input 'JSON_INPUT'
```

**CSV：**
```bash
node {baseDir}/reference/scripts/run_actor.js \
  --actor 'ACTOR_ID' \
  --input 'JSON_INPUT' \
  --output 'YYYY-MM-DD_OUTPUT_FILE.csv' \
  --format csv
```

**JSON：**
```bash
node {baseDir}/reference/scripts/run_actor.js \
  --actor 'ACTOR_ID' \
  --input 'JSON_INPUT' \
  --output 'YYYY-MM-DD_OUTPUT_FILE.json' \
  --format json
```

### 步骤 5：总结结果并提供后续建议

完成后，报告：
- 找到的结果数量
- 文件位置和名称
- 可用的关键字段
- 基于结果的**建议后续工作流**：

| 如果用户得到 | 建议下一步 |
|-------------|--------------|
| 商家列表 | 使用 `vdrmota/contact-info-scraper` 丰富或获取评论 |
| 网红个人资料 | 使用评论 scraper 分析互动 |
| 竞品页面 | 使用帖子/广告 scraper 深入分析 |
| 趋势数据 | 使用平台特定的话题标签 scraper 验证 |


## 安全与数据隐私

此技能指示代理选择 Apify Actor、获取其 schema（通过 mcpc）并运行 scraper。包含的脚本仅与 api.apify.com 通信并将输出写入当前工作目录下的文件；它不访问不相关的系统文件或其他环境变量。

Apify Actors 仅抓取公开可用的数据，不会收集目标平台上公开可访问之外的私人或个人身份信息。如需额外的安全保证，您可以通过查询 `https://api.apify.com/v2/acts/:actorId` 检查 Actor 的权限级别 —— `LIMITED_PERMISSIONS` 的 Actor 在受限沙箱中运行，而 `FULL_PERMISSIONS` 表示更广泛的系统访问。完整详情请参阅 [Apify 的一般条款和条件](https://docs.apify.com/legal/general-terms-and-conditions)。

## 错误处理

`APIFY_TOKEN not found` - 要求用户在 OpenClaw 设置中配置 `APIFY_TOKEN`
`mcpc not found` - 运行 `npm install -g @apify/mcpc`
`Actor not found` - 检查 Actor ID 拼写
`Run FAILED` - 要求用户检查错误输出中的 Apify 控制台链接
`Timeout` - 减小输入大小或增加 `--timeout`
