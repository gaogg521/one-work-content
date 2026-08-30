---
name: java-development
description: Guides Java SDK development in Apache Beam, including building, testing, running 示例, and understanding the 项目 structure. Use when working with Java code in sdks/java/, runners/, or 示例/java/.
---

# Java Development in Apache Beam

## 项目 Structure

### 键 Directories
- `sdks/java/core` - Core Java SDK (PCollection, PTransform, 管道)
- `sdks/java/harness` - SDK harness (container entrypoint)
- `sdks/java/io/` - I/O connectors (51+ connectors including BigQuery, Kafka, JDBC, etc.)
- `sdks/java/extensions/` - Extensions (SQL, ML, protobuf, etc.)
- `runners/` - Runner implementations:
  - `runners/direct-java` - Direct Runner (local execution)
  - `runners/flink/` - Flink Runner
  - `runners/spark/` - Spark Runner
  - `runners/google-cloud-dataflow-java/` - Dataflow Runner
- `examples/java/` - Java 示例 including WordCount

### 构建 System
Apache Beam uses Gradle with a custom `BeamModulePlugin`. Every Java 项目's `构建.g`bu`构建.g`CODE_2__`with:
```groovy
apply plugin: 'org.apache.beam.module'
applyJavaNature( ... )
```

## Common 命令

### 构建 命令
```bash
# Compile a specific project
./gradlew -p sdks/java/core compileJava

# Build a project (compile + tests)
./gradlew :sdks:java:harness:build

# Run WordCount example
./gradlew :examples:java:wordCount
```

### Running 单位 Tests
```bash
# Run all tests in a project
./gradlew :sdks:java:harness:test

# Run a specific test class
./gradlew :sdks:java:harness:test --tests org.apache.beam.fn.harness.CachesTest

# Run tests matching a pattern
./gradlew :sdks:java:harness:test --tests *CachesTest

# Run a specific test method
./gradlew :sdks:java:harness:test --tests *CachesTest.testClearableCache
```

### Running Integration Tests
Integration tests have filenames ending in `IT.java` and u`TestPipeline````.

```bash
# Run I/O integration tests on Direct Runner
./gradlew :sdks:java:io:google-cloud-platform:integrationTest

# Run with custom GCP project
./gradlew :sdks:java:io:google-cloud-platform:integrationTest \
  -PgcpProject=<project> -PgcpTempRoot=gs://<bucket>/path

# Run on Dataflow Runner
./gradlew :runners:google-cloud-dataflow-java:examplesJavaRunnerV2IntegrationTest \
  -PdisableSpotlessCheck=true -PdisableCheckStyle=true -PskipCheckerFramework \
  -PgcpProject=<project> -PgcpRegion=us-central1 -PgcsTempRoot=gs://<bucket>/tmp
```

### Code Formatting
```bash
# Format Java code
./gradlew spotlessApply
```

## Writing Integration Tests

```java
@Rule public TestPipeline pipeline = TestPipeline.create();

@Test
public void testSomething() {
  pipeline.apply(...);
  pipeline.run().waitUntilFinish();
}
```

集合 管道 选项 via `-DbeamTestPipelineOptions='[...]'`:
```bash
-DbeamTestPipelineOptions='["--runner=TestDataflowRunner","--project=myproject","--region=us-central1","--stagingLocation=gs://bucket/path"]'
```

## Using Modified Beam Code

### Publish 迁移到 Maven Local
```bash
# Publish a specific module
./gradlew -Ppublishing -p sdks/java/io/kafka publishToMavenLocal

# Publish all modules
./gradlew -Ppublishing publishToMavenLocal
```

### Building SDK Container
```bash
# Build Java SDK container (for Runner v2)
./gradlew :sdks:java:container:java11:docker

# Tag and push
docker tag apache/beam_java11_sdk:2.XX.0.dev \
  "us-docker.pkg.dev/your-project/beam/beam_java11_sdk:custom"
docker push "us-docker.pkg.dev/your-project/beam/beam_java11_sdk:custom"
```

### Building Dataflow 工作者 Jar
```bash
./gradlew :runners:google-cloud-dataflow-java:worker:shadowJar
```

## 测试 Naming Conventions
- 单位 tests: `*Test.java`
- Integration tests: `*IT.java`

## JUnit Report Location
After running tests, 查找 HTML reports at:
`<project>/build/reports/tests/test/index.html`

## IDE 设置 (IntelliJ)
1. Open `/beam` (the 仓库 r`sdks/java`va``va`a`)
2. Wait for indexing 迁移到 完成
3. 查找 `examples/java/build.gradle` and click 运行 next 迁移到 wordCount 任务 迁移到 验证 设置