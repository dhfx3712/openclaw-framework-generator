---
name: feishu-bitable-skill
description: 通用飞书多维表格 CRUD 模板。基于 OpenClaw 内置 feishu_bitable_* 工具封装，通过「槽位配置」将字段定义与业务逻辑解耦，提高执行效率和数据质量。
---

# 飞书多维表格通用 CRUD 技能

## 设计原则

1. **槽位驱动**：每个表（table）的字段结构预定义为「槽位配置」，CRUD 操作通过槽位名引用字段，不硬编码 field_key
2. **类型感知**：根据字段类型自动选择正确的写入格式（文本、单选、多选、日期、用户、数字、URL 等）
3. **校验前置**：创建/更新前按槽位配置做类型和必填校验
4. **统一入口**：所有 CRUD 通过 `feishu_bitable_*` 工具完成，不混用其他 API

## 工作流程

```
用户指令
  └→ 解析意图（CRUD + 表名 + 数据）
       └→ 加载对应表的「槽位配置」
            └→ 参数校验（类型/必填/枚举）
                 └→ 调用 feishu_bitable_* 工具
                      └→ 结果格式化返回
```

## 槽位配置规范

每个表对应一个 YAML 配置，定义在 `datas/bitable-slots/` 目录下。

### 配置文件格式

```yaml
# datas/bitable-slots/{业务表名}.yaml

# 飞书多维表格元信息
app_token: "{{app_token}}"
table_id: "{{table_id}}"

# 表显示名（用于用户交互）
table_name: "{{表显示名}}"

# 字段槽位定义
slots:
  - name: "title"                    # 槽位名（AI 引用用）
    label: "标题"                     # 显示名
    field_key: "fldxxxxx1"           # 飞书 field_key（从 feishu_bitable_list_fields 获取）
    type: "text"                     # 字段类型（见下方类型映射表）
    required: true                   # 是否必填
    description: "任务的标题"          # 语义说明（帮助 AI 理解）
    max_length: 200                  # 可选：最大长度

  - name: "status"
    label: "状态"
    field_key: "fldxxxxx2"
    type: "single_select"
    required: true
    options:                         # 单选/多选的可选值列表
      - "待处理"
      - "进行中"
      - "已完成"
      - "已取消"

  - name: "priority"
    label: "优先级"
    field_key: "fldxxxxx3"
    type: "single_select"
    required: false
    options:
      - "P0"
      - "P1"
      - "P2"
      - "P3"

  - name: "assignee"
    label: "负责人"
    field_key: "fldxxxxx4"
    type: "user"                     # 用户字段
    required: false

  - name: "due_date"
    label: "截止日期"
    field_key: "fldxxxxx5"
    type: "date"
    required: false

  - name: "tags"
    label: "标签"
    field_key: "fldxxxxx6"
    type: "multi_select"
    required: false
    options:
      - "重要"
      - "紧急"
      - "bug"
      - "feature"

  - name: "progress"
    label: "进度"
    field_key: "fldxxxxx7"
    type: "number"
    required: false
    min: 0
    max: 100

  - name: "link"
    label: "关联链接"
    field_key: "fldxxxxx8"
    type: "url"
    required: false

  - name: "remark"
    label: "备注"
    field_key: "fldxxxxx9"
    type: "text"
    required: false
    max_length: 500

# 查询默认排序
default_order:
  field: "fldxxxxx5"   # 截止日期
  desc: false

# 列表页大小
page_size: 20
```

## 字段类型 ↔ 飞书 API 写入格式映射

| 槽位 type | 飞书字段类型 | feishu_bitable_create_record / update_record 的字段值格式 |
|-----------|-------------|------------------------------------------------------|
| `text` | 1-文本 | 字符串 `"内容"` |
| `number` | 2-数字 | 数字 `123` |
| `single_select` | 3-单选 | 字符串 `"选项名"` |
| `multi_select` | 4-多选 | 字符串数组 `["选项A", "选项B"]` |
| `date` | 5-日期 | 毫秒时间戳 `1700000000000` |
| `checkbox` | 7-复选框 | 布尔值 `true` / `false` |
| `user` | 11-用户 | 对象数组 `[{"id": "ou_xxx"}]` |
| `phone` | 13-电话 | 字符串 `"13800138000"` |
| `url` | 15-链接 | 对象 `{"text": "显示文本", "link": "https://..."}` |
| `attachment` | 17-附件 | 字符串数组 `["file_token"]` |
| `created_time` | 1001-创建时间 | 自动，不写入 |
| `modified_time` | 1002-修改时间 | 自动，不写入 |
| `created_user` | 1003-创建人 | 自动，不写入 |
| `modified_user` | 1004-修改人 | 自动，不写入 |
| `auto_number` | 1005-自动编号 | 自动，不写入 |

## CRUD 操作模板

### 1. 查询记录（LIST）

```yaml
# 模板参数
action: list
table: "{{表名}}"
filters: {}                    # 可选：过滤条件 { "槽位名": "值" }
page_size: 20                  # 可选：默认从槽位配置读取
page_token: ""                 # 可选：分页令牌
```

**AI 执行步骤：**
1. 加载 `datas/bitable-slots/{{表名}}.yaml` 获取 `app_token`、`table_id`
2. 调用 `feishu_bitable_list_records(app_token, table_id, page_size, page_token)`
3. 结果按槽位配置的 `label` 映射为可读字段名返回

### 2. 查询单条记录（GET）

```yaml
# 模板参数
action: get
table: "{{表名}}"
record_id: "{{record_id}}"
```

