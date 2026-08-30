---
name: clauditor
description: 用于 Clawdbot 代理的防篡改审计看门狗。检测并记录可疑的文件系统活动，并生成 HMAC 链式证据。
homepage: https://github.com/apollostreetcompany/clauditor
metadata: None
clawdbot: None
emoji: 🛡️
requires: None
bins:
- cargo
- systemctl
os: linux
tags:
- 安全
---

# Clauditor

安全看门狗，监控文件系统活动并创建防篡改日志。即使 Clawdbot 被入侵，也无法停止看门狗、伪造条目或删除证据。

## 安装（引导式）

使用 CLI 向导引导用户完成安装：

```bash
cd /path/to/clauditor
cargo build --release
```

### 代理工作流

1. **检查状态：** `./target/release/clauditor wizard status`
   - 返回 JSON，包含 `current_step`（1-6）和 `complete`（true/false）

2. **获取下一步：** `./target/release/clauditor wizard next`
   - 显示它做什么、为什么重要，以及要运行的确切 COMMAND

3. **用户运行 sudo 命令**

4. **验证：** `./target/release/clauditor wizard verify`
   - 确认步骤已完成

5. **重复直到 `complete: true`**

### 示例交互

```
用户："安装 clauditor"

代理：[运行 wizard status → current_step: 1]
代理："第 1 步，共 6 步：创建系统用户

这将创建一个运行看门狗的 'sysaudit' 用户。即使你被入侵，也无法杀死另一个用户拥有的进程。

运行以下命令：
sudo useradd --system --shell /usr/sbin/nologin --no-create-home sysaudit

完成后告诉我。"

用户："done"

代理：[运行 wizard verify → success]
代理：[运行 wizard next → step 2...]
```

## 安装后

### 检查状态
```bash
systemctl status systemd-journaldd
```

### 生成摘要
```bash
./target/release/clauditor digest \
  --log /var/lib/.sysd/.audit/events.log \
  --key /etc/sysaudit/key \
  --format markdown
```

## 快速安装（高级用户）

```bash
sudo bash wizard/wizard.sh
```

## 配置

- 配置：`/etc/sysaudit/config.toml`
- 密钥：`/etc/sysaudit/key`
- 日志：`/var/lib/.sysd/.audit/events.log`

编辑配置以自定义 `watch_paths` 和 `target_uid`。
