---
name: hidpi-mouse
description: 通用 HiDPI 鼠标点击处理，用于 Linux 桌面自动化(desktop automation)。自动检测 scale factor 或针对任意屏幕分辨率/DPI 进行校准，将 Claude 显示坐标转换为 xdotool 屏幕坐标。触发词：HiDPI 鼠标(mouse)、桌面自动化(desktop automation)、坐标校准(calibration)、xdotool
metadata: None
os:
- linux
requires: None
bins:
- xdotool
- scrot
- python3
user-invocable: False
tags:
- Linux
- 自动化
---

# HiDPI Mouse Skill

通用鼠标坐标处理，用于不同屏幕配置下的桌面自动化。

## 快速开始

```bash
# 在 Claude 显示坐标处点击（自动 scaling）
./scripts/click.sh 500 300

# 首次使用？运行校准以获得最佳精度
./scripts/calibrate.sh
```

## 工作原理

当 Claude 显示截图时，它会将其缩小。此 skill 转换坐标：

```
Claude 显示坐标 → Scale Factor → xdotool 屏幕坐标
```

Scale factor 取决于：
- 屏幕分辨率（1080p、1440p、4K 等）
- DPI 设置（96、144、192 等）
- Claude 的显示 viewport

## 脚本

### click.sh - 在坐标处点击
```bash
./scripts/click.sh <x> <y>           # 自动 scaled 点击
./scripts/click.sh --raw <x> <y>     # 无 scaling（屏幕坐标）
./scripts/click.sh --double <x> <y>  # 双击
./scripts/click.sh --right <x> <y>   # 右键点击
```

### calibrate.sh - 设置与配置
```bash
./scripts/calibrate.sh              # 交互式校准
./scripts/calibrate.sh info         # 显示当前配置
./scripts/calibrate.sh test         # 测试当前 scale
./scripts/calibrate.sh set 2.08     # 手动设置 scale
./scripts/calibrate.sh reset        # 重置为自动检测
```

### detect-scale.sh - 获取 scale factor
```bash
./scripts/detect-scale.sh           # 返回 scale（例如 2.08）
```

### 其他脚本
```bash
./scripts/move.sh <x> <y>           # 移动鼠标
./scripts/drag.sh <x1> <y1> <x2> <y2>  # Drag
./scripts/reliable_click.sh <x> <y> [--window "Name" --relative]
```

## 校准（推荐用于新系统）

为了在你的系统上获得最佳精度：

```bash
./scripts/calibrate.sh
```

这将：
1. 创建一张带已知位置标记点的校准图像
2. 询问你标记点在 Claude 显示中的位置
3. 计算并保存精确的 scale factor

## 常见 Scale Factors

| 屏幕 | DPI | 典型 Scale |
|--------|-----|---------------|
| 1920×1080 | 96 | 1.0 - 1.2 |
| 2560×1440 | 96 | 1.3 - 1.5 |
| 3024×1772 | 192 | 2.08 |
| 3840×2160 | 192 | 2.0 - 2.5 |

## 故障排除

### 点击位置偏移
```bash
# 运行校准
./scripts/calibrate.sh

# 或手动调整
./scripts/calibrate.sh set 2.1  # 尝试不同值
```

### 检查当前配置
```bash
./scripts/calibrate.sh info
```

### 重置所有设置
```bash
./scripts/calibrate.sh reset
rm -f /tmp/hidpi_scale_cache
```

## 配置文件

- `~/.config/hidpi-mouse/scale.conf` - 用户设置的 scale（最高优先级）
- `/tmp/hidpi_scale_cache` - 自动检测的 scale 缓存（1 小时 TTL）

## 通用兼容性

此 skill 自动适配：
- 不同的屏幕分辨率（1080p 到 4K+）
- 不同的 DPI 设置（96、120、144、192 等）
- HiDPI/Retina 显示器
- 多显示器设置（主显示器）

## 使用提示

1. **始终校准** 新系统以获得 100% 精度
2. **重新校准** 如果你更改了显示设置
3. **使用 `--raw`** 如果你已有屏幕坐标
4. **检查 `calibrate.sh info`** 查看当前设置

## 示例工作流

```bash
# 1. 截图
scrot /tmp/screen.png

# 2. 在 Claude 中查看，识别按钮在显示坐标 (500, 300) 处

# 3. 点击它
./scripts/click.sh 500 300

# 4. 如果位置偏移，进行校准
./scripts/calibrate.sh
```

---

*测试环境: Ubuntu/Debian with X11, 各种分辨率和 DPI 设置*
