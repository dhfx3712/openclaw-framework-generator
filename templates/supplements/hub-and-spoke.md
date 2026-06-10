# hub-and-spoke 模式 — 手动补充清单

## 架构说明
1 主 Agent + N 个子 Agent，星型数据流，适用于多任务编排项目。

## 需要手动补充

### 🔴 高优先级（必须）
- [ ] **主Agent 身份**：填写 `agents/main-agent/IDENTITY.md`（名称、定位、核心职责）
- [ ] **主Agent 技能**：填写 `agents/main-agent/skill.yaml`（路由规则、技能清单）
- [ ] **子Agent 定义**：在 `AGENTS.md` 中定义每个子 Agent 的职责和能力边界
- [ ] **子Agent 身份**：为每个子 Agent 创建 `agents/{子Agent名}/IDENTITY.md`
- [ ] **子Agent 技能**：为每个子 Agent 创建 `agents/{子Agent名}/skill.yaml`
- [ ] **路由规则**：在 `SOUL.md` 中定义主 Agent 如何将请求路由到子 Agent
- [ ] **业务指令定义**：在 `TOOLS.md` 中定义所有指令
- [ ] **数据底座配置**：在 `datas/config.json` 的 `storage.config` 中填写连接信息
- [ ] **SOUL.md 角色定位**：补充主 Agent 和子 Agent 的角色描述
- [ ] **IDENTITY.md 身份信息**：填写名称、定位、核心使命

### 🟡 中优先级（建议）
- [ ] **子 Agent 间通信协议**：定义子 Agent 之间如何传递数据
- [ ] **输出格式规范**：在 `SCRIPTS.md` 中定义业务输出格式
- [ ] **状态机定义**：在 `ARCHITECTURE.md` 中定义实体状态流转
- [ ] **cron 任务注册**：如有定时任务，通过 OpenClaw cron 模块注册

### 🟢 低优先级（按需）
- [ ] 业务脚本编写（`scripts/` 目录）
- [ ] 心跳任务配置（`HEARTBEAT.md`）
