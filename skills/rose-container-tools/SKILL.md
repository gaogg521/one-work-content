---
name: rose-container-tools
version: 0.1.0
description: 在 Docker 容器中构建和运行 ROSE 编译器工具。适用于开发源代码到源代码(source-to-source)翻译器、调用图(callgraph)分析器、AST 遍历(AST traversal)处理器及链接 librose.so 的工具。触发词：ROSE tool、callgraph、AST traversal、source-to-source、librose、编译器工具
tags:
- Docker
---

# ROSE 容器工具

使用安装在容器中的 ROSE 构建和运行基于 ROSE 的源代码分析工具。

## ⚠️ 始终使用 Makefile

**永远不要使用临时脚本或命令行编译 ROSE 工具。**

- 所有构建使用 `Makefile`
- 启用 `make -j` 并行
- 确保一致的 flags
- 支持 `make check` 进行测试

## 为什么使用容器？

ROSE 需要 GCC 7-10 和特定的 Boost 版本。大多数现代主机没有这些。容器提供：
- 预装在 `/rose/install` 的 ROSE
- 正确的编译器工具链
- 所有依赖已配置

## 快速开始

### 1. 启动容器

```bash
# 如果容器已存在
docker start rose-tools-dev
docker exec -it rose-tools-dev bash

# 或创建新容器
docker run -it --name rose-tools-dev \
  -v /home/liao/rose-install:/rose/install:ro \
  -v $(pwd):/work \
  -w /work \
  rose-dev:latest bash
```

### 2. 使用 Makefile 构建

**始终使用 Makefile 构建 ROSE 工具。永远不要使用临时脚本。**

```bash
# 在容器内部
make        # 构建所有工具
make check  # 构建并测试
```

### 3. 运行工具

```bash
./build/my_tool -c input.c
```

## Makefile（必需）

为你的工具创建 `Makefile`：

```makefile
ROSE_INSTALL = /rose/install

CXX      = g++
CXXFLAGS = -std=c++14 -Wall -g -I$(ROSE_INSTALL)/include/rose
LDFLAGS  = -L$(ROSE_INSTALL)/lib -Wl,-rpath,$(ROSE_INSTALL)/lib
LIBS     = -lrose

BUILDDIR = build
SOURCES  = $(wildcard tools/*.cpp)
TOOLS    = $(patsubst tools/%.cpp,$(BUILDDIR)/%,$(SOURCES))

.PHONY: all clean check

all: $(TOOLS)

$(BUILDDIR)/%: tools/%.cpp
	@mkdir -p $(BUILDDIR)
	$(CXX) $(CXXFLAGS) $< -o $@ $(LDFLAGS) $(LIBS)

check: all
	@for tool in $(TOOLS); do \
		echo "Testing $$tool..."; \
		LD_LIBRARY_PATH=$(ROSE_INSTALL)/lib $$tool -c tests/hello.c; \
	done

clean:
	rm -rf $(BUILDDIR)
```

## 示例：Identity Translator

最小的 ROSE 工具，解析并反解析代码：

```cpp
// tools/identity.cpp
#include "rose.h"

int main(int argc, char* argv[]) {
    SgProject* project = frontend(argc, argv);
    if (!project) return 1;
    
    AstTests::runAllTests(project);
    return backend(project);
}
```

构建并运行：
```bash
make
./build/identity -c tests/hello.c
# 输出: rose_hello.c (反解析)
```

## 示例：Call Graph Generator

```cpp
// tools/callgraph.cpp
#include "rose.h"
#include <CallGraph.h>

int main(int argc, char* argv[]) {
    ROSE_INITIALIZE;
    SgProject* project = new SgProject(argc, argv);
    
    CallGraphBuilder builder(project);
    builder.buildCallGraph();
    
    AstDOTGeneration dotgen;
    dotgen.writeIncidenceGraphToDOTFile(
        builder.getGraph(), "callgraph.dot");
    
    return 0;
}
```

## 示例：AST Node Counter

```cpp
// tools/ast_stats.cpp
#include "rose.h"
#include <map>

class NodeCounter : public AstSimpleProcessing {
public:
    std::map<std::string, int> counts;
    
    void visit(SgNode* node) override {
        if (node) counts[node->class_name()]++;
    }
};

int main(int argc, char* argv[]) {
    SgProject* project = frontend(argc, argv);
    
    NodeCounter counter;
    counter.traverseInputFiles(project, preorder);
    
    for (auto& [name, count] : counter.counts)
        std::cout << name << ": " << count << "\n";
    
    return 0;
}
```

## 常见 ROSE 头文件

| Header | 用途 |
|--------|---------|
| `rose.h` | 主头文件（包含大部分内容） |
| `CallGraph.h` | 调用图构建 |
| `AstDOTGeneration.h` | AST/图的 DOT 输出 |
| `sageInterface.h` | AST 操作工具 |

## AST 遍历模式

### 简单遍历（preorder/postorder）
```cpp
class MyTraversal : public AstSimpleProcessing {
    void visit(SgNode* node) override {
        // 处理每个节点
    }
};

MyTraversal t;
t.traverseInputFiles(project, preorder);
```

### 自顶向下传递继承属性
```cpp
class MyTraversal : public AstTopDownProcessing<int> {
    int evaluateInheritedAttribute(SgNode* node, int depth) override {
        return depth + 1;  // 传递给子节点
    }
};
```

### 自底向上传递综合属性
```cpp
class MyTraversal : public AstBottomUpProcessing<int> {
    int evaluateSynthesizedAttribute(SgNode* node, 
        SynthesizedAttributesList childAttrs) override {
        int sum = 0;
        for (auto& attr : childAttrs) sum += attr;
        return sum + 1;  // 返回给父节点
    }
};
```

## 在容器中测试

```bash
# 从宿主机运行
docker exec -w /work rose-tools-dev make check

# 或交互式
docker exec -it rose-tools-dev bash
cd /work
make && make check
```

## 故障排除

### "rose.h not found"
```bash
# 检查 include 路径
echo $ROSE/include/rose
ls $ROSE/include/rose/rose.h
```

### "cannot find -lrose"
```bash
# 检查库路径
ls $ROSE/lib/librose.so
```

### 运行时："librose.so not found"
```bash
# 设置库路径
export LD_LIBRARY_PATH=$ROSE/lib:$LD_LIBRARY_PATH
```

### 大文件段错误
```bash
# 增加栈大小
ulimit -s unlimited
```

## 容器参考

| 路径 | 内容 |
|------|----------|
| `/rose/install` | ROSE 安装（头文件、库、二进制文件） |
| `/rose/install/include/rose` | 头文件 |
| `/rose/install/lib` | librose.so 和依赖 |
| `/rose/install/bin` | ROSE 工具（identityTranslator 等） |
| `/work` | 挂载的工作区（你的代码） |
