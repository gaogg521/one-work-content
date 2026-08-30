---
name: feishu-native-emoji
description: 提供对飞书原生表情符号集（例如 [Smile], [Like]）的访问，以实现更真实的交互。
tags:
- 飞书
- emoji
- ui
---

# 🎭 Feishu Native Emoji

This skill manages the mapping and usage of Feishu's native emoji codes (e.g., `[微笑]`, `[捂脸]`) instead of standard unicode emojis.

## Usage

This is primarily a passive resource for the Agent to "inject" personality into messages.

### Resources
- `emoji_list.txt`: Raw list of supported codes.

## Integration
When constructing messages for Feishu, prefer using codes from `emoji_list.txt`.
