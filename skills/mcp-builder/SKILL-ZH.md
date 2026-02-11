---
name: mcp-builder
description: Guide for creating high-quality MCP (Model Context Protocol) servers that enable LLMs to interact with external services through well-designed tools. Use when building MCP servers to integrate external APIs or services, whether in Python (FastMCP) or Node/TypeScript (MCP SDK).
license: Complete terms in LICENSE.txt
---

# MCP服务器开发指南

## 概述

创建MCP（模型上下文协议）服务器，使LLM能够通过设计良好的工具与外部服务交互。MCP服务器的质量通过LLM完成现实世界任务的有效程度来衡量。

---

# 流程

## 🚀 高层次工作流程

创建高质量的MCP服务器涉及四个主要阶段：

### 第1阶段：深度研究和规划

#### 1.1 理解现代MCP设计

**API覆盖 vs 工作流工具**：在全面的API端点覆盖和专业化工作流工具之间取得平衡。工作流工具对于特定任务可能更方便，而全面覆盖让代理能够灵活组合操作。性能因客户端而异——一些客户端受益于结合基本工具的代码执行，而另一些则更适合高级别工作流。当不确定时，优先考虑全面的API覆盖。

**工具命名和可发现性**：清晰、描述性的工具名称帮助代理快速找到工具。使用一致的前缀（例如`github_create_issue`、`github_list_repos`）和动作导向的命名。

**上下文管理**：代理受益于简洁的工具描述和过滤/分页结果的能力。设计工具返回集中、相关的数据。一些客户端支持代码执行，可以帮助代理过滤和处理数据。

**可操作错误消息**：错误消息应该用具体建议和后续步骤指导代理朝向解决方案。

#### 1.2 研究MCP协议文档

**导航MCP规范**：

从站点地图开始：`https://modelcontextprotocol.io/sitemap.xml`

然后获取带.md后缀的具体页面以获取markdown格式（例如`https://modelcontextprotocol.io/specification/draft.md`）。

要审查的关键页面：
- 规范概述和架构
- 传输机制（可流式HTTP、stdio）
- 工具、资源和提示定义

#### 1.3 研究框架文档

**推荐的堆栈**：
- **语言**：TypeScript（高质量SDK支持和在许多执行环境例如MCPB中良好的兼容性。加上AI模型擅长生成TypeScript代码，受益于其广泛使用、静态类型和良好的linting工具）
- **传输**：用于远程服务器的可流式HTTP，使用无状态JSON（更简单可扩展和维护，与有状态会话和流式响应相反）。stdio用于本地服务器。

**加载框架文档**：

- **MCP最佳实践**：[📋 查看最佳实践](./reference/mcp_best_practices.md) - 核心指南

**对于TypeScript（推荐）**：
- **TypeScript SDK**：使用WebFetch加载`https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`
- [⚡ TypeScript指南](./reference/node_mcp_server.md) - TypeScript模式和示例

**对于Python**：
- **Python SDK**：使用WebFetch加载`https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`
- [🐍 Python指南](./reference/python_mcp_server.md) - Python模式和示例

#### 1.4 规划你的实现

**理解API**：
审查服务的API文档以识别关键端点、身份验证要求和数据模型。根据需要使用网络搜索和WebFetch。

**工具选择**：
优先全面的API覆盖。列出要实现的端点，从最常见的操作开始。

---

### 第2阶段：实现

#### 2.1 设置项目结构

参见语言特定指南以获取项目设置：
- [⚡ TypeScript指南](./reference/node_mcp_server.md) - 项目结构、package.json、tsconfig.json
- [🐍 Python指南](./reference/python_mcp_server.md) - 模块组织、依赖项

#### 2.2 实现核心基础设施

创建共享实用程序：
- 带身份验证的API客户端
- 错误处理辅助程序
- 响应格式（JSON/Markdown）
- 分页支持

#### 2.3 实现工具

对于每个工具：

**输入模式**：
- 使用Zod（TypeScript）或Pydantic（Python）
- 包括约束和清晰描述
- 在字段描述中添加示例

