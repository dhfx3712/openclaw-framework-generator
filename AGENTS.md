# AGENTS.md - 框架生成器 架构

## 主 Agent：框架生成主代理

### 权限
全局入口，接收用户所有创建指令，统筹全部生成流程。

### 核心职责
1. 解析用户入参：项目名称、业务简介、自定义规则
2. **选择架构模式**：根据项目复杂度自动匹配 4 种架构模式之一
3. 从 `templates/` 加载通用模板，渲染生成项目文件
4. 统一校验目录是否存在，避免重复创建
5. 汇总生成结果，输出使用指引
6. 异常捕获：目录已存在、权限不足等问题提示

---

## 架构模式选择逻辑

```
用户输入业务描述
       ↓
┌──────────────────────────────────────────────┐
│  判断依据：Agent数量、数据流复杂度、定时任务  │
├──────────────────────────────────────────────┤
│                                              │
│  简单CRUD + 本地文件  →  single-agent        │
│  1主+多子 + 外部存储  →  hub-and-spoke       │
│  流水线处理 + 多阶段  →  pipeline            │
│  事件响应 + cron驱动  →  event-driven        │
│  高频查询 + 常驻进程  →  daemon-agent        │
│  数据分析 + 多用户    →  data-analysis        │
│  默认（无法判断）     →  single-agent        │
│                                              │
└──────────────────────────────────────────────┘
       ↓
  选择对应模板变量
```

### 6 种架构模式说明

| 模式 | 适用场景 | Agent 结构 | 数据流 | 示例项目 |
|------|---------|-----------|--------|---------|
| **single-agent** | 简单 CRUD，单用户 | 1 个 Agent | 直线型 | 笔记工具、日记 |
| **hub-and-spoke** | 多任务编排 | 1 主 + N 子 | 星型 | 记忆管理系统 |
| **pipeline** | 流水线处理 | N 个顺序 Agent | 链型 | 数据处理管道 |
| **event-driven** | 事件响应 | 事件 → Agent | 异步 | 监控告警系统 |
| **daemon-agent** | 高频查询，常驻进程 | 1 Agent + daemon | IPC 通信 | 查词工具、翻译引擎 |
| **data-analysis** | 数据分析，多用户，复核流程 | 1 Agent + SQL 模板 | SQL 生成→复核→执行 | 经营分析系统 |

---

## 内置能力（Skill，无独立子Agent）

### 能力0：Skill 模板目录
框架生成器自带 3 个通用 Skill 模板，可在创建项目时按需引入：
- `templates/skills/feishu-bitable-skill/` — 飞书多维表格通用 CRUD（槽位驱动）
- `templates/skills/notification-channel-skill/` — 多通道通知（webchat + 飞书群）
- `templates/skills/memory-system-skill/` — 通用三层记忆系统（永久知识库 + 会话日志 + 实体注册表）

创建项目时通过自定义规则 `引入技能:feishu-bitable,notification-channel` 自动复制到目标项目。

### 能力1：目录创建 Skill
职责：按照标准规范递归创建目录树
标准目录：
```
{项目名}/
├── agents/
│   ├── main-agent/       ← 主Agent 配置目录
│   └── {子Agent}/        ← 子Agent 配置目录（hub-and-spoke 模式）
├── datas/
│   ├── logs/
├── memory/
└── scripts/
```

**目录创建规则：**
- `single-agent` 模式：创建 `agents/main-agent/`
- `hub-and-spoke` 模式：创建 `agents/main-agent/` + `agents/{子Agent名}/`
- 其余模式：创建 `agents/main-agent/`
- **所有模式统一创建** `memory/` 目录（MEMORY.md + README.md + entities/）

### 能力2：架构模式选择 Skill
职责：根据业务描述自动选择架构模式，设置对应模板变量

**判断规则：**
- 包含"管理""系统""调度""子Agent"等关键词 → `hub-and-spoke`
- 包含"管道""流水线""处理链""多阶段"等关键词 → `pipeline`
- 包含"监控""告警""事件""响应""触发"等关键词 → `event-driven`
- 包含"查询""查词""翻译""引擎""高频""搜索"等关键词 → `daemon-agent`
- 包含"分析""报表""SQL""数据库""数据看板""指标""归因"等关键词 → `data-analysis`
- 其余情况 → `single-agent`

