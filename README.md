# OpenClaw 框架生成器 Generator

## 简介

通用脚手架 Agent，在 OpenClaw 内部一键生成标准化项目框架。
全流程闭环，新建项目后可独立加载、迭代、运行。

**版本**：V3.3 — 6 种架构模式 + agents/ 目录 + 架构模式补充清单

---

## 快速开始

### 创建项目

```
/创建项目 【项目名】 【业务描述】
```

**示例**：
```
/创建项目 daily-notes 个人每日工作笔记与待办管理工具
```

### 带自定义规则

```
/创建项目 【项目名】 【业务描述】 【规则1,规则2,...】
```

**示例**：
```
/创建项目 data-pipeline 数据管道监控系统 数据底座:本地JSON文件,删除模式:硬删除
```

### 指定架构模式

```
/创建项目 【项目名】 【业务描述】 架构模式:【模式名】
```

**示例**：
```
/创建项目 workspace-mes 办公任务管理机器人 架构模式:hub-and-spoke,数据底座:飞书多维表格
```

### 辅助指令

| 指令 | 功能 |
|------|------|
| `/查看模板规范` | 展示标准目录结构、配置规则、模板说明 |
| `/查看已创建项目` | 列出已生成的项目目录 |
| `/查看模板` | 查看 templates/ 目录下的所有模板文件 |

---

## 6 种架构模式

### 模式选择规则

创建项目时，根据业务描述中的关键词自动匹配架构模式：

| 模式 | 触发关键词 | 适用场景 | 数据流 | 示例项目 |
|------|-----------|---------|--------|---------|
| **single-agent** | 默认（无匹配） | 简单 CRUD，单用户 | 直线型 | 笔记工具、日记 |
| **hub-and-spoke** | 管理/系统/调度/子Agent | 多任务编排 | 星型 | 任务管理系统、mes-agent |
| **pipeline** | 管道/流水线/处理链/多阶段 | 流水线处理 | 链型 | 数据处理管道 |
| **event-driven** | 监控/告警/事件/响应/触发 | 事件响应 | 异步 | 监控告警系统 |
| **daemon-agent** | 查询/查词/翻译/引擎/高频/搜索 | 高频查询，常驻进程 | IPC 通信 | 查词工具、EnglishPartner |
| **data-analysis** | 分析/报表/SQL/数据库/指标/归因 | 数据分析，多用户，复核流程 | SQL→复核→执行 | 经营分析系统、static-agent |

### 模式选择建议

| 你的项目类似 | 推荐模式 | 原因 |
|-------------|---------|------|
| 笔记/日记/简单 CRUD | single-agent | 1个Agent搞定 |
| 待办管理/飞书表格 | single-agent | 单Agent + 定时任务 |
| 记忆管理系统/多Agent协作 | hub-and-spoke | 1主+N子 |
| 查词/翻译/高频查询 | daemon-agent | 需要常驻进程 |
| 数据分析/SQL报表 | data-analysis | 多用户+复核流程 |
| 软件交付/开发全流程 | hub-and-spoke | PMO/PM/前后端/测试 |

---

## 生成的项目框架

### 目录结构

```
{项目名}/
├── agents/                  # Agent 配置目录
│   ├── main-agent/          # 主Agent 身份+技能（所有模式都有）
│   │   ├── IDENTITY.md      # 主Agent 身份
│   │   └── skill.yaml       # 主Agent 技能（含路由规则）
│   └── {子Agent}/           # 子Agent 身份+技能（hub-and-spoke 模式）
│       ├── IDENTITY.md
│       └── skill.yaml
├── datas/                   # 共享数据目录
│   ├── config.json          # 全局超参数（分层结构）
│   ├── config.schema.json   # 配置校验规则
│   └── logs/                # 运行日志
├── memory/                  # 工作记忆/运行记录
├── scripts/                 # 业务脚本（初始为空）
│   ├── daemon/              # 守护进程类（daemon-agent 模式）
│   ├── cli/                 # CLI 客户端类
│   ├── build/               # 数据构建类
│   ├── security/            # 安全加固类
│   └── utils/               # 工具函数类
├── AGENTS.md                # Agent 架构（含架构模式）
├── ARCHITECTURE.md          # 架构设计文档（数据流+状态机+契约）
├── BOOTSTRAP.md             # 启动引导（含就绪检查清单）
├── HEARTBEAT.md             # 心跳任务
├── IDENTITY.md              # 项目身份
├── README.md                # 项目说明
├── SCRIPTS.md               # 脚本架构（分类+编排+输出格式）
├── SOUL.md                  # 项目理念
├── TOOLS.md                 # 指令工具清单
└── USER.md                  # 用户配置
```

