# Picode

Picode 是一个本地运行的 AI Agent 工作台，提供 CLI、TUI 和常驻 Core 守护进程。项目保留了 Agent Loop、工具调用、事件流、权限审批、会话记忆、上下文压缩、Skills、MCP 和 Subagent 等基础能力，便于在此基础上开发自己的智能编程与自动化功能。

## 项目定位

Picode 不只是一次大模型 API 调用，而是一套可扩展的本地 Agent Runtime：

```text
用户目标
  → picode CLI / picode TUI
  → JSON-RPC 2.0 over NDJSON
  → picode-core daemon
  → AgentRunner / AgentLoop
  → LLM Provider
  → ToolRegistry / PermissionManager
  → EventBus / Session Store / Trace
```

核心特性：

- ReAct Agent Loop：模型可以规划、调用工具并根据结果继续执行
- 工具注册与参数校验：统一管理内置工具和自定义工具
- 权限审批：对文件修改、命令执行等敏感操作进行控制
- 事件流：实时展示模型、工具、任务和会话事件
- 多进程架构：Core 负责执行，CLI/TUI 负责交互
- Session、Notes 与 Context：支持多轮会话和项目级上下文
- 上下文治理：支持工具结果截断和自动压缩
- Skills、MCP 与 Subagent：支持后续扩展生态

## 快速开始

项目要求 Python 3.12。

```bash
uv sync
cp .env.example .env
```

启动 Core 守护进程：

```bash
uv run picode-core
```

另开终端执行健康检查：

```bash
uv run picode ping
uv run picode --version
```

执行一次 Agent 任务：

```bash
uv run picode run --goal "分析当前项目并给出改进建议"
```

启动终端界面：

```bash
uv run picode-tui
```

也可以通过 CLI 管理 Core：

```bash
uv run picode core start
uv run picode core status
uv run picode core stop
```

## 配置

配置优先级为：内置默认值 → `~/.picode/config.toml` → 项目 `.picode/config.toml` → `.env` → 系统环境变量。

环境变量统一使用 `PICODE_` 前缀，例如：

```env
PICODE_HOST=127.0.0.1
PICODE_PORT=7437
PICODE_LOG_LEVEL=INFO
PICODE_LOG_FILE=~/.picode/logs/core.log
PICODE_LLM_DEFAULT_MODEL=claude-sonnet-4-6
PICODE_MAX_STEPS=20
```

项目级扩展文件放在 `.picode/`，用户级配置放在 `~/.picode/`，包括：

```text
.picode/context.md
.picode/skills/
.picode/agents/
.picode/config.toml
~/.picode/sessions/
~/.picode/policy.toml
~/.picode/traces/
```

## 开发命令

```bash
uv run ruff check src tests scripts
uv run mypy src
uv run pytest tests/unit -v
uv run pytest tests/integration -v
uv run pytest tests/ -v
```

修改 IPC 命令或事件模型后，重新生成协议文档：

```bash
uv run python scripts/gen_protocol_doc.py
uv run python scripts/gen_protocol_doc.py --check
```

## 代码结构

```text
src/picode/
├── cli/          CLI 命令和客户端入口
├── core/
│   ├── agents/   Agent Profile
│   ├── bus/      IPC 命令、事件和 JSON-RPC 模型
│   ├── compact/  上下文压缩
│   ├── llm/      LLM Provider
│   ├── mcp/      MCP 客户端和工具
│   ├── memory/   上下文加载
│   ├── session/  会话和消息存储
│   ├── skills/   Skill 加载器
│   ├── subagent/ 子 Agent
│   ├── tools/    工具基类、注册表和内置工具
│   ├── trace/    调用追踪
│   └── transport/ IPC 网络传输
└── tui/          Textual 终端界面
```

新增工具时，建议继承 `BaseTool`，在 `AgentRunner` 中注册，并同步补充单元测试和事件展示。不要把具体业务逻辑直接堆进 `AgentLoop`。

## 项目状态

Picode 当前是一个持续演进的个人项目。后续功能会在独立分支中开发，并通过测试验证后合并。

## 许可证

本项目基于 MIT License。原始项目的版权和许可证声明保留在 [LICENSE](LICENSE) 文件中。
