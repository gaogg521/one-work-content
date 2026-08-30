---
name: data-validation
description: 基于 schema 跨语言与格式验证数据，支持 JSON Schema、Zod (TypeScript)、Pydantic (Python)，用于 API 请求/响应结构验证、CSV/JSON 完整性检查及服务间数据契约(data contract)。触发词：数据验证(data validation)、JSON Schema、Zod、Pydantic、数据契约(data contract)。
metadata: None
clawdbot: None
emoji: ✅
os:
- linux
- darwin
- win32
requires: None
anyBins:
- node
- python3
- jq
tags:
- API
- Python
- Schema
---

# Data Validation

基于 schema 的跨语言和格式数据验证。涵盖 JSON Schema、Zod (TypeScript)、Pydantic (Python)、API 边界验证、数据契约和完整性检查。

## 何时使用

- 定义 API 请求/响应体的结构
- 在处理前验证用户输入
- 在服务之间建立数据契约
- 在导入前检查 CSV/JSON 文件完整性
- 迁移数据（ETL 是否保留了所有内容？）
- 从 schema 生成类型或文档

## JSON Schema

### 基本 schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["name", "email", "age"],
  "properties": {
    "name": {
      "type": "string",
      "minLength": 1,
      "maxLength": 100
    },
    "email": {
      "type": "string",
      "format": "email"
    },
    "age": {
      "type": "integer",
      "minimum": 0,
      "maximum": 150
    },
    "role": {
      "type": "string",
      "enum": ["user", "admin", "moderator"],
      "default": "user"
    },
    "tags": {
      "type": "array",
      "items": { "type": "string" },
      "uniqueItems": true,
      "maxItems": 10
    },
    "address": {
      "type": "object",
      "properties": {
        "street": { "type": "string" },
        "city": { "type": "string" },
        "zip": { "type": "string", "pattern": "^\\d{5}(-\\d{4})?$" }
      },
      "required": ["street", "city"]
    }
  },
  "additionalProperties": false
}
```

### 常见模式

```json
// 可空字段
{ "type": ["string", "null"] }

// 联合类型（string 或 number）
{ "oneOf": [{ "type": "string" }, { "type": "number" }] }

// 条件：如果 role 是 admin，则要求 permissions
{
  "if": { "properties": { "role": { "const": "admin" } } },
  "then": { "required": ["permissions"] }
}

// 模式属性（动态键）
{
  "type": "object",
  "patternProperties": {
    "^env_": { "type": "string" }
  }
}

// 可复用定义
{
  "$defs": {
    "address": {
      "type": "object",
      "properties": {
        "street": { "type": "string" },
        "city": { "type": "string" }
      }
    }
  },
  "properties": {
    "home": { "$ref": "#/$defs/address" },
    "work": { "$ref": "#/$defs/address" }
  }
}
```

### 命令行验证

```bash
# 使用 ajv-cli (Node.js)
npx ajv-cli validate -s schema.json -d data.json

# 使用 jsonschema (Python)
pip install jsonschema
python3 -c "
import json, jsonschema
schema = json.load(open('schema.json'))
data = json.load(open('data.json'))
jsonschema.validate(data, schema)
print('Valid')
"

