---
name: parallel-ai-research
description: 对主题进行开放式研究，构建活的 markdown 文档。支持交互式研究和深度异步研究两种模式。
tags:
- AI
- 搜索
- 研究
---

# Research Skill

## 描述
对一个主题进行开放式研究，构建一个活的 markdown 文档。对话是短暂的；文档才是重要的。

## 触发条件
当用户想要时激活：
- 研究一个主题、想法或问题
- 在承诺构建之前探索某事
- 调查选项、模式或方法
- 创建 "research doc" 或 "investigation"
- 对复杂主题运行深度异步研究

## 研究目录

每个研究主题都有自己的文件夹：
```
~/.openclaw/workspace/research/<topic-slug>/
├── prompt.md          # 原始研究问题/提示
├── research.md        # 主要发现（Parallel 输出或交互式笔记）
├── research.pdf       # PDF 导出（生成时）
└── ...                # 任何其他相关文件（数据、图像等）
```

---

## 两种研究模式

### 1. 交互式研究（默认）
对于你在对话中一起探索的主题。你搜索、综合并实时更新文档。

### 2. 深度研究（异步）
对于需要全面调查的复杂主题。通过 `parallel-research` CLI 使用 Parallel AI API。需要分钟到小时，返回详细的 markdown 报告。

**何时使用深度研究：**
- 市场分析、竞争格局
- 需要大量来源收集的技术深度研究
- 受益于并行探索的多方面问题
- 当用户说 "deep research" 或想要全面覆盖时

---

## 交互式研究工作流

### 1. 初始化研究

1. **在 `~/.openclaw/workspace/research/<topic-slug>/` 创建研究文件夹**

2. **使用原始问题创建 prompt.md**：
   ```markdown
   # <Topic Title>

   > <核心问题或好奇心>

   **Started:** <date>
   ```

3. **使用工作结构创建 research.md**：
   ```markdown
   # <Topic Title>

   **Status:** Active Research
   **Started:** <date>
   **Last Updated:** <date>

   ---

   ## Open Questions
   - <要探索的初始问题>

   ## Findings
   <!-- 研究时填充 -->

   ## Options / Approaches
   <!-- 如果比较解决方案 -->

   ## Resources
   <!-- Links、references、sources -->

   ## Next Steps
   <!-- 接下来探索什么，或 "graduate to project" -->
   ```

4. **与用户确认** - 显示文件夹已创建并询问首先探索什么。

### 2. 研究循环

对于每次交流：

1. **进行研究** - Web search、fetch docs、explore code
2. **更新文档** - 添加发现、移动已回答的问题、添加来源
3. **展示进度** - 注意添加了什么（不要重复所有内容）
4. **提示下一步方向** - 以问题或建议结尾

**关键行为：**
- 更新现有部分而不是创建新的
- 对发现使用 bullet points；对摘要使用 prose
- 注明不确定性（"seems like"、"according to X"、"unverified"）
- 尽可能链接到来源

### 3. 综合检查点

每 5-10 次交流，主动提出：
- 撰写 "Current Understanding" 摘要
- 修剪冗余的发现
- 如果内容冗长则重新组织
- 检查盲点

### 4. 完成

当研究完成时，更新 `research.md` 中的状态：

- **"Status: Complete"** —— 完成，作为参考保留在原处
- **"Status: Ongoing"** —— 活文档，会随时间更新

**如果研究是专门用于构建项目的：**
- 作为项目 spec 升级到 `~/specs/<project-name>.md`
- 或根据发现直接创建项目
- 将状态更新为 **"Status: Graduated → ~/specs/..."**

大多数研究只是研究 —— 它不需要成为 spec。只有当你真的从中构建东西时才升级。

---

## 深度研究工作流

### 1. 开始深度研究

```bash
parallel-research create "Your research question" --processor ultra --wait
```

**Processor 选项：**
- `lite`、`base`、`core`、`pro`、`ultra`（默认）、`ultra2x`、`ultra4x`、`ultra8x`
- 添加 `-fast` 后缀以速度优先于深度：`ultra-fast`、`pro-fast` 等

