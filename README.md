# 🚀 基于deepagents的Deep Research 智能体示例

## 介绍
根据用户需求，生成深度研究报告。使用了deepseek chat 模型。

## 🚀 快速开始

**前置条件**：安装 [uv](https://docs.astral.sh/uv/) 包管理器：

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

请确保位于 `deep_research` 目录下：

```bash
cd deep_research
```

安装依赖：

```bash
uv sync
```

设置 deepseek 的 API 密钥，以及用于搜索的 tavily API 密钥：

```bash
export DEEPSEEK_API_KEY=your_anthropic_api_key_here  # Claude 模型所需
export TAVILY_API_KEY=your_tavily_api_key_here        # 用于网页搜索所需（[在此获取](https://www.tavily.com/)），并提供慷慨的免费套餐
```

## 用法

### 使用LangGraph Server 启动

用带 Web 界面的方式在本地启动一个 [LangGraph server](https://langchain-ai.github.io/langgraph/tutorials/langgraph-platform/local-server/)：

```bash
langgraph dev
```

LangGraph server 将在新浏览器窗口中打开 Studio 界面，你可以在其中提交搜索查询：

<img width="2869" height="1512" alt="截图 2025-11-17 于 11 42 59 上午" src="https://github.com/user-attachments/assets/03090057-c199-42fe-a0f7-769704c2124b" />

你也可以把 LangGraph server 连接到一个专为 [deepagents 设计的 UI](https://github.com/langchain-ai/deep-agents-ui)：

```bash
git clone https://github.com/langchain-ai/deep-agents-ui.git
cd deep-agents-ui
yarn install
yarn dev
```

然后按照 [deep-agents-ui README](https://github.com/langchain-ai/deep-agents-ui?tab=readme-ov-file#connecting-to-a-langgraph-server) 中的说明，把 UI 连接到正在运行的 LangGraph server。

该方式提供了用户友好的聊天界面，并可视化 state 中的文件内容。

<img width="2039" height="1495" alt="截图 2025-11-17 于 1 11 27 下午" src="https://github.com/user-attachments/assets/d559876b-4c90-46fb-8e70-c16c93793fa8" />

## 📚 资源


### 自定义模型

本示例中，`deepagents` 使用 `"deepseek-chat"`。你可以通过传入任意 [LangChain 模型对象](https://python.langchain.com/docs/integrations/chat/) 来进行自定义。


### 自定义指令

深度研究智能体会使用在 `research_agent/prompts.py` 中定义的自定义指令：它用于补充默认中间件指令（而不是重复）。你可以按自己的需求对这些指令进行任意修改。

| 指令集 | 用途 |
|----------------|---------|
| `RESEARCH_WORKFLOW_INSTRUCTIONS` | 定义 5 步研究工作流：保存请求 → 使用 TODO 进行规划 → 委派给子智能体 → 整合 → 响应。包含研究专属的规划准则，例如将相似任务合并批处理，以及针对不同查询类型的扩展规则。 |
| `SUBAGENT_DELEGATION_INSTRUCTIONS` | 通过示例提供具体的委派策略：简单查询使用 1 个子智能体，对比使用每个要素 1 个子智能体，多方面研究按每个方面 1 个子智能体。并设置并行执行的限制（最多 3 个并发）以及迭代轮数限制（最多 3 轮）。 |
| `RESEARCHER_INSTRUCTIONS` | 指导单个研究子智能体开展聚焦的网页搜索。包含硬性限制（简单查询 2-3 次搜索，复杂查询最多 5 次），强调每次搜索后使用 `think_tool` 进行策略反思，并定义停止条件。 |

### 自定义工具

深度研究智能体会在内置 deepagent 工具之外新增以下自定义工具。你也可以使用自己的工具（包括通过 MCP 服务器）。更多细节请参阅 Deep Agents 包的 [README](https://github.com/langchain-ai/deepagents?tab=readme-ov-file#mcp)。

| 工具名称 | 描述 |
|-----------|-------------|
| `tavily_search` | 网页搜索工具，仅将 Tavily 用作 URL 发现引擎。通过 Tavily API 搜索以找到相关 URL，使用 HTTP 抓取完整网页内容（并带上合适的 User-Agent 头，避免 403 错误），将 HTML 转换为 markdown，并在不做摘要的情况下返回完整内容，以保留所有信息供智能体分析。支持 Claude 与 Gemini 模型。 |
| `think_tool` | 战略反思机制，帮助智能体在两次搜索之间暂停并评估进展，分析已获得的发现，识别信息缺口，并规划下一步。 |
