# {{项目名}} - 记忆系统

## 三层记忆架构

```
memory/
├── MEMORY.md                  # Layer 1: 永久知识库
│   ├── 用户偏好
│   ├── 项目状态
│   ├── 决策记录（ADR）
│   ├── 已知问题
│   └── 关键事实
│
├── YYYY-MM-DD.md              # Layer 2: 会话日志
│   ├── 会话概要
│   ├── 完成事项
│   ├── 关键事实（→ 合并到 MEMORY.md）
│   └── 待办
│
└── entities/                  # Layer 3: 实体注册表
    ├── users/                 # 用户信息、偏好
    ├── projects/              # 项目状态、配置
    ├── tools/                 # 工具配置、连接信息
    └── concepts/              # 领域概念、术语
```

## 使用规则

### 会话开始
1. 读取 `MEMORY.md` 了解永久知识（偏好、项目状态、决策）
2. 读取 `entities/` 了解当前活跃实体的详细状态

### 会话中
- 用户明确说"记住..." → 立即写入 `MEMORY.md` 对应分类
- 做出架构决策 → 写入 `MEMORY.md` 决策记录 + `entities/` 对应实体
- 项目状态变更 → 更新 `entities/` 对应实体
- 零散事实 → 记录在会话日志的关键事实区块

### 会话结束
1. 写会话日志 `memory/YYYY-MM-DD.md`
2. 从关键事实区块提取 → 更新 `MEMORY.md`
3. 更新 `entities/` 中变更的实体

## 目录结构

```
memory/
├── MEMORY.md
├── entities/
│   ├── users/
│   ├── projects/
│   ├── tools/
│   └── concepts/
```