**选项：**
- `-w, --wait` —— 等待完成并显示结果
- `-p, --processor` —— 选择 processor tier
- `-j, --json` —— 原始 JSON 输出

### 2. 设置自动检查（OpenClaw）

创建任务后，设置一个 cron job 来检查结果并将其交付给用户。使用 `deleteAfterRun: true` 以便它自动清理。

**⚠️ 关键：始终正确计算 `atMs`！**

```bash
# 获取当前时间戳（毫秒）并添加 15 分钟（900000 毫秒）
date +%s%3N  # 当前时间，epoch ms
# 示例：1770087600000 + 900000 = 1770088500000
```

**始终验证计划的时间在未来且年份正确：**
```bash
date -d @$((1770088500000/1000))  # 应显示大约 15 分钟后的时间，正确的年份
```

```json
{
  "action": "add",
  "job": {
    "name": "Check research: <topic>",
    "schedule": {"kind": "at", "atMs": <VERIFY: 必须是当前 epoch ms + delay>},
    "sessionTarget": "isolated",
    "payload": {
      "kind": "agentTurn",
      "message": "Check research task <run_id>. Run: parallel-research result <run_id>. If complete, summarize key findings. If still running, reschedule another check in 10 min.",
      "deliver": true,
      "channel": "<source channel, e.g. telegram>",
      "to": "<source chat/topic, e.g. -1001234567890:topic:123>"
    },
    "deleteAfterRun": true
  }
}
```

**关键点：**
- 使用 `cron` 工具并设置 `action: "add"`
- **始终验证 `atMs` 是否正确** —— 运行 `date -d @$((atMs/1000))` 确认年份和时间
- `atMs` 应该是从现在起约 10-15 分钟（ultra processor）或约 5 分钟（fast processors）
- `deleteAfterRun: true` 在成功完成后移除 job
- 交付回请求研究的同一 channel/topic
- 如果仍在运行，cron job 可以创建另一个检查
- `PARALLEL_API_KEY` 作为 env var 可用 —— 无需内联

### 3. 手动检查（如需要）

```bash
parallel-research status <run_id>
parallel-research result <run_id>
```

### 4. 保存到研究文件夹

创建研究文件夹并保存结果：
```
~/.openclaw/workspace/research/<topic-slug>/
├── prompt.md          # 原始问题 + run metadata
├── research.md        # 完整的 Parallel 输出
```

**prompt.md** 应包含：
```markdown
# <Topic Title>

> <原始研究问题>

**Run ID:** <run_id>
**Processor:** <processor>
**Started:** <date>
**Completed:** <date>
```

**research.md** 包含完整的 Parallel 输出，以及任何后续笔记。

---

## PDF 导出

使用 `export-pdf` 脚本将研究文档转换为 PDF：

```bash
export-pdf ~/.openclaw/workspace/research/<topic-slug>/research.md
# 创建：~/.openclaw/workspace/research/<topic-slug>/research.pdf
```

或指定自定义输出路径：
```bash
export-pdf research.md ~/Desktop/output.pdf
```

**注意：** Tables 渲染为堆叠的行（PyMuPDF 限制）。对于研究文档来说可以接受。

---

## 命令

- **"new research: <topic>"** - 启动交互式研究文档
- **"deep research: <topic>"** - 启动异步深度研究
- **"show doc"** / **"show research"** - 显示当前研究文件
- **"summarize"** - 综合检查点
- **"graduate"** - 将研究推进到下一阶段
- **"archive"** - 标记为完整参考
- **"export pdf"** - 导出为 PDF
- **"check research"** - 检查待处理的深度研究任务的状态

---

## 文档原则

- **Atomic findings** - 每个 bullet 一个见解
- **Link everything** - Sources、docs、repos
- **Capture context** - 我们为什么看这个？
- **Note confidence** - 不确定时使用限定词
- **Date important findings** - 特别是对于快速变化的主题

---

## 设置

查看 `SETUP.md` 以进行首次安装：
- `parallel-research` CLI
- PDF 导出工具（pandoc、PyMuPDF）
