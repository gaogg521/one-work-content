---
name: oracle-sqlwrite
description: 编写 Oracle SQL 和 PL/SQL，使用正确的语法、hints 和性能模式。
---

## 语法差异

- `ROWNUM` 用于限制行数—`WHERE ROWNUM <= 10`；12c+ 支持 `FETCH FIRST 10 ROWS ONLY`
- `DUAL` 表用于表达式—`SELECT sysdate FROM dual`
- `VARCHAR2` 而非 `VARCHAR`—VARCHAR 是保留字，VARCHAR2 是标准
- 字符串拼接使用 `||`—不用于多个值的 CONCAT
- 空字符串等于 NULL—`'' IS NULL` 为 true；这会破坏来自其他数据库的逻辑

## 分页

- ROWNUM 在 ORDER BY 之前分配—包装在子查询中：`SELECT * FROM (SELECT ... ORDER BY x) WHERE ROWNUM <= 10`
- Offset 需要嵌套子查询：`SELECT * FROM (SELECT a.*, ROWNUM rn FROM (...) a WHERE ROWNUM <= 20) WHERE rn > 10`
- 12c+：`OFFSET 10 ROWS FETCH NEXT 10 ROWS ONLY`—更简洁，可用时使用

## NULL 处理

- `NVL(col, default)` 用于 null 替换—对于两个参数比 COALESCE 更快
- `NVL2(col, if_not_null, if_null)` 用于条件判断—常见的 Oracle 模式
- 空字符串是 NULL—`LENGTH('')` 返回 NULL，不是 0
- `NULLIF(a, b)` 如果相等返回 NULL—用于避免除以零

## 日期

- `SYSDATE` 用于当前日期时间—无括号
- `TO_DATE('2024-01-15', 'YYYY-MM-DD')` 用于字符串转日期—需要格式
- `TO_CHAR(date, 'YYYY-MM-DD HH24:MI:SS')` 用于日期转字符串
- 日期算术以天为单位—`SYSDATE + 1` 是明天，`SYSDATE + 1/24` 是一小时后

## 序列

- 创建：`CREATE SEQUENCE seq_name START WITH 1 INCREMENT BY 1`
- 获取下一个：`seq_name.NEXTVAL`—`SELECT seq_name.NEXTVAL FROM dual`
- 当前值：`seq_name.CURRVAL`—仅在同一会话中 NEXTVAL 之后
- 12c+：identity 列—`GENERATED ALWAYS AS IDENTITY`

## 层次查询

- `CONNECT BY PRIOR child = parent` 用于树遍历
- `START WITH parent IS NULL` 用于根节点
- `LEVEL` 伪列显示深度—`WHERE LEVEL <= 3` 限制深度
- `SYS_CONNECT_BY_PATH(col, '/')` 构建路径字符串

## 绑定变量

- 始终使用绑定变量—字面量每次都导致硬解析
- PL/SQL：`:variable_name` 语法
- 性能关键—字面量值填满 shared pool，导致争用
- `CURSOR_SHARING=FORCE` 作为变通方法但不建议长期使用

## Hints

- `/*+ INDEX(table idx_name) */` 强制使用索引
- `/*+ FULL(table) */` 强制全表扫描
- `/*+ PARALLEL(table, 4) */` 启用并行查询
- Hints 放在 `SELECT /*+ hint */` 内—通常放在 SELECT 关键字之后

## PL/SQL 块

- Anonymous block：`BEGIN ... END;` 以 `/` 在新行执行
- `DBMS_OUTPUT.PUT_LINE()` 用于调试输出—先执行 `SET SERVEROUTPUT ON`
- 异常处理：`EXCEPTION WHEN OTHERS THEN`—始终处理或记录
- `EXECUTE IMMEDIATE 'sql string'` 用于动态 SQL—注意注入

## 事务

- 默认无自动提交—必须显式 `COMMIT`
- `SAVEPOINT name` 然后 `ROLLBACK TO name` 用于部分回滚
- DDL 自动提交—`CREATE TABLE` 提交任何挂起的事务
- `SELECT FOR UPDATE WAIT 5` 等待 5 秒获取锁—避免无限挂起

## 性能

- `EXPLAIN PLAN FOR sql; SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY)`—显示计划
- `V$SQL` 和 `V$SESSION` 用于监控—需要权限
- 避免 `SELECT *`—获取所有列包括 LOBs
- 当优化器选择错误时使用索引 hint—`/*+ INDEX(t idx) */`

## 常见陷阱

- `MINUS` 替代 `EXCEPT`—Oracle 使用 MINUS 进行集合差集
- `DECODE` 是 Oracle 特有的—使用 CASE 以便移植
- 隐式类型转换—`WHERE num_col = '123'` 可以工作但阻止索引使用
- `ROWID` 是物理的—不要存储或在事务间依赖
