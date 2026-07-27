## 前置依赖

本文件规则依赖以下概念，如未加载请先查阅：

| 依赖 | 位置 |
|------|------|
| 两种运行模式 | SKILL.md |
| 阻断阈值 | SKILL.md commit 阻断阈值 |

---


## P11：按需修复（On-demand Auto-fix）

### 触发方式

用户看完报表后发出修复指令：
- `修复` → 全部可自动修复项
- `修复 #1 #3 #7` → 仅修复指定编号

### 修复黑白名单

| ✅ Allowlist（允许自动修复） | ❌ Blocklist（禁止自动修复） |
|---------------------------|---------------------------|
| 样式兼容（rpx↔px、vw↔%） | 并发逻辑（锁/乐观锁/幂等） |
| 导入语句（添加/删除/排序） | 事务代码（`$transaction`、`BEGIN/COMMIT`） |
| 条件编译包裹（`process.env.TARO_ENV`） | 支付流程（notify/callback/验签/退款） |
| 文案格式（统一提示语） | 状态流转/状态机（订单/预约/权益状态变更） |
| TS 类型注解遗漏 | 签名/验签逻辑 |
| 清理未使用变量/导入 | 数据库操作（SQL/Prisma query/migration） |
| 添加 `aria-label` 属性 | 删除文件恢复 |
| 添加 `disabled` / `loading` 到按钮 | 权限/鉴权逻辑 |
| | 消息队列消费者/生产者逻辑 |
| | 租户隔离/数据隔离逻辑 |

### 修复流程

```
用户指令 "修复"
    ↓
执行自动修复（仅 Allowlist 项）
    ↓
强制运行 TS 类型校验（npx tsc --noEmit）
    ├── 失败 → 回滚本次修复 → 终止 → 剩余问题移交人工
    └── 通过 ↓
强制运行 ESLint 校验（npx eslint）
    ├── 失败 → 回滚本次修复 → 终止 → 剩余问题移交人工
    └── 通过 ↓
重跑 P2~P9 完整评审（基于修复后的代码快照）
    ├── 本轮 issue 全部消除 → 进入 commit
    ├── 仍有 issue 且修复轮次 < 2 → 回到修复循环
    └── 修复轮次 >= 2 → 终止 → 输出剩余问题，移交人工
```

### 重要约束

- 修复基于**内存中的代码快照**，不反复写入 git 暂存区，避免污染 git 索引
- 最多 2 轮修复迭代，到达上限停止
- 第 2 轮修复仅处理第 1 轮验证后发现的新问题（不重新修同一项）
- 每轮修复后**必须**重跑完整验证
- 全量基线模式下 P11 不启动

### 实现细节：临时目录与回滚策略（Implementation Details）

#### 文件写入流程

```
用户指令 "修复"
     ↓
① 将 diff 中的所有变更文件复制到 .hermes/tmp/auto-fix/{timestamp}/
   （确保不影响工作区和 git 暂存区）
     ↓
② 在临时副本上依次应用 Allowlist 修复
     ↓
③ 将修复后的文件写入 .hermes/tmp/auto-fix/{timestamp}/patched/
     ↓
④ 以 patched/ 目录为工作区执行验证：
   npx tsc --noEmit --project .hermes/tmp/auto-fix/{timestamp}/tsconfig.json
   npx eslint .hermes/tmp/auto-fix/{timestamp}/patched/
     ↓
   ├── 验证通过 → ⑤
   └── 验证失败 → ⑥
     ↓
⑤ 将 patched/ 中的文件逐个复制回原路径（覆盖工作区文件）
   然后执行 git add 更新暂存区 → 进入下一轮或完成
     ↓
⑥ 丢弃本次修复：删除 .hermes/tmp/auto-fix/{timestamp}/
   输出验证失败的具体错误信息 → 人工介入
```

#### 临时目录结构

```
.hermes/tmp/auto-fix/
└── {timestamp}/
    ├── original/        # 原始 diff 文件副本（用于回滚）
    ├── patched/         # 修复后的文件
    └── tsconfig.json    # 临时 TS 配置（映射到 patched/ 路径）
```

#### 回滚策略

| 场景 | 回滚动作 |
|------|---------|
| `npx tsc --noEmit` 失败 | 删除临时目录，工作区文件不受影响 |
| `npx eslint` 失败 | 同上 |
| P2~P9 重验证仍有未消除 issue | 同上 + 输出剩余问题列表 |
| 修复轮次达到上限（2轮） | 同上 |
| 修复过程中进程崩溃 | 临时目录在 `.hermes/tmp/` 下，下次启动自动清理 |

> **关键安全保证**：修复只在临时副本上执行，验证失败后自动丢弃。工作区文件在验证通过前**不会被修改**。

---
