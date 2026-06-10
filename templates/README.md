# {{项目名}}

{{业务描述}}

## 目录结构

```
{{项目名}}/
├── agents/             # Agent 配置目录
│   ├── main-agent/     # 主Agent 身份+技能
│   └── ...             # 子Agent 身份+技能（hub-and-spoke 模式）
├── datas/              # 共享数据目录
│   ├── config.json     # 全局超参数
│   ├── logs/           # 运行日志
│   └── ...             # 业务配套文档
├── memory/             # 工作记忆/运行记录
├── scripts/            # 业务脚本（sh / python / exec）
├── AGENTS.md           # Agent 架构
├── ARCHITECTURE.md     # 架构设计文档
├── BOOTSTRAP.md        # 启动引导
├── HEARTBEAT.md        # 心跳任务
├── IDENTITY.md         # Agent 身份
├── README.md           # 本文件
├── SCRIPTS.md          # 脚本架构
├── SOUL.md             # 项目理念
├── TOOLS.md            # 指令工具清单
└── USER.md             # 用户配置
```

## 快速开始

1. 确认 `datas/config.json` 配置正确
2. 查看 `BOOTSTRAP.md` 了解启动就绪状态
3. 通过 OpenClaw 加载项目目录
4. 使用 `/` 指令与 Agent 交互

## 配置说明

所有全局参数统一存放在 `datas/config.json`，支持按需修改。
业务文档位于 `datas/` 目录下。
