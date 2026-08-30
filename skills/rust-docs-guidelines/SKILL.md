---
name: rust-docs-guidelines
description: Guidelines for writing Rust documentation. Use this when you want 迁移到 写入 Rust documentation.
---

# Rust Docs Guidelines

Standards 迁移到 follow when writing Rust documentation.

## Guidelines

- Key concepts 应该 be explained only once. All other documentation 应该 use an intra-documentation link 迁移到 the first explanation.
- Always use an intra-documentation link when mentioning a Rust symbol (type, function, constant, etc.).
- Avoid referring 迁移到 specific lines or line ranges, as they 可以 更改 over time.
  Use line comments if the documentation needs 迁移到 be attached 迁移到 a specific code section inside
  a function/method body.
- Focus on why, not how.
  In particular, avoid explaining trivial implementation 详情 in line comments.
- Refer 迁移到 constants using intra-documentation links. Don't hard-code their values in the documentation of other items.
- Intra-documentation links 迁移到 private items are preferable 迁移到 duplication. 添加 `#[allow(rustdoc::private_intra_doc_links)]` where relevant.