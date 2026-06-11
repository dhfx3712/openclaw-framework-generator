---
name: notification-channel-skill
description: 多通道通知技能。统一封装通知发送逻辑，支持 webchat 和飞书群两种渠道。通过「渠道配置」管理目标，AI 只需指定消息内容和目标，无需关心底层 API。
---

# 多通道通知技能

## 设计原则

1. **渠道抽象**：AI 只需说「通知 XXX」，由配置决定发到 webchat 还是飞书群
2. **配置驱动**：目标群/用户的 ID 和路由规则统一在配置中管理
3. **渐进扩展**：先 webchat 后飞书群，渠道可增量添加
4. **消息模板**：重复性通知使用模板，减少 AI 生成成本

## 渠道类型

| 渠道 | 底层工具 | 适用场景 | 优先级 |
|------|---------|---------|-------|
| `webchat` | 当前 session 回复 | 开发调试、个人通知 | 默认 |
| `feishu` | `message(action=send, channel="feishu")` | 团队协作、群通知 | 可选 |

## 渠道配置

```yaml
# datas/notification-channels.yaml

# 默认渠道（未指定目标时使用）
default_channel: "webchat"

# 渠道定义
channels:
  webchat:
    type: "webchat"
    label: "WebChat 控制台"
    enabled: true
    # webchat 无需额外配置，直接回复当前 session

  feishu:
    type: "feishu"
    label: "飞书群"
    enabled: true
    # 目标群列表
    groups:
      - id: "oc_xxxxxxxxxxxxx"     # 飞书群 chat_id
        name: "项目A通知群"          # 群名（用于 AI 识别）
        label: "项目A"              # 别名（用于指令中的目标名）
        enabled: true

      - id: "oc_yyyyyyyyyyyyy"
        name: "运维告警群"
        label: "运维"
        enabled: true

# 通知模板
templates:
  task_created:
    title: "📋 新任务创建"
    body: |
      任务：{{task_name}}
      负责人：{{assignee}}
      截止日期：{{due_date}}
      优先级：{{priority}}

  task_completed:
    title: "✅ 任务完成"
    body: |
      任务：{{task_name}}
      完成人：{{completed_by}}
      完成时间：{{completed_at}}

  alert:
    title: "🚨 告警通知"
    body: |
      级别：{{level}}
      内容：{{message}}
      时间：{{time}}

  system_status:
    title: "📊 系统状态"
    body: |
      {{status_lines}}
```

## 指令模板

### 1. 发送通知

```
/通知 【目标】 【消息内容】
/通知 【目标】 使用模板:【模板名】 【模板参数】
```

**示例：**
```
/通知 webchat 任务已创建成功
/通知 项目A 使用模板:task_created task_name=数据分析 report due_date=2026-06-15
/通知 运维 使用模板:alert level=高 message=服务器CPU超过90%
```

### 2. 通知目标查询

```
/通知列表
```

列出所有可用通知目标和渠道。

### 3. 通知目标管理

```
/通知 添加群 【群名】 【群chat_id】
/通知 移除群 【群名/标签】
```

## AI 执行步骤

### 发送通知流程

```
用户指令
  └→ 解析目标（webchat / 飞书群标签 / 群名）
       └→ 加载 datas/notification-channels.yaml
            ├→ 目标 = webchat
            │    └→ 直接回复当前 session（回复当前消息）
            │
            ├→ 目标 = 飞书群（通过 label 或 name 匹配）
            │    └→ 加载模板（如有）
            │         └→ 填充模板参数
            │              └→ 调用 message(action=send, channel="feishu", target="oc_xxx", message="...")
            │
            └→ 目标 = 全部
                 └→ 同时发送到所有 enabled 渠道
```

### 飞书群通知的 chat_id 获取

**首次接入一个飞书群：**

```
方法1：从飞书群 URL 获取
  打开飞书群 → 群设置 → 更多 → 复制群二维码/链接
  URL 中通常包含 chat_id（oc_xxx 格式）

方法2：通过 feishu_chat 工具
  如果 bot 已在群中，通过事件消息中的 chat_id 获取

方法3：手动添加
  在飞书群中添加机器人 → 机器人会收到入群事件
  从事件中提取 chat_id → 写入配置
```

> ⚠️ 用户提到的痛点：「不在飞书群里跟机器人互动一次，应该找不到对应群」
> 确实，飞书 API 没有「列出 bot 所在的所有群」的接口。
> 首次接入需要：
> 1. 将机器人拉入目标群
> 2. 在群里发一条消息（或等待入群事件）
> 3. 从消息/事件中获取 chat_id
> 4. 写入 `datas/notification-channels.yaml`

## 飞书群消息格式

### 纯文本消息

```python
# AI 调用
message(
  action="send",
  channel="feishu",
  target="oc_xxxxxxxxx",
  message="📋 新任务创建\n任务：数据分析报告\n负责人：张三\n截止日期：2026-06-15"
)
```

### 富文本消息（飞书支持 Markdown 子集）

支持：**加粗**、*斜体*、~~删除线~~、`行内代码`、[链接](url)、换行

## 配置管理

### 初始化配置

```yaml
# datas/notification-channels.yaml（首次创建）
default_channel: "webchat"
channels:
  webchat:
    type: "webchat"
    label: "WebChat 控制台"
    enabled: true
  feishu:
    type: "feishu"
    label: "飞书群"
    enabled: true
    groups: []
templates: {}
```

### 添加飞书群

```yaml
# 在 feishu.groups 中添加条目
- id: "oc_xxxxxxxxx"
  name: "项目A通知群"
  label: "项目A"
  enabled: true
```

## 错误处理

| 错误场景 | 提示信息 | 处理方式 |
|---------|---------|---------|
| 目标不存在 | `❌ 未找到通知目标 "{目标}"，可用目标：{列表}` | 列出可用目标 |
| 飞书群未配置 | `❌ 飞书群未配置，请先添加群` | 提示添加流程 |
| 飞书 API 错误 | `❌ 飞书消息发送失败：{错误信息}` | 返回原始错误 |
| 模板不存在 | `❌ 未找到模板 "{模板名}"` | 列出可用模板 |
| 模板参数缺失 | `❌ 模板缺少参数：{参数列表}` | 提示补充参数 |

## 目录结构

```
datas/
├── notification-channels.yaml    # 通知渠道配置
└── notification-templates/       # 可选：独立模板文件目录（按业务拆分）
    ├── task.yaml
    ├── alert.yaml
    └── report.yaml
```

## 快速接入步骤

### 第1步：初始化配置

创建 `datas/notification-channels.yaml`，写入基础配置。

### 第2步：添加飞书群

1. 在飞书中创建一个群（或使用已有群）
2. 将机器人添加到群中
3. 在群里发一条消息（或等入群事件）
4. 从消息中获取 `chat_id`
5. 写入配置文件的 `feishu.groups` 列表

### 第3步：注册指令

在 `SOUL.md` 或 `TOOLS.md` 中注册通知相关指令：

```
/通知 <目标> <消息>       — 发送通知
/通知列表                 — 查看可用目标
/通知 添加群 <名> <id>    — 添加飞书群
```

### 第4步：定义模板

在 `templates` 中定义业务通知模板，减少重复生成。