# 验证多个文件
for f in data/*.json; do
  npx ajv-cli validate -s schema.json -d "$f" 2>&1 || echo "INVALID: $f"
done
```

## Zod (TypeScript)

### 基本 schema

```typescript
import { z } from 'zod';

// 基本类型
const nameSchema = z.string().min(1).max(100);
const ageSchema = z.number().int().min(0).max(150);
const emailSchema = z.string().email();
const urlSchema = z.string().url();

// 对象
const userSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  age: z.number().int().min(0),
  role: z.enum(['user', 'admin', 'moderator']).default('user'),
  tags: z.array(z.string()).max(10).default([]),
  createdAt: z.string().datetime(),
});

// 从 schema 推断 TypeScript 类型
type User = z.infer<typeof userSchema>;
// { name: string; email: string; age: number; role: "user" | "admin" | "moderator"; ... }

// 验证
const result = userSchema.safeParse(data);
if (result.success) {
  console.log(result.data); // 类型为 User
} else {
  console.log(result.error.issues); // 验证错误
}

// Parse（无效时抛出异常）
const user = userSchema.parse(data);
```

### 高级模式

```typescript
// 可选和可空
const schema = z.object({
  name: z.string(),
  nickname: z.string().optional(),       // string | undefined
  middleName: z.string().nullable(),     // string | null
  suffix: z.string().nullish(),          // string | null | undefined
});

// 转换（先验证再转换）
const dateSchema = z.string().datetime().transform(s => new Date(s));
const trimmed = z.string().trim().toLowerCase();
const parsed = z.string().transform(s => parseInt(s, 10)).pipe(z.number().int());

// 可辨识联合（tagged unions）
const eventSchema = z.discriminatedUnion('type', [
  z.object({ type: z.literal('click'), x: z.number(), y: z.number() }),
  z.object({ type: z.literal('keypress'), key: z.string() }),
  z.object({ type: z.literal('scroll'), delta: z.number() }),
]);

// 递归类型
const categorySchema: z.ZodType<Category> = z.object({
  name: z.string(),
  children: z.lazy(() => z.array(categorySchema)).default([]),
});

// 精细化（自定义验证）
const passwordSchema = z.string()
  .min(8)
  .refine(s => /[A-Z]/.test(s), '必须包含大写字母')
  .refine(s => /[0-9]/.test(s), '必须包含数字')
  .refine(s => /[^a-zA-Z0-9]/.test(s), '必须包含特殊字符');

// 扩展/合并对象
const baseUser = z.object({ name: z.string(), email: z.string() });
const adminUser = baseUser.extend({ permissions: z.array(z.string()) });

// Pick/omit
const createUser = userSchema.omit({ createdAt: true });
const userSummary = userSchema.pick({ name: true, email: true });

// 透传（允许额外字段）
const flexible = userSchema.passthrough();

// 移除未知字段
const strict = userSchema.strict(); // 额外字段时报错
```

### 使用 Zod 进行 API 验证

```typescript
// Express 中间件
import { z } from 'zod';

const createUserBody = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  password: z.string().min(8),
});

app.post('/api/users', (req, res) => {
  const result = createUserBody.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ errors: result.error.issues });
  }
  const { name, email, password } = result.data;
  // ... 创建用户
});

// 查询参数验证
const listParams = z.object({
  page: z.coerce.number().int().min(1).default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  sort: z.enum(['newest', 'oldest', 'name']).default('newest'),
  q: z.string().optional(),
});

app.get('/api/users', (req, res) => {
  const params = listParams.parse(req.query);
  // params.page 是 number 类型，params.sort 是类型化的
});
```

## Pydantic (Python)

### 基本模型

```python
from pydantic import BaseModel, Field, EmailStr, field_validator
from typing import Optional
from datetime import datetime
from enum import Enum

class Role(str, Enum):
    USER = "user"
    ADMIN = "admin"
    MODERATOR = "moderator"

class Address(BaseModel):
    street: str
    city: str
    zip_code: str = Field(pattern=r"^\d{5}(-\d{4})?$")

class User(BaseModel):
    name: str = Field(min_length=1, max_length=100)
    email: EmailStr
    age: int = Field(ge=0, le=150)
    role: Role = Role.USER
    tags: list[str] = Field(default_factory=list, max_length=10)
    address: Optional[Address] = None
    created_at: datetime = Field(default_factory=datetime.now)

    @field_validator("name")
    @classmethod
    def name_must_not_be_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("name 不能为空")
        return v.strip()

# 验证
user = User(name="Alice", email="alice@example.com", age=30)
print(user.model_dump())      # dict
print(user.model_dump_json())  # JSON 字符串

# 验证错误
try:
    User(name="", email="bad", age=-1)
except Exception as e:
    print(e)  # 详细的验证错误
```

### 高级模式

```python
from pydantic import BaseModel, model_validator, ConfigDict
from typing import Literal, Union, Annotated

# 可辨识联合
class ClickEvent(BaseModel):
    type: Literal["click"]
    x: int
    y: int

class KeypressEvent(BaseModel):
    type: Literal["keypress"]
    key: str

Event = Annotated[Union[ClickEvent, KeypressEvent], Field(discriminator="type")]

# 模型级验证（跨字段）
class DateRange(BaseModel):
    start: datetime
    end: datetime

    @model_validator(mode="after")
    def end_after_start(self):
        if self.end <= self.start:
            raise ValueError("end 必须在 start 之后")
        return self

# 严格模式（不进行类型强制转换）
class StrictUser(BaseModel):
    model_config = ConfigDict(strict=True)
    age: int  # "30" 会被拒绝，必须是 int 30

# 别名（在输入中接受不同的字段名）
class APIResponse(BaseModel):
    user_name: str = Field(alias="userName")
    created_at: datetime = Field(alias="createdAt")

    model_config = ConfigDict(populate_by_name=True)

# 计算字段
from pydantic import computed_field

class Order(BaseModel):
    items: list[dict]
    tax_rate: float = 0.08

    @computed_field
    @property
    def total(self) -> float:
        subtotal = sum(i.get("price", 0) * i.get("qty", 1) for i in self.items)
        return round(subtotal * (1 + self.tax_rate), 2)

# 生成 JSON Schema
print(User.model_json_schema())
```

### FastAPI 集成

```python
from fastapi import FastAPI, Query
from pydantic import BaseModel

app = FastAPI()

class CreateUser(BaseModel):
    name: str = Field(min_length=1)
    email: EmailStr
    password: str = Field(min_length=8)

class UserResponse(BaseModel):
    id: int
    name: str
    email: str

@app.post("/api/users", response_model=UserResponse)
async def create_user(body: CreateUser):
    # body 已经过验证
    return {"id": 1, "name": body.name, "email": body.email}

@app.get("/api/users")
async def list_users(
    page: int = Query(default=1, ge=1),
    limit: int = Query(default=20, ge=1, le=100),
    q: str | None = Query(default=None),
):
    # 所有参数均已验证并类型化
    pass
```

## 数据完整性检查

### CSV 验证

```bash
#!/bin/bash
# validate-csv.sh — 检查 CSV 结构和数据质量
FILE="${1:?Usage: validate-csv.sh <file.csv>}"

echo "=== CSV 验证: $FILE ==="

# 行数
ROWS=$(wc -l < "$FILE")
echo "行数: $ROWS (包含表头)"

# 列数一致性
HEADER_COLS=$(head -1 "$FILE" | awk -F',' '{print NF}')
echo "列数 (表头): $HEADER_COLS"

BAD_ROWS=$(awk -F',' -v expected="$HEADER_COLS" 'NR>1 && NF!=expected {count++} END {print count+0}' "$FILE")
if [ "$BAD_ROWS" -gt 0 ]; then
    echo "错误: $BAD_ROWS 行的列数不正确"
    awk -F',' -v expected="$HEADER_COLS" 'NR>1 && NF!=expected {print "  第 "NR" 行: "NF" 列 (预期 "expected")"}' "$FILE" | head -5
else
    echo "列数: 一致"
fi

# 空字段
EMPTY=$(awk -F',' '{for(i=1;i<=NF;i++) if($i=="") count++} END {print count}' "$FILE")
echo "空字段: $EMPTY"

# 重复行
DUPES=$(($(sort "$FILE" | uniq -d | wc -l)))
echo "重复行: $DUPES"

echo "=== 完成 ==="
```

### JSON 验证

```bash
# 检查文件是否为有效 JSON
jq empty data.json && echo "有效的 JSON" || echo "无效的 JSON"

# 验证数组中每个对象的结构
jq -e '
  .[] |
  select(
    (.name | type) != "string" or
    (.email | type) != "string" or
    (.age | type) != "number" or
    .age < 0
  )
' data.json && echo "发现无效记录" || echo "所有记录均有效"

# 检查必填字段
jq -e '.[] | select(.id == null or .name == null)' data.json

# 检查 ID 是否唯一
jq '[.[].id] | length != (. | unique | length)' data.json
# true = 存在重复

# 比较源数据和目标数据的记录数
SRC=$(jq length source.json)
TGT=$(jq length target.json)
echo "源数据: $SRC, 目标数据: $TGT, 是否匹配: $([ "$SRC" = "$TGT" ] && echo yes || echo NO)"
```

### 迁移验证

```python
#!/usr/bin/env python3
"""验证数据迁移是否保留了所有记录。"""
import json
import sys

def validate_migration(source_path, target_path, key_field="id"):
    with open(source_path) as f:
        source = {r[key_field]: r for r in json.load(f)}
    with open(target_path) as f:
        target = {r[key_field]: r for r in json.load(f)}

    missing = set(source) - set(target)
    extra = set(target) - set(source)
    changed = []

    for key in set(source) & set(target):
        if source[key] != target[key]:
            changed.append(key)

    print(f"源记录数: {len(source)}")
    print(f"目标记录数: {len(target)}")
    print(f"目标中缺失: {len(missing)}")
    print(f"目标中多余: {len(extra)}")
    print(f"已变更: {len(changed)}")

    if missing:
        print(f"\n缺失的 ID (前10个): {list(missing)[:10]}")
    if extra:
        print(f"\n多余的 ID (前10个): {list(extra)[:10]}")
    if changed:
        print(f"\n已变更的 ID (前5个): {changed[:5]}")
        for key in changed[:3]:
            print(f"\n  {key}:")
            for field in set(source[key]) | set(target[key]):
                s = source[key].get(field)
                t = target[key].get(field)
                if s != t:
                    print(f"    {field}: {s!r} → {t!r}")

    return len(missing) == 0 and len(extra) == 0

if __name__ == "__main__":
    ok = validate_migration(sys.argv[1], sys.argv[2], sys.argv[3] if len(sys.argv) > 3 else "id")
    sys.exit(0 if ok else 1)
```

## 提示

- 在系统边界（API 端点、文件导入、消息队列）进行验证，而不是在业务逻辑深处。信任内部数据。
- Zod 和 Pydantic 都可以从其定义生成 JSON Schema。可用于文档、OpenAPI 规范和跨语言契约。
- JSON Schema 中的 `additionalProperties: false` 可以捕获字段名拼写错误。在严格的 API 中使用它。
- Pydantic v2 明显快于 v1。当你不希望隐式类型强制转换时，使用 `model_config = ConfigDict(strict=True)`。
- Zod 的 `.safeParse()` 返回结果对象；`.parse()` 抛出异常。在 API 处理程序中使用 `safeParse` 以返回结构化错误。
- 对于 CSV 验证，始终首先检查列数一致性 —— 大多数下游错误都可追溯到列错位。
- 数据迁移验证应比较记录数、检查缺失/多余记录，并抽样检查字段值。仅计数是不够的。