**输出模式**：
- 在可能的地方定义`outputSchema`
- 在工具响应中使用`structuredContent`（TypeScript SDK功能）
- 帮助客户端理解和处理工具输出

**工具描述**：
- 功能简洁总结
- 参数描述
- 返回类型模式

**实现**：
- 用于I/O操作的async/await
- 带有可操作消息的正确错误处理
- 在适用时支持分页
- 在使用现代SDK时返回文本内容和结构化数据

**注释**：
- `readOnlyHint`：true/false
- `destructiveHint`：true/false
- `idempotentHint`：true/false
- `openWorldHint`：true/false

---

### 第3阶段：审查和测试

#### 3.1 代码质量

审查：
- 没有重复代码（DRY原则）
- 一致的错误处理
- 完整类型覆盖
- 清晰的工具描述

#### 3.2 构建和测试

**TypeScript**：
- 运行`npm run build`以验证编译
- 使用MCP Inspector测试：`npx @modelcontextprotocol/inspector`

**Python**：
- 验证语法：`python -m py_compile your_server.py`
- 使用MCP Inspector测试

参见语言特定指南以获取详细测试方法和质量清单。

---

### 第4阶段：创建评估

实现MCP服务器后，创建全面测试其有效性的评估。

**加载[✅ 评估指南](./reference/evaluation.md)以获取完整的评估指南。**

#### 4.1 理解评估目的

使用评估来测试LLM是否能够有效使用你的MCP服务器回答现实的、复杂的问题。

#### 4.2 创建10个评估问题

要创建有效的评估，遵循评估指南中概述的过程：

1. **工具检查**：列出可用工具并理解其能力
2. **内容探索**：使用只读操作探索可用数据
3. **问题生成**：创建10个复杂的、现实的问题
4. **答案验证**：自己解决每个问题以验证答案

#### 4.3 评估要求

确保每个问题是：
- **独立**：不依赖于其他问题
- **只读**：只需要非破坏性操作
- **复杂**：需要多个工具调用和深度探索
- **基于现实**：基于人类会关心的真实用例
- **可验证**：单一、清晰的答案可以通过字符串比较验证
- **稳定**：答案不会随时间改变

#### 4.4 输出格式

创建具有此结构的XML文件：

```xml
<evaluation>
  <qa_pair>
    <question>Find discussions about AI model launches with animal codenames. One model needed a specific safety designation that uses the format ASL-X. What number X was being determined for the model named after a spotted wild cat?</question>
    <answer>3</answer>
  </qa_pair>
<!-- 更多qa_pairs... -->
</evaluation>
```

---

# 参考文件

## 📚 文档库

在开发过程中根据需要加载这些资源：

### 核心MCP文档（首先加载）

- **MCP协议**：从`https://modelcontextprotocol.io/sitemap.xml`的站点地图开始，然后获取带.md后缀的具体页面
- [📋 MCP最佳实践](./reference/mcp_best_practices.md) - 通用MCP指南包括：
  - 服务器和工具命名约定
  - 响应格式指南（JSON vs Markdown）
  - 分页最佳实践
  - 传输选择（可流式HTTP vs stdio）
  - 安全和错误处理标准

### SDK文档（在第1/2阶段加载）

- **Python SDK**：从`https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`获取
- **TypeScript SDK**：从`https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`获取

### 语言特定实现指南（在第2阶段加载）

- [🐍 Python实现指南](./reference/python_mcp_server.md) - 完整的Python/FastMCP指南包括：
  - 服务器初始化模式
  - Pydantic模型示例
  - 使用`@mcp.tool`的工​​具注册
  - 完整工作示例
  - 质量清单

- [⚡ TypeScript实现指南](./reference/node_mcp_server.md) - 完整的TypeScript指南包括：
  - 项目结构
  - Zod模式模式
  - 使用`server.registerTool`的工​​具注册
  - 完整工作示例
  - 质量清单

### 评估指南（在第4阶段加载）

- [✅ 评估指南](./reference/evaluation.md) - 完整的评估创建指南包括：
  - 问题创建指南
  - 答案验证策略
  - XML格式规范
  - 示例问题和答案
  - 使用提供的脚本运行评估
