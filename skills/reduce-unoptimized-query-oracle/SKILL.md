---
name: reduce-unoptimized-query-oracle
description: 将 unoptimized-query-oracle 测试失败日志缩减为最简单的可能重现案例。当您有来自失败 roachtest 的 unoptimized-query-oracle*.log 文件并需要找到重现错误的最小 SQL 时使用。
disable-model-invocation: True
---

# 缩减未优化查询 Oracle 测试失败

将 unoptimized-query-oracle 测试失败日志缩减为最简单的可能重现案例。

unoptimized-query-oracle roachtest 运行一系列随机 SQL 语句来创建随机数据集，然后执行两次随机的"感兴趣查询"，使用不同的优化设置。如果两次执行返回不同的结果，则表明 CockroachDB 中存在错误。

## 何时使用

在以下情况使用此技能：
- 您有来自 unoptimized-query-oracle roachtest 的测试失败。
- 您需要找到重现测试失败的最小 SQL。

## 步骤 1：定位工件

**询问用户工件目录在哪里。**

在工件目录中找到相关文件：
- **测试参数**：`params.log`（来自 roachtest 的参数）
- **测试日志**：`test.log`（来自 roachtest 的日志）
- **失败日志**：`failure*.log`（来自 roachtest 的失败日志）
- **完整 SQL 日志**：`unoptimized-query-oracle*.log`（导致失败的 SQL 语句）
- **感兴趣查询日志**：`unoptimized-query-oracle*.failure.log`（包含
  感兴趣查询和可能有关失败的更多信息）
- **Cockroach 日志**：`logs/1.unredacted/cockroach.log` 或
  `logs/unredacted/cockroach.log`（包含 git 提交）

## 步骤 2：确定测试配置

从 `cockroach.log` 确定 git 提交：
```bash
grep "binary: CockroachDB" cockroach.log
```
在版本字符串中查找提交哈希（例如，`cb94db961b8f55e3473f279d98ae90f0eeb0adcb`）。

通过检查以下内容确定是否启用了运行时断言：
- `params.log` 中的 `"runtimeAssertionsBuild": "true"`
- 或 `test.log` 中的 `Runtime assertions enabled`

通过查找以下内容确定是否应用了变形设置：
- `params.log` 中的如下行：
  ```
  "metamorphicBufferedSender": "true",
  "metamorphicWriteBuffering": "true",
  ```
- 或 `test.log` 中的如下行：
  ```
  metamorphically setting "kv.rangefeed.buffered_sender.enabled" to 'true'
  metamorphically setting "kv.transaction.write_buffering.enabled" to 'true'
  ```

从 `cockroach.log` 开头确定环境变量：
```bash
grep -A10 "using local environment variables:" cockroach.log
```

重要的环境变量包括：
- `COCKROACH_INTERNAL_CHECK_CONSISTENCY_FATAL`
- `COCKROACH_INTERNAL_DISABLE_METAMORPHIC_TESTING`
- `COCKROACH_RANDOM_SEED`
- `COCKROACH_TESTING_FORCE_RELEASE_BRANCH`
但可能还有更多重要的环境变量，因此最好获取所有环境变量。

通过检查以下内容确定这是多区域测试还是单区域测试：
- 测试名称（例如，`test.log` 中的 `seed-multi-region` 表示多区域）
- 或完整 SQL 日志中是否存在 `\connect` 行
如果两者都缺失，则为单区域测试。

## 步骤 3：检出和构建

对于正常构建使用：
```bash
git checkout <commit-hash>
./dev build short
```

如果启用了运行时断言，则改用测试构建：
```bash
git checkout <commit-hash>
./dev build short -- --crdb_test
```

**注意：** 仅当重现使用地理空间函数（BOX2D、geometry、geography 等）时才构建 libgeos：
```bash
./dev build libgeos
```

## 步骤 4：准备完整 SQL 日志文件

首先，检查以下语句是否在完整 SQL 日志文件顶部。如果没有，请添加它们：
```sql
SET statement_timeout='1m0s';
SET sql_safe_updates = false;
```

如果使用了变形设置，也请将它们添加到完整 SQL 日志文件顶部：
```sql
SET CLUSTER SETTING kv.rangefeed.buffered_sender.enabled = true;
SET CLUSTER SETTING kv.transaction.write_buffering.enabled = true;
```

