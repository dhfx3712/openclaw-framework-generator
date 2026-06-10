# OpenClaw 框架生成器 Generator

## 简介
通用脚手架 Agent，用于在 OpenClaw 内部一键生成标准化项目框架，替代外部部署脚本。
全流程闭环，新建项目后可独立加载、迭代、运行。

## 目录结构

```
openclaw-framework-generator/
├── templates/           # 通用模板文件（供渲染生成新项目使用）
│   ├── AGENTS.md        # Agent 架构模板
│   ├── SOUL.md          # 项目理念模板
│   ├── TOOLS.md         # 指令工具模板
│   ├── USER.md          # 用户配置模板
│   ├── IDENTITY.md      # Agent 身份模板
│   ├── README.md        # 项目说明模板
│   ├── BOOTSTRAP.md     # 启动引导模板
│   ├── HEARTBEAT.md     # 心跳任务模板
│   └── config.json      # 全局超参数模板
├── datas/               # 共享数据目录
│   └── logs/            # 运行日志
├── scripts/             # 业务脚本（初始为空）
├── AGENTS.md            # Agent 架构
├── BOOTSTRAP.md         # 启动引导
├── HEARTBEAT.md         # 心跳任务
├── IDENTITY.md          # Agent 身份
├── README.md            # 本文件
├── SOUL.md              # 项目理念
├── TOOLS.md             # 指令工具清单
└── USER.md              # 用户配置
```

## 使用方式

### 创建新项目
```
/创建项目 【项目名】 【业务描述】
```

示例：
```
/创建项目 task-manager 团队任务管理与进度追踪系统
```

### 查看模板规范
```
/查看模板规范
```

### 查看已创建项目
```
/查看已创建项目
```

## 模板渲染说明

所有模板文件位于 `templates/` 目录，使用 `{{变量名}}` 占位符。
创建项目时自动读取模板，替换占位符后写入目标项目目录。

支持的占位符：
- `{{项目名}}` — 项目英文名
- `{{业务描述}}` — 用户提供的业务简介
- `{{核心理念}}` — 项目核心理念
- `{{身份定位}}` — Agent 身份定位
- `{{数据底座}}` — 数据存储方式

## 版本
V2.0 — 通用模板骨架，支持单Agent/多Agent动态生成