**AI 执行步骤：**
1. 加载槽位配置
2. 调用 `feishu_bitable_get_record(app_token, table_id, record_id)`
3. 结果按槽位映射返回

### 3. 创建记录（CREATE）

```yaml
# 模板参数
action: create
table: "{{表名}}"
data:
  title: "示例标题"            # 使用槽位名
  status: "待处理"
  priority: "P2"
  assignee: [{"id": "ou_xxx"}]
  due_date: 1700000000000
  tags: ["重要"]
  progress: 50
```

**AI 执行步骤：**
1. 加载槽位配置
2. **校验**：检查所有 `required: true` 的槽位是否都有值
3. **校验**：检查 `single_select`/`multi_select` 的值是否在 `options` 列表中
4. **校验**：检查 `number` 是否在 `min`/`max` 范围内
5. **校验**：检查 `text` 是否超过 `max_length`
6. 按类型映射表将值转换为飞书 API 格式
7. 调用 `feishu_bitable_create_record(app_token, table_id, fields)`
8. 返回创建结果（含新 record_id）

### 4. 更新记录（UPDATE）

```yaml
# 模板参数
action: update
table: "{{表名}}"
record_id: "{{record_id}}"
data:
  status: "进行中"
  progress: 80
```

**AI 执行步骤：**
1. 加载槽位配置
2. **校验**：只校验传入的字段（部分更新）
3. 按类型映射表转换值
4. 调用 `feishu_bitable_update_record(app_token, table_id, record_id, fields)`
5. 返回更新结果

### 5. 删除记录（DELETE）

```yaml
# 模板参数
action: delete
table: "{{表名}}"
record_id: "{{record_id}}"
mode: "soft"                   # soft=软删除（标记状态）, hard=硬删除（不支持）
```

**AI 执行步骤：**
1. 加载槽位配置
2. 如果 `mode: soft`，查找是否有 `status` 或类似状态字段，将其设为"已删除"/"已作废"
3. 如果无状态字段且 `mode: hard`，提示不支持硬删除（飞书 API 无删除记录接口）
4. 软删除实际调用 `feishu_bitable_update_record` 更新状态字段

> ⚠️ 飞书多维表格 API **不支持删除记录**。软删除是推荐做法：通过状态字段标记。

### 6. 列出字段（DESCRIBE）

```yaml
# 模板参数
action: describe
table: "{{表名}}"
```

**AI 执行步骤：**
1. 加载槽位配置
2. 返回字段列表（含 label、type、required、options）
3. 可选择性调用 `feishu_bitable_list_fields` 做交叉验证

## 槽位配置管理流程

### 首次接入一个新表

```
1. 获取飞书多维表格 URL
2. 调用 feishu_bitable_get_meta(url) → 获取 app_token + table_id
3. 调用 feishu_bitable_list_fields(app_token, table_id) → 获取所有字段信息
4. 根据字段信息编写 datas/bitable-slots/{表名}.yaml
5. 在 SOUL.md 或 AGENTS.md 中注册该表的路由
```

### 表结构变更时

```
1. 调用 feishu_bitable_list_fields 获取最新字段列表
2. 对比现有 datas/bitable-slots/{表名}.yaml
3. 更新新增/修改/删除的槽位
```

## 多表路由规则

在 `SOUL.md` 或 `AGENTS.md` 中定义：

```yaml
# 用户意图 → 表名映射
intent_routes:
  - intent: "创建任务|添加待办|新建任务"
    table: "tasks"
    action: "create"
  - intent: "查看任务|查询任务|列出任务"
    table: "tasks"
    action: "list"
  - intent: "更新任务|修改任务|完成任务"
    table: "tasks"
    action: "update"
  - intent: "删除任务|取消任务"
    table: "tasks"
    action: "delete"
```

## 错误处理

| 错误场景 | 提示信息 | 处理方式 |
|---------|---------|---------|
| 必填字段缺失 | `❌ 缺少必填字段：{字段名}` | 要求用户补充 |
| 枚举值不合法 | `❌ {字段名} 的值必须是：{可选值列表}` | 列出可选值让用户选择 |
| 数字超出范围 | `❌ {字段名} 的取值范围是 {min}-{max}` | 提示正确范围 |
| 文本超长 | `❌ {字段名} 最长 {max_length} 字` | 提示截断或修改 |
| 表名不存在 | `❌ 未找到表 "{表名}"，可用表：{表列表}` | 列出可用表 |
| API 错误 | `❌ 飞书 API 错误：{错误信息}` | 返回原始错误信息 |

## 性能优化建议

1. **批量操作**：需要批量创建/更新时，串行逐个调用（飞书 API 无批量接口）
2. **分页查询**：大数据量时使用 `page_token` 分页，默认 `page_size: 20`
3. **缓存槽位配置**：槽位配置加载后可在会话内缓存，避免重复读取
4. **字段列表缓存**：`feishu_bitable_list_fields` 结果可缓存，仅在表结构变更时刷新

## 目录结构

```
datas/
├── bitable-slots/          # 槽位配置目录
│   ├── tasks.yaml          # 任务表槽位配置
│   ├── users.yaml          # 用户表槽位配置
│   └── ...                 # 更多表
```

## 快速接入步骤

1. 在 `datas/bitable-slots/` 下创建 `{表名}.yaml`
2. 填写 `app_token`、`table_id`、`table_name`
3. 调用 `feishu_bitable_list_fields` 获取所有字段
4. 为每个字段定义槽位（name、label、field_key、type、required）
5. 在 `SOUL.md` 的 `intent_routes` 中注册路由
6. 完成，AI 即可通过槽位名进行 CRUD 操作