在工件目录中或仓库根目录中创建适当的目录用于存放临时文件。

## 步骤 5：初始重现

根据测试类型确定正确的 demo 命令：
- **多区域测试**：使用 `--nodes=9`
- **单区域测试**：省略 `--nodes` 选项

使用如下命令尝试从完整 SQL 日志文件重现测试失败。此命令可能需要长达 20 分钟才能完成。

```bash
<env vars> ./cockroach demo --multitenant=false --nodes=9 --insecure --set=errexit=false --no-example-database --format=tsv -f <full-sql-log-file>
```

**检查输出是否重现了失败日志中描述的测试失败。** 有许多可能的失败模式。查找以下之一，它们应与失败日志匹配：

1. 两次执行"感兴趣查询"之间的**不同结果**
   （这是在日志末尾附近、包装在各种 SET 和 RESET 语句中重复两次的随机生成 SELECT 语句）。这些不同的结果可能采取不同结果集的形式，也可能是一种情况有错误而另一种情况没有错误。这是**"oracle"失败**。
2. 或者，`internal error` 或**断言失败**。记下错误消息用于缩减步骤。
3. 或者，**panic**。记下错误消息用于缩减步骤。
4. 或者，**超时**。记下超时的语句。

### 故障排除

**重要：** 许多失败是非确定性的，特别是对于多区域测试。如果第一次运行没有发生失败，在断定它无法重现之前尝试最多 10 次。

此时将输出与 `failure*.log` 进行比较会很有帮助，后者应显示原始测试运行的失败。

**如果初始运行 10 次后仍无法重现，请在此处暂停并向用户报告无法重现失败，并显示尝试的命令。** 用户可能有额外的说明。

如果看起来可以重现，则继续下一步。

## 步骤 6：使用缩减工具

构建缩减工具：
```bash
./dev build reduce
```

### 再次准备完整 SQL 日志文件

对于多区域测试，删除 `\connect` 行（它们在 `reduce` 工具中会导致语法错误）：
```bash
grep -v '^\\connect' <full-sql-log-file> > <cleaned-log>
```

### 运行缩减

**重要：** 缩减工具必须从 cockroach 仓库根目录运行，因为它在当前目录中查找 `./cockroach`。

对多区域测试使用 `-multi-region` 选项，对单区域测试省略它。

**对于"oracle"失败（不同结果）：**
```bash
./bin/reduce -unoptimized-query-oracle -multi-region -chunk 25 -v -file <cleaned-log> 2>&1 | tee reduce-output.log
```
`-unoptimized-query-oracle` 选项检查两次执行"感兴趣查询"是否产生相同的结果。

**对于内部错误/断言失败/panic：**
```bash
./bin/reduce -contains "<error-regex>" -multi-region -chunk 25 -v -file <cleaned-log> 2>&1 | tee reduce-output.log
```
使用错误消息的 distinctive 部分作为 `-contains` 正则表达式（例如，`"nil LeafTxnInputState"`）。

缩减工具可能需要长达一小时才能运行。

### 提取缩减后的 SQL

缩减工具输出进度行，然后是最终 SQL。仅提取 SQL：
```bash
grep -A1000 "^reduction: " reduce-output.log | tail -n +2 > reduced.sql
```

**重要：** 在手动简化之前立即保存缩减输出的备份：
```bash
cp reduced.sql reduced_original.sql
```
这提供了在工作文件在简化过程中损坏时的恢复点。

**如果缩减工具无法重现，请在此处暂停并向用户报告。他们可能有额外的说明。** 偶尔我们必须修改缩减工具本身，如果测试失败无法重现。

## 步骤 7：创建测试脚本并确定重现率

**重要：** 许多错误是非确定性的。在手动简化之前，创建一个可重用的测试脚本并确定重现率。

创建一个小型测试脚本（根据需要调整）：
```bash
cat > test_repro.sh << 'EOF'
#!/bin/bash
# 测试 reduced_v2.sql 是否重现错误（首次成功时退出，最多 10 次尝试）
for i in {1..10}; do
  if ./cockroach demo --multitenant=false --nodes=9 --insecure \
     --set=errexit=false --no-example-database --format=tsv \
     -f reduced_v2.sql 2>&1 | grep -q "<error-pattern>"; then
    echo "Run $i: REPRODUCED"
    exit 0
  else
    echo "Run $i: no error"
  fi
done
echo "FAILED"
EOF
chmod +x test_repro.sh
```
对于"oracle"失败，测试脚本可能需要隔离和比较两次执行"感兴趣查询"的结果，而不是检查错误模式。

