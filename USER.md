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

### 目录固定规则
1. `datas/`：全局配置、数据、日志、文档
2. `datas/logs/`：运行日志
3. `memory/`：工作记忆/运行记录
4. `scripts/`：exec 调用脚本（sh / python 等），初始为空

### 根目录固定文件
AGENTS.md、SOUL.md、TOOLS.md、USER.md、IDENTITY.md、README.md、BOOTSTRAP.md、HEARTBEAT.md

## 模板目录配置
- 模板位置：`templates/`
- 模板格式：Markdown / JSON，使用 `{{变量名}}` 占位符
- 模板版本：V2.0（通用模板骨架）

## 生成行为配置
- 覆盖保护：禁止自动覆盖已有目录/文件
- 日志输出：生成过程简要日志展示
- 模板渲染：按需替换占位符，未匹配占位符保留原样
