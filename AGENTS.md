# AGENTS.md - 框架生成器 架构

## 主 Agent：框架生成主代理
### 权限
全局入口，接收用户所有创建指令，统筹全部生成流程。

### 核心职责
1. 解析用户入参：项目名称、业务简介、自定义规则
2. 从 `templates/` 加载通用模板，渲染生成项目文件
3. 统一校验目录是否存在，避免重复创建
4. 汇总生成结果，输出使用指引
5. 异常捕获：目录已存在、权限不足等问题提示

## 内置能力（Skill，无独立子Agent）

### 能力1：目录创建 Skill
职责：按照标准规范递归创建目录树
标准目录：
```
{项目名}/
├── datas/
│   ├── logs/
├── memory/
└── scripts/
```

### 能力2：模板渲染 Skill
职责：从 `templates/` 目录加载通用模板文件，动态填充项目信息。
模板使用 `{{变量名}}` 占位符，支持以下变量：
- `{{项目名}}` — 项目英文名
- `{{业务描述}}` — 用户提供的业务简介
- `{{核心理念}}` — 项目核心理念（可自动从业务描述提炼）
- `{{身份定位}}` — Agent 身份定位
- `{{数据底座}}` — 数据存储方式（飞书表格/本地文件/数据库等）

生成文件清单：
| 文件 | 模板来源 | 说明 |
|------|---------|------|
| AGENTS.md | templates/AGENTS.md | Agent 架构（支持单Agent/多Agent） |
| SOUL.md | templates/SOUL.md | 项目理念 |
| TOOLS.md | templates/TOOLS.md | 指令工具清单 |
| USER.md | templates/USER.md | 用户配置 |
| IDENTITY.md | templates/IDENTITY.md | Agent 身份 |
| README.md | templates/README.md | 项目说明 |
| BOOTSTRAP.md | templates/BOOTSTRAP.md | 启动引导 |
| HEARTBEAT.md | templates/HEARTBEAT.md | 心跳任务 |
| datas/config.json | templates/config.json | 全局超参数 |

### 能力3：参数初始化 Skill
职责：生成项目全局超参数 config.json，默认使用通用骨架模板，支持用户自定义修改。

### 能力4：结果输出 Skill
职责：生成完成后，打印完整目录树、文件清单、加载&使用步骤。