运行测试脚本以确定重现率。并不总是 100%。

此速率决定了测试简化时需要尝试的次数：
- 100% 率：单次尝试足够
- 50% 率：通常 2-3 次尝试足够
- 10% 率：需要约 10 次尝试才能确信
- <5% 率：可能需要 20+ 次尝试

请注意，在某些情况下，可能需要将以下设置添加回缩减文件以获得重现：
```sql
SET statement_timeout='1m0s';
SET sql_safe_updates = false;
```

**如果缩减后的 SQL 在 10 次尝试后仍无法重现，请在此处暂停并向用户报告。他们可能有额外的说明。**

## 步骤 8：手动简化

现在在保持重现的同时迭代简化 SQL。

**关键：** 对于非确定性失败，您必须根据重现率用足够的尝试次数测试每个简化。单次失败尝试并不意味着简化破坏了重现 —— 可能只是非确定性。

### 每个简化的工作流程

1. 将 `reduced.sql` 复制到 `reduced_v2.sql`
2. 对 `reduced_v2.sql` 进行**一处**小改动
3. 运行 `./test_repro.sh`（它测试 `reduced_v2.sql`）
4. 如果重现：将 `reduced_v2.sql` 复制到 `reduced.sql`，继续简化
5. 如果足够尝试次数后仍未重现：丢弃 `reduced_v2.sql`，尝试不同的更改（即回溯）。

此工作流程避免了需要恢复文件 —— 您始终在 `reduced.sql` 中保留最后一个工作版本。

**重要：** 将复制、编辑和测试作为单独的 bash 命令运行（不要用 `&&` 链接）。这减少了权限检查的数量。

### 尝试删除的内容（大致顺序）

1. **查询投影和聚合** - 将 SELECT 列表简化为仅基本列
2. **查询谓词** - 简化 WHERE 子句
3. **索引** - 尝试删除二级索引
4. **查询连接** - 简化 WHERE 子句
5. **CREATE TABLE 中的列** - 删除失败查询中未引用的列
6. **怪异字符** - 从名称和数据中删除或替换非 ASCII 字符
7. 其他 SQL 简化

**对于"oracle"失败，在编辑感兴趣查询时，请务必编辑**两次**感兴趣查询的副本，使它们完全相同。** 否则在比较结果集时就不会是同类比较。

### 常见必需元素

这些通常无法删除：
- **优化器随机种子**：`SET testing_optimizer_random_seed = <value>` - 这个特定值通常无法更改，因为它决定了禁用哪些优化器规则
- **优化器规则概率**：`SET testing_optimizer_disable_rule_probability` - 影响查询计划选择
- 优化器设置的特定 RESET/SET 序列，如 distsql 和 vectorize
- 某些索引（影响查询计划）
- 分布式查询错误的多节点设置（`--nodes=9`）（尽管首先尝试单节点 - 它可能有效且更简单）
- `CREATE STATISTICS` 语句（影响查询计划）

### 回溯

如果更改破坏了重现：
1. 丢弃 `reduced_v2.sql`（不要将其复制到 `reduced.sql`）
2. 验证 `reduced.sql` 仍然重现。如果没有，这意味着重现是非确定性的。（它可能一开始就是非确定性的，或者可能在简化过程中变得非确定性。）尝试重现它 10 次并记下新的重现率。使用新的重现率来调整每个简化步骤前进期间的重现尝试次数。
3. 尝试**不同**的简化

永远不要从损坏状态继续简化。

**如果您陷入困境（即在回溯几次后无法再次重现），请停止并使用您尝试的确切命令向用户报告。**

## 步骤 9：最终验证和输出

**在大约 20 分钟的简化后，或者如果在回溯几次后没有更多简化，就是停止的时候了。**

1. 运行重现 10+ 次以确认稳定性并确定最终重现率
2. 记录最小重现步骤
3. 记下哪些元素是必需的 vs 可选的