### 生成文件数

| 模式 | 文件数 | 说明 |
|------|:-----:|------|
| single-agent | ~15 | 1主Agent + 标准文件 |
| hub-and-spoke | ~17+N | N = 子Agent数 |
| pipeline | ~15 | 1主Agent + 标准文件 |
| event-driven | ~15 | 1主Agent + 标准文件 |
| daemon-agent | ~15 | 1主Agent + 标准文件 |
| data-analysis | ~15 | 1主Agent + 标准文件 |

---

## 生成后操作

### 第一步：查看补充清单

生成项目后，打开 `BOOTSTRAP.md`，会显示：

```
根据当前架构模式（xxx），请参考以下补充清单完成手动配置：
supplements/xxx.md
```

### 第二步：按优先级完成

每个补充清单分三个优先级：

| 优先级 | 含义 | 示例 |
|:-----:|------|------|
| 🔴 高优先级 | 必须做，否则项目无法运行 | 填写主Agent身份、配置数据底座 |
| 🟡 中优先级 | 建议做，否则功能不完整 | 定义输出格式、注册cron任务 |
| 🟢 低优先级 | 按需做 | 编写业务脚本、配置心跳 |

### 第三步：针对复杂业务

如果业务逻辑复杂（多步流程、条件分支、数据校验），参考 `ARCHITECTURE.md` 第7节 **flow/skill 分层架构**，将确定性逻辑从 AI 中剥离到 Python 代码中。

---

## 真实项目 Prompt 参考

### EnglishPartner（查词工具）

```
/创建项目 english-partner 英语学习查词工具，支持查词、词根、发音、语法分析 架构模式:daemon-agent,数据底座:本地JSON文件
```

**生成后重点补充**：守护进程实现、CLI客户端、数据构建管道

### records-agent（待办管理）

```
/创建项目 records-agent Eric的待办管理助手，从飞书群消息提取待办写入多维表格，每日定时推送提醒和回顾 架构模式:single-agent,数据底座:飞书多维表格
```

**生成后重点补充**：skill.yaml 业务逻辑、cron 任务注册

### mes-agent（任务编排）

```
/创建项目 workspace-mes 办公任务管理机器人，对接飞书多维表格，支持新增待办、查询提醒、21点自动日报 架构模式:hub-and-spoke,数据底座:飞书多维表格
```

**生成后重点补充**：flow/skill 分层架构、intent 意图定义、agents/ 子Agent配置

### static-agent（数据分析）

```
/创建项目 workspace-static 北极星指标数据分析项目，基于MySQL和SQL模板，支持GMV异动分析、维度拆解、归因分析、报告生成，多用户角色 架构模式:data-analysis,数据底座:MySQL
```

**生成后重点补充**：SQL 模板目录、semantic.yaml 语义层、多用户角色定义、复核流程

### workspace-mvp-iteration（软件交付）

```
/创建项目 workspace-mvp-iteration 软件产品MVP迭代开发工作区，支持需求拆解、PRD撰写、前后端开发、测试、CI/CD部署全流程，5个Agent协作（PMO/PM/后端/前端/测试） 架构模式:hub-and-spoke,数据底座:本地代码仓库
```

**生成后重点补充**：agents/ 5个子Agent配置、backend/frontend 源码目录、CI/CD 流水线

---

## 模板文件说明

### 模板目录结构