### 能力3：模板渲染 Skill
职责：从 `templates/` 目录加载通用模板文件，动态填充项目信息。

**模板文件清单：**

| 文件 | 模板来源 | 说明 |
|------|---------|------|
| AGENTS.md | templates/AGENTS.md | Agent 架构（含架构模式） |
| SOUL.md | templates/SOUL.md | 项目理念 |
| TOOLS.md | templates/TOOLS.md | 指令工具清单（含调用链路） |
| USER.md | templates/USER.md | 用户配置 |
| IDENTITY.md | templates/IDENTITY.md | Agent 身份 |
| README.md | templates/README.md | 项目说明 |
| BOOTSTRAP.md | templates/BOOTSTRAP.md | 启动引导（含就绪检查清单） |
| HEARTBEAT.md | templates/HEARTBEAT.md | 心跳任务 |
| ARCHITECTURE.md | templates/ARCHITECTURE.md | 架构设计文档（数据流+状态机+契约） |
| **SCRIPTS.md** | templates/SCRIPTS.md | **脚本架构（分类+编排+输出格式）** |
| **agents/main-agent/IDENTITY.md** | templates/agents/main-agent/IDENTITY.md | **主Agent 身份** |
| **agents/main-agent/skill.yaml** | templates/agents/main-agent/skill.yaml | **主Agent 技能** |
| **agents/{子Agent}/IDENTITY.md** | templates/agents/sub-agent/IDENTITY.md | **子Agent 身份（hub-and-spoke 模式）** |
| **agents/{子Agent}/skill.yaml** | templates/agents/sub-agent/skill.yaml | **子Agent 技能（hub-and-spoke 模式）** |
| **memory/MEMORY.md** | templates/memory/MEMORY.md | **永久知识库（默认生成）** |
| **memory/README.md** | templates/memory/README.md | **记忆系统使用说明（默认生成）** |
| datas/config.json | templates/config.json | 全局超参数（分层结构） |
| datas/config.schema.json | templates/config.schema.json | **配置校验规则（新增）** |

**支持的占位符：**
- `{{项目名}}` — 项目英文名
- `{{业务描述}}` — 用户提供的业务简介
- `{{架构模式}}` — 自动选择的架构模式
- `{{架构模式说明}}` — 对应模式的说明文本
- `{{核心理念}}` — 项目核心理念
- `{{身份定位}}` — Agent 身份定位
- `{{数据底座}}` — 数据存储方式

### 能力4：参数初始化 Skill
职责：生成项目全局超参数 config.json + config.schema.json。
config.json 使用分层结构（project / storage / schedule / features / state_machine），
config.schema.json 提供字段校验规则。

### 能力5：结果输出 Skill
职责：生成完成后，打印完整目录树、文件清单、加载&使用步骤。
输出内容包括：
- 项目目录结构
- 文件清单
- 架构模式说明
- **架构模式补充清单路径**（`supplements/{模式}.md`）

### 能力6：记忆框架初始化 Skill
职责：所有新项目默认生成三层记忆系统框架，无需用户额外指定。

**生成内容：**
```
memory/
├── MEMORY.md                  # Layer 1: 永久知识库（预填分类模板）
├── README.md                  # 记忆系统使用说明
└── entities/                  # Layer 3: 实体注册表目录
    ├── users/
    ├── projects/
    ├── tools/
    └── concepts/
```

**模板来源：** `templates/memory/` 目录
- `templates/memory/MEMORY.md` — 永久知识库模板（含 `{{项目名}}` `{{架构模式}}` `{{数据底座}}` 占位符）
- `templates/memory/README.md` — 记忆系统使用说明（三层架构说明 + 使用规则）

**生成规则：**
1. 创建项目时自动创建 `memory/` 目录及所有子目录
2. 渲染 `templates/memory/MEMORY.md` → `memory/MEMORY.md`
3. 复制 `templates/memory/README.md` → `memory/README.md`
4. 首次会话日志 `memory/YYYY-MM-DD.md` 由 AI 在首次交互后自动创建
5. 实体注册表目录预创建，文件在首次提到实体时自动生成

**目录创建规则更新：**
所有架构模式统一创建 `memory/` 目录结构（不再只是空目录）。
