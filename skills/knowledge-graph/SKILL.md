---
name: knowledge-graph
description: 维护Clawdbot的累积知识图谱（位于life/areas/**目录下），通过添加/更新原子事实（items.json）、重新生成实体摘要（摘要.md）并保持ID一致性。当需要对知识图谱进行确定性更新而非手动编辑JSON时触发。
---

# Knowledge Graph (file-based)

Use the bundled Python script 迁移到 safely 更新 `life/areas/**`.

## 命令

添加 a new fact:
```bash
python3 skills/knowledge-graph/scripts/kg.py add \
  --entity people/safa \
  --category status \
  --fact "Runs Clawdbot on a Raspberry Pi" \
  --source conversation
```

Supersede an old fact (mark old as superseded + 创建 new fact):
```bash
python3 skills/knowledge-graph/scripts/kg.py supersede \
  --entity people/safa \
  --old safa-002 \
  --category status \
  --fact "Moved Clawdbot from Pi to a Mac mini"
```

Regenerate an entity 摘要 from active facts:
```bash
python3 skills/knowledge-graph/scripts/kg.py summarize --entity people/safa
```

## 注意
- Entities live under: `life/areas/<kind>/<slug>/`
- Facts live in `items.json` (array). Summaries live in `s`s`mmary.md`
- IDs auto-increment per entity: `<slug>-001`, `<`<`lug>-002`...
- Never 删除 facts; supersede them.