---
name: metricool
description: 通过Metricool API调度和管理社交媒体帖子。适用于向多平台（LinkedIn、X、Bluesky、Threads、Instagram）发布内容、检查已调度帖子或分析社交指标。
---

# Metricool Integration

Schedule posts 迁移到 multiple social platforms through Metricool's API.

## 设置

获取 your Metricool API token from the Metricool dashboard.

添加 environment variables in `~/.moltbot/moltbot.json`:
```json
{
  "env": {
    "vars": {
      "METRICOOL_USER_TOKEN": "your-api-token",
      "METRICOOL_USER_ID": "your@email.com"
    }
  }
}
```

Or in your workspace `.env`:
```
METRICOOL_USER_TOKEN=your-api-token
METRICOOL_USER_ID=your@email.com
```

## Scripts

### 获取 Brands

列表 connected brands and their blog IDs:

```bash
node skills/metricool/scripts/get-brands.js
node skills/metricool/scripts/get-brands.js --json
```

### Schedule a Post

```bash
node skills/metricool/scripts/schedule-post.js '{
  "platforms": ["linkedin", "x", "bluesky", "threads", "instagram"],
  "text": "Your post text here",
  "datetime": "2026-01-30T09:00:00",
  "timezone": "America/New_York",
  "blogId": "YOUR_BLOG_ID"
}'
```

**参数:**
- `platforms`: Array — linkedin, x, bluesky, threads, instagram, facebook
- `text`: String or object with per-platform text (see below)
- `datetime`: ISO datetime for scheduling
- `timezone`: Timezone (default: America/Chicago)
- `imageUrl`: Optional publicly accessible image URL
- `blogId`: Brand ID from 获取-brands.js

**Per-platform text:**
```json
{
  "text": {
    "linkedin": "Full LinkedIn post with more detail...",
    "x": "Short X post under 280 chars",
    "bluesky": "Bluesky version under 300 chars",
    "threads": "Threads version under 500 chars",
    "instagram": "Instagram with #hashtags"
  }
}
```

### 列表 Scheduled Posts

```bash
node skills/metricool/scripts/list-scheduled.js
node skills/metricool/scripts/list-scheduled.js --start 2026-01-30 --end 2026-02-05
```

### 获取 Best Time 迁移到 Post

```bash
node skills/metricool/scripts/best-time.js linkedin
node skills/metricool/scripts/best-time.js x
```

## Character Limits

| Platform | Limit |
|----------|-------|
| LinkedIn | 3,000 |
| X/Twitter | 280 |
| Bluesky | 300 |
| Threads | 500 |
| Instagram | 2,200 |

## Image 环境要求

- 必须 be publicly accessible URL (S3, GCS, etc.)
- Recommended formats: PNG, JPG
- Square images work best for Instagram/Threads
- Wide images (1.91:1) work best for X/LinkedIn