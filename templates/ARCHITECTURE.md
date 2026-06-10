# {{项目名}} - 架构设计文档

## 1. 架构模式

**模式：{{架构模式}}**

{{架构模式说明}}

## 2. 数据流

```
{{#数据流}}
{{描述}}
{{/数据流}}
```

### 数据流说明

| 环节 | 输入 | 处理 | 输出 | 错误处理 |
|------|------|------|------|---------|
| {{#数据流环节}}
| {{环节名}} | {{输入}} | {{处理}} | {{输出}} | {{错误处理}} |
{{/数据流环节}}

## 3. 实体状态机

{{#实体列表}}
### 实体：{{实体名}}

**状态定义：**
{{#状态}}
- `{{状态名}}` — {{状态说明}}
{{/状态}}

**状态流转：**
```
{{#流转}}
{{前置状态}} ──[{{触发条件}}]──→ {{后置状态}}
{{/流转}}
```

**状态所有者：** {{状态所有者}}

{{/实体列表}}

## 4. 脚本集成契约

### 调用规范

| 维度 | 规范 |
|------|------|
| 调用方式 | `exec` 命令调用，OpenClaw 统一调度 |
| 输入方式 | 命令行参数 / stdin JSON / 环境变量 |
| 输出格式 | stdout 输出 JSON（必须可解析） |
| 成功标志 | 退出码 0 |
| 错误标志 | 退出码非 0，stderr 输出错误信息 |
| 超时处理 | 默认 30s 超时，超时视为失败 |

### 脚本清单

{{#脚本列表}}
#### {{脚本名}}（{{脚本路径}}）

```python
# 输入示例
{{输入示例}}

# 输出示例
{{输出示例}}
```

| 项目 | 说明 |
|------|------|
| 用途 | {{用途}} |
| 调用方 | {{调用方}} |
| 输入参数 | {{输入参数}} |
| 输出格式 | {{输出格式}} |
| 错误码 | {{错误码}} |

{{/脚本列表}}

## 5. 定时任务规范

### 任务清单

{{#定时任务列表}}
#### {{任务名}}

| 项目 | 说明 |
|------|------|
| 调度表达式 | {{cron表达式}} |
| 触发动作 | {{触发动作}} |
| 执行脚本 | {{执行脚本}} |
| 结果处理 | {{结果处理}} |
| 失败处理 | {{失败处理}} |

{{/定时任务列表}}

### 注册方式

通过 OpenClaw cron 模块注册：
```
cron(action=add, job={
  name: "{{任务名}}",
  schedule: { kind: "cron", expr: "{{cron表达式}}" },
  payload: { kind: "systemEvent", text: "{{触发消息}}" },
  sessionTarget: "main"
})
```

## 6. 错误处理模式

| 错误类型 | 处理策略 | 重试策略 | 通知方式 |
|---------|---------|---------|---------|
| {{#错误处理列表}}
| {{错误类型}} | {{处理策略}} | {{重试策略}} | {{通知方式}} |
{{/错误处理列表}}

## 7. 复杂业务架构参考（flow/skill 分层）

> 当业务逻辑复杂（多步流程、条件分支、数据校验），纯 AI 指令不可靠时，
> 建议参考以下分层架构，将确定性逻辑从 AI 中剥离到 Python 代码中。

### 分层结构

```
AI 层（SOUL.md 路由表）
  │  职责：意图识别 + 参数提取（AI 擅长的事）
  │  输出：exec run_flow.py <flow_name> '<slots_json>' '<ctx_json>'
  ▼
桥接层（run_flow.py）
  │  职责：统一入口，按 flow_name 路由到对应 flow
  │  文件：run_flow.py（约 30 行）
  ▼
流程层（flow/）
  │  职责：业务流程编排，调用多个 skill 组合完成一个业务
  │  结构：flow/{业务名}_flow/flow_run.py + flow.yaml
  ▼
技能层（skill/）
  │  职责：原子业务操作，一个 skill 只做一件事
  │  结构：skill/{业务名}_skill/{技能名}_skill.py + skill.yaml
  ▼
公共层（common/）
  │  职责：共享工具函数（存储读写、时间解析、数据模型）
  │  结构：common/{工具名}.py
  ▼
数据层（飞书表格 / 本地文件 / 数据库）
```

### 各层职责

| 层级 | 适合 AI 做？ | 适合 Python 做？ | 说明 |
|------|:---:|:---:|------|
| **AI 层** | ✅ | ❌ | 理解用户意图、提取参数、判断优先级 |
| **桥接层** | ❌ | ✅ | 固定路由，统一入口 |
| **流程层** | ❌ | ✅ | 多步编排，条件分支，确定性的 |
| **技能层** | ❌ | ✅ | 数据校验、API 调用、计算逻辑 |
| **公共层** | ❌ | ✅ | 工具函数，可复用 |
| **数据层** | ❌ | ✅ | 存储读写 |

### 目录结构

```
{项目名}/
├── run_flow.py              # 桥接脚本（统一入口）
├── flow/                    # 流程层
│   ├── {业务1}_flow/
│   │   ├── flow.yaml        # 流程元信息（flow_name、cron、bind_agent）
│   │   └── flow_run.py      # 流程执行逻辑
│   └── {业务2}_flow/
├── skill/                   # 技能层
│   ├── {业务1}_skill/
│   │   ├── skill.yaml       # 技能元信息
│   │   ├── {技能1}.py       # 原子技能实现
│   │   └── {技能2}.py
│   └── {业务2}_skill/
├── common/                  # 公共层
│   ├── storage.py           # 存储读写
│   ├── time_utils.py        # 时间解析
│   └── model.py             # 数据模型
├── intent/                  # 意图定义（可选）
│   ├── {业务1}_intent.yaml
│   └── {业务2}_intent.yaml
└── cron/                    # 定时任务配置（可选）
    └── {任务名}_cron.json
```

### 何时使用此架构

| 条件 | 建议 |
|------|------|
| 业务逻辑简单（CRUD） | 不需要，AI 直接调工具即可 |
| 有 3 步以上流程 | ✅ 建议使用 flow/skill |
| 有复杂数据校验 | ✅ 建议使用 skill |
| 有定时任务需要稳定执行 | ✅ 建议使用 flow |
| 多个业务共享工具函数 | ✅ 建议使用 common |
| 需要单元测试 | ✅ 建议使用 flow/skill |

### 参考示例

```python
# run_flow.py — 桥接脚本
import sys, json, asyncio

async def main():
    flow_name = sys.argv[1]
    slots = json.loads(sys.argv[2]) if len(sys.argv) > 2 else {}
    ctx = json.loads(sys.argv[3]) if len(sys.argv) > 3 else {}

    if flow_name == "todo_create":
        from flow.todo_create_flow.flow_run import run_flow
        result = await run_flow(slots, ctx)
    elif flow_name == "remind_query":
        from flow.remind_query_flow.flow_run import run_flow
        result = await run_flow(slots, ctx)
    # ... 新增 flow 在此注册

    print(json.dumps(result, ensure_ascii=False))

if __name__ == "__main__":
    asyncio.run(main())
```

```python
# skill/todo_skill/save_todo_skill.py — 原子技能示例
async def save_todo(slots: dict) -> dict:
    """保存待办到飞书表格"""
    # 数据校验
    if not slots.get("title"):
        return {"success": False, "msg": "待办标题不能为空"}
    # 调用飞书 API
    result = await feishu_bitable_create_record(...)
    return {"success": True, "data": result}
```

---

## 8. 扩展点

| 扩展点 | 说明 | 如何扩展 |
|--------|------|---------|
| {{#扩展点列表}}
| {{扩展点}} | {{说明}} | {{扩展方式}} |
{{/扩展点列表}}