```
templates/
├── AGENTS.md               # Agent 架构模板（含架构模式）
├── SOUL.md                 # 项目理念模板
├── TOOLS.md                # 指令工具模板（含调用链路）
├── USER.md                 # 用户配置模板
├── IDENTITY.md             # Agent 身份模板
├── README.md               # 项目说明模板
├── BOOTSTRAP.md            # 启动引导模板（含就绪检查清单）
├── HEARTBEAT.md            # 心跳任务模板
├── ARCHITECTURE.md         # 架构设计模板（数据流+状态机+契约）
├── SCRIPTS.md              # 脚本架构模板（分类+编排+输出格式）
├── agents/
│   ├── main-agent/
│   │   ├── IDENTITY.md     # 主Agent 身份模板
│   │   └── skill.yaml      # 主Agent 技能模板（含路由规则）
│   └── sub-agent/
│       ├── IDENTITY.md     # 子Agent 身份模板
│       └── skill.yaml      # 子Agent 技能模板
├── supplements/
│   ├── single-agent.md     # single-agent 补充清单
│   ├── hub-and-spoke.md    # hub-and-spoke 补充清单
│   ├── pipeline.md         # pipeline 补充清单
│   ├── event-driven.md     # event-driven 补充清单
│   ├── daemon-agent.md     # daemon-agent 补充清单
│   └── data-analysis.md    # data-analysis 补充清单
├── config.json             # 全局超参数模板（分层结构）
└── config.schema.json      # 配置校验规则模板
```

### 模板占位符

| 占位符 | 说明 | 示例值 |
|--------|------|--------|
| `{{项目名}}` | 项目英文名 | `daily-notes` |
| `{{业务描述}}` | 用户提供的业务简介 | `个人每日工作笔记与待办管理工具` |
| `{{架构模式}}` | 自动选择的架构模式 | `single-agent` |
| `{{架构模式说明}}` | 对应模式的说明文本 | `单Agent，直线型数据流` |
| `{{核心理念}}` | 项目核心理念 | `每日记录，不遗漏任何工作事项` |
| `{{身份定位}}` | Agent 身份定位 | `个人每日工作笔记与待办管理助手` |
| `{{数据底座}}` | 数据存储方式 | `飞书多维表格` / `本地JSON文件` |
| `{{执行模式}}` | 脚本执行模式 | `one-shot` / `daemon` |
| `{{多用户模式}}` | 是否多用户 | `false` / `true` |
| `{{复核流程}}` | 是否有复核流程 | `false` / `true` |

---

## 注意事项

### 项目名规则

| ✅ 正确 | ❌ 错误 |
|---------|---------|
| `daily-notes` | `每日笔记`（含中文） |
| `task-manager` | `Task Manager`（含大写+空格） |
| `data_pipeline` | `数据管道` |

> 项目名必须是**英文小写**，用 `-` 或 `_` 连接。

### 业务描述原则

```
✅ 好的描述：个人每日工作笔记与待办管理工具
   → 清晰，能提炼核心理念、身份定位

❌ 差的描述：一个工具
   → 太模糊，无法生成有意义的内容
```

> 业务描述建议 **一句话**（10-30字），说明「谁用 + 做什么」。

### 自定义规则格式

```
数据底座:飞书多维表格,删除模式:软删除,节点标记:自动,架构模式:hub-and-spoke
```

- 用 `,` 分隔多个规则
- 规则名和值用 `:` 分隔
- 支持规则：`数据底座`、`删除模式`、`节点标记`、`架构模式`

### 覆盖保护

- 目录已存在则提示，**不会覆盖**原有文件
- 模板文件本身可编辑扩展，不视为"已有项目文件"

---

## 版本历史

| 版本 | 新增内容 |
|:----:|---------|
| V1.0 | 基础目录创建 + 6个根文件 |
| V2.0 | templates/ 目录，通用模板骨架 |
| V3.0 | 4种架构模式 + ARCHITECTURE.md + config.schema.json |
| V3.1 | daemon-agent 模式 + SCRIPTS.md |
| V3.2 | data-analysis 模式 + supplements/ 补充清单 |
| V3.3 | agents/ 目录（主Agent + 子Agent） |