### 输出

最终输出应包含两个可以显示给用户的文件：

1. **reduced.sql** - 重现错误的最小 SQL 脚本
2. **bisect_run.sh** - 用于 `git bisect run` 的脚本

以可以复制粘贴到终端的方式编写输出。

### 示例输出格式

（此输出中的命令应编辑为匹配重现所需的内容。）

```bash
# 最小重现

# reduced.sql
cat > reduced.sql << 'EOF'
CREATE TABLE t ();

SET testing_optimizer_random_seed = 1234567890;
SET testing_optimizer_disable_rule_probability = 0.5;

SELECT ...;
EOF

# bisect_run.sh
cat > bisect_run.sh << 'EOF'
#!/bin/bash
# Git bisect run 脚本
# 退出代码：0=良好（不存在错误），1=错误（存在错误），125=跳过（构建失败）

REPO_DIR="/path/to/cockroach"
REPRO_SQL="/path/to/reduced.sql"

cd "$REPO_DIR" || exit 125

echo "=== 测试提交 $(git rev-parse --short HEAD) ==="

# 构建（如果原始测试中启用了运行时断言，则使用 --crdb_test）
if ! ./dev build short -- --crdb_test 2>&1 | grep -q "Successfully built"; then
    echo "构建失败 - 跳过"
    exit 125
fi

# 测试错误（对不稳定错误尝试 3 次）
for i in {1..3}; do
    if ./cockroach demo --multitenant=false --insecure \
        --set=errexit=false --no-example-database --format=tsv \
        -f "$REPRO_SQL" 2>&1 | grep -q "<error-pattern>"; then
        echo "存在错误 - 标记为错误"
        exit 1
    fi
done

echo "不存在错误 - 标记为良好"
exit 0
EOF
chmod +x bisect_run.sh

# 重现命令
git checkout <commit-hash>
./bisect_run.sh

# bisect 命令
git bisect start ...
git bisect run bisect_run.sh

# 失败
# <在此处粘贴堆栈跟踪或相关失败详情>

# 重现率：~X%（可能需要多次尝试）
```

**显示此输出后，询问用户是否想在 master 分支上尝试重现错误。**

## 可选步骤 10：检查错误是否在 Master 上已修复

在 bisect 之前，检查错误是否已在 master 上修复。

```bash
git stash  # 如果需要
git checkout master
./dev build short -- --crdb_test
./cockroach demo --multitenant=false --insecure --set=errexit=false --no-example-database --format=tsv -f reduced.sql
```

多次运行此命令以考虑不稳定性。记下错误是否在 master 上重现。

## 可选步骤 11：Bisect

如果用户想要找到引入或修复错误的提交，请使用 `git bisect`。

### 如果错误已在 Master 上修复

Bisect 以找到**修复提交**（错误不再重现的第一个提交）。使用自定义术语，因为"良好"提交（master）比"错误"提交更新：

```bash
git bisect start --first-parent --term-old=broken --term-new=fixed
git bisect broken <commit-where-bug-exists>   # 例如，原始失败提交
git bisect fixed master                        # master 已修复

git bisect run ./bisect_run.sh

# 完成后
git bisect reset
```

**注意：** `--first-parent` 选项仅跟随主分支上的合并提交，避免绕道进入功能分支。当错误不存在（已修复）时，bisect 脚本必须返回 0；当错误存在（损坏）时，返回 1。

### 如果错误在 Master 上仍然存在

Bisect 以找到**回归提交**（引入错误的第一个提交）：

```bash
git bisect start --first-parent
git bisect good <known-good-commit>   # 例如，之前的发布标签
git bisect bad master                  # master 有错误

git bisect run ./bisect_run.sh

# 完成后
git bisect reset
```

Bisect 将识别引入或修复错误的提交。

### 找到良好提交

如果您不知道良好提交（错误不存在的地方），您可以及时跳回以找到一个。

```bash
# 找到主分支上大约 6 个月前的提交
git rev-list --first-parent -1 --before="6 months ago" HEAD
```

测试该提交是否存在错误。如果不存在，请将其用作 bisect 的良好提交。如果错误仍然存在，请尝试及时跳回更远，但不要超过 1 年。

**如果在 1 年内找不到已知良好提交，请停止并向用户报告。**
