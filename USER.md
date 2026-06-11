# USER.md - 框架生成器 用户配置

## 基础偏好
- 运行环境：OpenClaw 内置运行，Linux/Mac 兼容
- 文件编码：统一 UTF-8
- 时间格式：YYYY-MM-DD
- 交互语言：中文

## 项目默认基准配置（新生成项目继承此规则）

### 通用规则
- 交互方式：OpenClaw 指令优先，外部存储（表格/文件）为数据底座
- 删除模式：默认软删除（标记作废）
- 节点/状态：手动标记，预留自动交互扩展位
- 架构模式：自动选择（4种模式之一）

### 目录固定规则
1. `agents/`：Agent 配置目录（主Agent + 子Agent）
2. `datas/`：全局配置、数据、日志、文档
3. `datas/logs/`：运行日志
4. `memory/`：三层记忆系统（默认生成 MEMORY.md + README.md + entities/）
5. `memory/entities/`：实体注册表（users/ projects/ tools/ concepts/）
6. `scripts/`：exec 调用脚本（sh / python 等），初始为空

### 根目录固定文件
AGENTS.md、SOUL.md、TOOLS.md、USER.md、IDENTITY.md、README.md、BOOTSTRAP.md、HEARTBEAT.md、ARCHITECTURE.md

## 模板目录配置
- 模板位置：`templates/`
- 模板格式：Markdown / JSON / YAML，使用 `{{变量名}}` 占位符
- 模板版本：V4.0（默认集成三层记忆系统框架，新增 templates/memory/ 模板目录）

## 生成行为配置
- 覆盖保护：禁止自动覆盖已有目录/文件
- 日志输出：生成过程简要日志展示
- 模板渲染：按需替换占位符，未匹配占位符保留原样
- 架构模式：自动选择，用户可通过自定义规则覆盖

## 架构模式选择规则（自动判断）
- 包含"管理""系统""调度""子Agent"等关键词 → hub-and-spoke
- 包含"管道""流水线""处理链""多阶段"等关键词 → pipeline
- 包含"监控""告警""事件""响应""触发"等关键词 → event-driven
- 包含"查询""查词""翻译""引擎""高频""搜索"等关键词 → daemon-agent
- 包含"分析""报表""SQL""数据库""数据看板""指标""归因"等关键词 → data-analysis
- 其余情况 → single-agent
