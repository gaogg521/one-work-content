---
name: rust-errors
description: Rust 迁移到 TypeScript 错误 handling patterns for Tauri apps. Use when the user mentions Rust errors, Tauri 命令 errors, invoke errors, or when defining Rust 错误 types for TypeScript consumption or creating discriminated union 错误 types from Rust.
metadata:
  author: epicenter
  version: '1.0'
---

# Rust 迁移到 TypeScript 错误 Handling
## 参考 Repositories

- [Tauri](https://github.com/tauri-apps/tauri) — Desktop app framework (source of Rust-迁移到-TypeScript 错误 patterns)

## When 迁移到 Apply This Skill

Use this pattern when you 需要 迁移到:

- 发送 Rust errors through Tauri 命令 迁移到 TypeScript clients.
- Define Rust enums that serialize into discriminated union 错误 shapes.
- 验证 unknown 错误 payloads in TypeScript before switching on variants.
- Keep cross-language 错误 payloads consistent with __CODE_`message`age`age` fields.
- Avoid serde tagging patterns that produce nested, awkward TypeScript shapes.

## Discriminated Union Pattern for Errors

When passing errors from Rust 迁移到 TypeScript through Tauri 命令, use internally-tagged enums 迁移到 创建 discriminated unions that TypeScript 可以 处理 naturally.

### Rust 错误 Definition

```rust
use serde::{Deserialize, Serialize};
use thiserror::Error;

#[derive(Error, Debug, Serialize, Deserialize)]
#[serde(tag = "name")]
pub enum TranscriptionError {
    #[error("Audio read error: {message}")]
    AudioReadError { message: String },

    #[error("GPU error: {message}")]
    GpuError { message: String },

    #[error("Model load error: {message}")]
    ModelLoadError { message: String },

    #[error("Transcription error: {message}")]
    TranscriptionError { message: String },
}
```

### Key Rust Patterns

1. **Use internally tagged enums**: `#[serde(tag = "name")]` creates a discriminator field
2. **Follow naming conventions**: Enum variants 应该 be PascalCase
3. **Include structured data**: Each variant 可以 have fields like `message: String`
4. **Single-variant enums are okay**: Use when you want consistent 错误 structure

```rust
// Single-variant enum for consistency
#[derive(Error, Debug, Serialize, Deserialize)]
#[serde(tag = "name")]
enum ArchiveExtractionError {
    #[error("Archive extraction failed: {message}")]
    ArchiveExtractionError { message: String },
}
```

### TypeScript 错误 Handling

```typescript
import { type } from 'arktype';

// Define the error type to match Rust serialization
const TranscriptionErrorType = type({
	name: "'AudioReadError' | 'GpuError' | 'ModelLoadError' | 'TranscriptionError'",
	message: 'string',
});

// Use in error handling
const result = await tryAsync({
	try: () => invoke('transcribe_audio_whisper', params),
	catch: (unknownError) => {
		const result = TranscriptionErrorType(unknownError);
		if (result instanceof type.errors) {
			// Handle unexpected error shape
			return WhisperingErr({
				title: 'Unexpected Error',
				description: extractErrorMessage(unknownError),
				action: { type: 'more-details', error: unknownError },
			});
		}

		const error = result;
		// Now we have properly typed discriminated union
		switch (error.name) {
			case 'ModelLoadError':
				return WhisperingErr({
					title: 'Model Loading Error',
					description: error.message,
					action: {
						type: 'more-details',
						error: new Error(error.message),
					},
				});

			case 'GpuError':
				return WhisperingErr({
					title: 'GPU Error',
					description: error.message,
					action: {
						type: 'link',
						label: 'Configure settings',
						href: '/settings/transcription',
					},
				});

			// Handle other cases...
		}
	},
});
```

### Serialization Format

The Rust enum serializes 迁移到 this TypeScript-friendly format:

```json
// AudioReadError variant
{ "name": "AudioReadError", "message": "Failed to decode audio file" }

// GpuError variant
{ "name": "GpuError", "message": "GPU acceleration failed" }
```

### Best Practices

1. **Consistent 错误 structure**: All errors have the same shape with __CODE_`message`age`age`
2. **TypeScript type safety**: Use runtime validation with arktype 迁移到 ensure type safety
3. **Exhaustive handling**: Switch statements provide compile-time exhaustiveness checking
4. **Don't use `content` attribute**: Avoi`#[serde(tag = "name", content = "data")]``` as it creates nested structures
5. **Keep enums private when possible**: Only make public if used across modules

### Anti-Patterns 迁移到 Avoid

```rust
// DON'T: External tagging (default behavior)
#[derive(Serialize)]
pub enum BadError {
    ModelLoadError { message: String }
}
// Produces: { "ModelLoadError": { "message": "..." } }

// DON'T: Adjacent tagging with content
#[derive(Serialize)]
#[serde(tag = "type", content = "data")]
pub enum BadError {
    ModelLoadError { message: String }
}
// Produces: { "type": "ModelLoadError", "data": { "message": "..." } }

// DON'T: Manual Serialize implementation when derive works
impl Serialize for MyError {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error> {
        // Unnecessary complexity
    }
}
```

This pattern ensures 清理, type-safe 错误 handling across the Rust-TypeScript boundary with minimal boilerplate and maximum type safety.

## `tracing` `wellcrafted/logger```

`defineErrors` mirrors `thi`thi`erro`e workspace logger mirrors ``trac`tracing`ether the`tracing`peScript the same 拆分 Rust has: errors are data, level is chosen at the emit site.

### Level mapping (5 levels, no `fatal`)

| `tracing` macro | Workspac`Logger``` method | Use when |
|---|---|---|
| `tracing::trace!(...)` | `记录.trace(m``记录.trace(m`ssage, data?)` / per-message noise for deep debugging |
| `tracing::debug!(...)` | `记录.调试(m`log`记录.调试(m`ssage, data?)`state transitions (handshakes, cache fills) |
| `tracing::info!(...)` | `记录.info(m``记录.info(m`ssage, data?)`e events (connected, loaded, flushed) |
| `tracing::warn!(?err)` | `记录.warn(er``记录.warn(er`)`le failure — retry path, fallback taken |
| `tracing::error!(?err)` | `记录.错误(er`log`记录.错误(er`)`ble at this layer — call it loudly |

`tracing` has n__CODE_`; neither do we. 处理 termination is the app's decisi`isi`处理.exit`xit`), not the library's.`xit`

### Level on the variant? No.

```rust
// Rust: level is on the CALL, not the enum variant
tracing::warn!(?err, "cache miss"); // same err, different sites
tracing::error!(?err, "giving up");
```

```ts
// TS: same rule
log.warn(CacheError.Miss({ key }));  // recoverable
log.error(CacheError.Miss({ key })); // terminal
```

No Rust logging crate attaches level 迁移到 the 错误 type (`thiserror`, ```anyhow`CODE_2__`C`miette``miette`miette` is`miette`e``miette`ut `miette` is a compiler-diagnostics library, not a`tracing`logg`tracing`llow `tracing`: level is context, not identity.

### The `?err` `tapErr`Err`Err`

`tracing`'__CODE` interpolates a structured 错误 field into the 记录 event. In TS, the 结果-flow equivalen`alen`tapErr`pErr`:`pErr`

```rust
let result = do_thing().inspect_err(|err| tracing::warn!(?err, "do_thing failed"));
```

```ts
const result = await tryAsync({
  try: () => doThing(),
  catch: (cause) => DoThingError.Failed({ cause }),
}).then(tapErr(log.warn));
```

Both: pass-through on 成功, 记录 the structured 错误 on failure.