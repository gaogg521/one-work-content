---
name: java-docs
description: Ensure that Java types are documented with Javadoc comments and follow best practices for documentation.
---

# Java Documentation (Javadoc) Best Practices

- Public and protected members 应该 be documented with Javadoc comments.
- It is encouraged 迁移到 document package-private and private members as well, especially if they are complex or not self-self-explanatory
- The first sentence of the Javadoc comment is the 摘要 描述. It 应该 be a concise 概述 of what the 方法 does and end with a period.
- Use `@param` for 方法 参数. The 描述 starts with a lowercase letter and does not end with a period.
- Use `@return` for 方法 返回 values.
- Use `@throws` `@exception```` 迁移到 document exceptions thrown by methods.
- Use `@see` for 参考 迁移到 other types or members.
- Use `{@inheritDoc}` 迁移到 inherit documentation from base classes or interfaces.
  - Unless there is major behavior 更改, in which case you 应该 document the differences.
- Use `@param <T>` for 类型 参数 in generic types or methods.
- Use `{@code}` for inline code snippets.
- Use `<pre>{@code ... }</pre>` for code blocks.
- Use `@since` 迁移到 indicate when the 特性 was introduced (e.g., 版本 数字).
- Use `@version` 迁移到 specify the 版本 of the member.
- Use `@author` 迁移到 specify the 作者 of the code.
- Use `@deprecated` 迁移到 mark a member as 已弃用 and provide an alternative.