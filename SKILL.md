---
name: fullstack-review
description: "React+TS+NestJS 服务业 SaaS 全栈预提交自动化评审 Skill，覆盖 Web/H5/RN/小程序四端，15 阶段流水线、7 大业务场景扫描，内置多租户鉴权、支付、MQ 安全校验，支持 AI 逻辑评审与可控自动修复，区分技术故障与用户投诉双维度风险。"
version: 6.3.1
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags:
      - code-review
      - fullstack
      - react
      - nestjs
      - service-industry
      - cross-platform
      - ux-review
    related_skills:
      - requesting-code-review
      - fullstack-nestjs-react
      - react-spa-dev
      - nestjs-prisma-mysql-fullstack
---

# Fullstack Review — 服务业全栈自动化审查

React + TypeScript + NestJS 全栈项目提交前自动审查，覆盖 Web / H5 / App / 小程序四端。
以服务业线上服务平台通用特征为基线，兼顾技术稳定性与跨端用户体验，不绑定特定业务产品。

---

## 术语使用强制规范（Terminology Enforcement）

> **本规范仅约束机器可读字段**：阶段编号（P0~P11）、枚举常量（Critical/High/Medium/Info）、配置字段名、命令行参数、JSON 字段 key、报表标签。**不约束业务描述语言**——风险解读、体验分析、检查说明使用简体中文，术语在其中自然出现即可（如"做幂等处理"和"实现 Idempotent 机制"均可接受）。

---

## 术语对照表（Glossary）

| 中文术语 | English（固定标识，不可修改） | 用途 |
|---------|---------------------------|------|
| 唯一可信源 | Single Source of Truth (SSOT) | 状态一致性检查 |
| 幂等 | Idempotent | 支付/核销/重复请求 |
| 状态机 | State Machine | 业务流转检查 |
| 乐观锁 | Optimistic Lock | 并发控制 |
| 悲观锁 / 行锁 | Pessimistic Lock / Row Lock | 事务检查 |
| 跨端体验一致性 | Cross-platform UX Consistency | P3 检查维度 |
| 弱网兜底 | Weak-network Fallback | 用户体验分支 |
| 中间态 | Intermediate State | 状态持久化检查 |
| 降级 | Degradation | 平台能力差异处理 |
| 自动修复 | Auto-fix | P11 阶段 |
| 自动修复白名单 | Auto-fix Allowlist | 允许自动修复的规则类型 |
| 自动修复黑名单 | Auto-fix Blocklist | 禁止自动修复的规则类型 |
| 增量模式 | Incremental Mode | 提交前流水线（默认） |
| 全量基线模式 | Full Baseline Mode | 手动触发全库扫描 |
| 置信度 | Confidence Score | AI Reviewer 结论可信度 |
| 体验风险度 | Experience Risk Level | 报表双标签之一 |
| 场景激活矩阵 | Scenario Activation Matrix | P0.5 输出结果 |
| 行为特征识别 | Behavioral Pattern Detection | P0.5 检测方法 |
| 预激活 | Pre-activation | P0.5 自动推断结果 |
| 死锁 | Deadlock | 数据库锁检查 |
| 竞态条件 | Race Condition | 并发请求检查 |
| 对账 | Reconciliation | 支付场景定时核对 |
| 回调 | Callback（sync） | 支付同步返回 |
| 异步通知 | Notify（async） | 支付异步回调 |
| 核销 | Redemption / Verification | 权益消耗/线下签到 |
| 死信队列 | Dead Letter Queue (DLQ) | 消息队列失败处理 |
| 多租户 | Multi-tenancy | SaaS 数据隔离 |
| 水平越权 | Horizontal Privilege Escalation | 同角色跨资源访问 |
| 垂直越权 | Vertical Privilege Escalation | 低权限访问高权限接口 |
| 租户隔离 | Tenant Isolation | 多租户数据隔离 |
| 修复优先级 | Fix Priority | 报表缺陷排序标签 |
| Monorepo | Monorepo | 多包单仓库架构 |
| CVE | CVE (Common Vulnerabilities and Exposures) | 依赖安全漏洞 |

---

## 场景标识枚举表（Scenario Identifiers）

配置文件 `scenarios` 字段与 `--include` / `--exclude` 参数使用以下英文标识：

| 中文名称 | 英文标识（用于 --include/--exclude/config） | 对应检查模块 |
|---------|------------------------------------------|-------------|
| 交易场景 | `transaction` | P5.1 |
| 资源抢占场景 | `resource_contention` | P5.3 |
| 线下履约场景 | `location_based` | P5.4 |
| 权益场景 | `benefits` | P5.5 |
| 异步长任务场景 | `async_long_task` | P8（独立阶段） |
| 信息交互场景 | `info_exchange` | P5.6 |
| 消息队列场景 | `message_queue` | P5.2-ext（按需激活） |
| AI 应用场景 | `ai_application` | P2.7（按需激活） |

---

## 全局严重度判定框架（Severity Framework）

以下为指导性分级框架，非强制死规则。新增检查项时参照此框架赋值：

| 严重度 | 标识 | 判定标准 |
|-------|------|---------|
| Critical | 🔴 | 会导致资金损失 / 数据泄露 / 服务不可用 / 法律合规风险 |
| High | 🟠 | 大概率引发用户投诉或线上故障，但不会立即造成资金或数据损失 |
| Medium | 🟡 | 影响局部体验或代码质量，不引发直接事故 |
| Info | 🔵 | 建议性改进，不修复不影响正常运行 |

> 不同维度（安全/性能/体验）的 Critical 伤害等级不同，允许按上下文调整，但必须在本框架范围内。

---

## CLI 参数汇总表（CLI Parameters）

| 参数 | 作用 | 示例 |
|------|------|------|
| `--platforms=<list>` | 手动指定覆盖端，覆盖 P0 自动推断 | `--platforms=web,h5,mini` |
| `--include=<list>` | 手动强制激活场景规则集 | `--include=transaction,message_queue` |
| `--exclude=<list>` | 手动关闭场景规则集 | `--exclude=benefits` |
| `--full` | 启用全量基线模式（替代默认增量模式） | `hermes review --full` |
| `--lang=<zh\|en>` | 报表语言切换（默认 `zh`） | `--lang=en` |
| `--force-review` | 跳过阻断，强制提交（仅限 High 及以下，Critical 不可跳过） | `--force-review` |

---

## 两种运行模式（Two Modes）

| | Incremental Mode（默认） | Full Baseline Mode |
|---|---|---|
| 触发方式 | `git commit` 前自动触发 | 手动执行 `hermes review --full` |
| 扫描范围 | `git diff --cached` | 对比分支完整差异（非暂存区） |
| 执行速度 | 快 | 较慢 |
| Auto-fix (P11) | ✅ 开放 | ❌ 禁止（防止大面积改动存量代码） |
| 适用场景 | 日常提交流水线 | CI 流水线 / 项目初次接入 / 季度审计 |
| 文案扫描 | 仅本次变更文件 | 全库文案一致性扫描 |
| 降级策略 | **上下文不足则跳过，报表标注** | **不允许降级，所有基线强制完整扫描** |

> 自动修复 P11 仅在 **Incremental Mode** 开放；Full Baseline Mode 只输出报告，不执行任何自动修改。

---

## Diff-Only 原则（Diff-Only Principle）

**Incremental Mode 核心规则：只审查 `git diff --cached` 中的文件，不遍历项目全库。**

- 凡是可以仅靠 diff 文件完成的检查 → 正常执行
- 凡是需要 diff 之外的文件作为参照才能完成的检查 → ⏭️ 跳过，报表标注原因
- 没有任何检查会静默跳过——所有跳过项都会在报表中显式展示原因

Full Baseline Mode（`--full`）不受此限，所有基线规则强制全库扫描，不允许降级。

> **设计哲学**：本地 pre-commit 以人为中心，只关心本次提交的变更，追求低延迟和符合直觉；
> CI 流水线作为代码入库最后一道闸门，使用 `--full` 模式兜底，不放过任何基线违规。

---

## 全局缺陷去重机制（Global Dedup）

所有阶段输出的缺陷在进入 P10 报表前执行去重：

```
去重键（Dedup Key） = file + line_number
合并规则:
  - 同一 file:line 命中多个阶段 → 合并为一条
  - 严重度取最高（Critical > High > Medium > Info）
  - 体验风险度取最高（high_complaint > medium_complaint > no_risk）
  - 来源标签合并（如同时被静态规则和 AI 命中 → 标注【静态规则+AI评审】）
```

---

## 暂存区读取规范（Staging Area Reading）

> **强制规则**：所有阶段读取变更文件内容时，必须使用 `git show :<path>` 获取暂存区快照，**禁止直接读取工作区文件**。
>
> 原因：用户可能使用 `git add -p` 只暂存文件的部分改动。若直接读工作区文件，会把未 add 的代码也纳入审查，直接违反 Diff-Only 原则。

---

## 非文本文件过滤（Non-text Filtering）

以下文件类型在 P1 后直接排除，不进入 P2~P9 代码扫描：

- **后缀黑名单**：`.png` `.jpg` `.gif` `.svg` `.ico` `.woff` `.woff2` `.ttf` `.eot` `.mp4` `.mp3` `.pdf` `.zip` `.lock` `.env` `.gitignore` `.dockerignore`
- **内容兜底**（实现层）：读取文件内容时，检测到高比例不可打印字符 → 自动标记为非文本，跳过扫描

---

## 流水线概览（Pipeline Overview）

```
P0    项目类型检测（含 Monorepo 检测）
P0.5  业务行为识别（仅 diff 文件推断场景）
P1    变更扫描（含删除文件路径匹配优先级 + 文案文件白名单）
      ├── 暂存区读取：强制 git show :<path>，禁止读工作区
      ├── 非文本过滤：二进制/图片/.lock 等直接排除
      └── Fast Path 六档分流（变更类型预判）
            ├── 🟢 纯非代码 (.md/.txt/.png/.lock/.gitignore)
            ├── 🟢 前端样式 (.css/.scss/.less)
            ├── 🟢 前端工具 (utils/通用组件，无路由/权限)
            ├── 🟢 后端工具 (纯.ts工具，无Controller/Service)
            ├── 🔴 高风险路径 (命中前后端默认名单 → 强制P2+P9)
            └── 🟡 全栈变更 (业务代码 → 完整流水线)
P2    安全扫描
        ├── P2.1 硬编码密钥/注入漏洞
        ├── P2.2 NestJS 专项安全
        ├── P2.3 React 前端专项安全
        ├── P2.4 Dependency Security（依赖 CVE 扫描）
        ├── P2.5 Log Desensitization（日志敏感信息脱敏）
        ├── P2.6 Env Config Isolation（环境配置隔离）
        └── P2.7 AI Application Security（AI 应用安全与可靠性，按需激活）
P3    平台适配 + 跨端体验一致性（仅 diff 文件）
P4    前后端契约 + 错误体系（两端同在 diff 才比对；缺一端→⏭️+弱提示）
P4.5  用户状态一致性基线（仅 diff 文件）
P4.6  多租户 & 权限鉴权基线（分层：自检✅ + 对比⏭️；含删除鉴权检测，前后端对称）
P5    业务场景深度审查（P0.5 输出驱动，仅加载激活场景规则；仅扫 diff 中归属文件）
        ├── 5.1 交易场景
        ├── 5.2 数据库锁与事务（diff 无 DB 操作→⏭️）
        │     └── 5.2-ext 消息队列（按需激活）
        ├── 5.3 资源抢占场景
        ├── 5.4 线下履约场景
        ├── 5.5 权益场景
        └── 5.6 信息交互场景
P6    性能与内存（仅 diff 组件）
P7    可访问性 & 国际化 & 输入容错 & 脱敏（仅 diff 组件）
P8    异步长任务体验（diff 无队列文件→⏭️）
P9    AI Reviewer 子智能体（含大 diff 分片规则）
P10   报表生成（含状态列/跳过原因/部分执行细化/CI提示条件/跳过统计行/路由弱提示）
P11   按需修复
```

### P4.5 / P5 / P8 边界三角

这是整个流水线最容易混淆的三个阶段，必须先划定边界：

```
┌─────────────────────────────────────────────────────────────┐
│ P4.5 → 全局状态一致性基线（通用机制，不关心"什么业务"）          │
│        检查：状态是否后端唯一可信源？                           │
│             切换端/刷新/杀进程后状态是否自动同步？               │
│             中间态（处理中/等待中/审核中）是否有统一 UI 兜底？     │
│        ❌ 不涉及：具体业务的状态机流转逻辑                       │
├─────────────────────────────────────────────────────────────┤
│ P5   → 业务场景深度审查（关心"具体业务"）                       │
│        🔒 技术风险分支：并发锁、事务、幂等、数据一致性            │
│        🧑 用户体验分支：状态持久化、失败分级提示、               │
│                        弱网兜底、操作防重、条件前置校验           │
│        ❌ 不涉及：长任务的生命周期体验（那是 P8 的活）            │
├─────────────────────────────────────────────────────────────┤
│ P8   → 异步长任务体验（跨场景通用，不关心"任务里做什么"）          │
│        检查：进度是否持久化？超时后如何处理？                     │
│             页面关闭后重进能否恢复？是否有预估时长？              │
│        ❌ 不涉及：任务内部的业务逻辑正确性（那是 P5 的活）         │
└─────────────────────────────────────────────────────────────┘
```

**举例说明边界**：
- 用户下单后支付 → 支付幂等、金额校验走 **P5.1 交易场景**
- 支付后触发后台发货任务 → 任务进度持久化、超时兜底走 **P8**
- 用户在任何端刷新页面看到支付状态一致 → 走 **P4.5**
- 用户 A 不应看到用户 B 的订单 → 走 **P4.6**

---

## Fast Path 六档分流（概念层）

P1 之后根据变更文件类型执行分流判定，避免"改了 2 个 CSS 文件跑全套流水线"。

**六档权重链**：`纯非代码 < 前端样式 < 前端工具/后端工具 < 全栈 < 高风险路径`

**优先级规则**：
- **就高不就低**：变更中包含更高权重文件 → 升级至对应路径
- **高风险路径最高优先级**：命中高风险名单→**叠加**当前档位检查+P2+P9（不替换）
- **CI `--full` 强制禁用**：全量模式不执行任何短路

> 具体分流规则表（每档命中条件/执行范围/跳过内容）→ [references/p1-change-scan.md](references/p1-change-scan.md)
> 高风险底层路径名单 → [references/p1-change-scan.md](references/p1-change-scan.md)

---

## commit 阻断阈值

```
存在任意 🔴Critical → 阻断提交（不可跳过，必须修复）
所有 Critical = 0，但存在 🟠High → 可通过 --force-review 强制跳过
所有 Critical = 0 且 High = 0 → 直接通过
```

---

## 两层防御架构

```
① 本地 pre-commit（Incremental Mode）
   仅审 diff → 快 → 上下文不足则跳过
   目标：增量前置拦截，保持开发流畅
   
② CI 流水线（Full Baseline Mode：hermes review --full）
   全库扫描 → 慢但完整 → 不允许降级
   目标：代码入库最后一道闸门
```

### 使用方式

```bash
# 模式1：本地 pre-commit【增量模式】（默认）
git add .
hermes review

# 模式2：CI 流水线【全基线模式】，不允许降级
hermes review --full
```

### 提交流程状态机

```
hermes review
    ↓
P0~P10 自动运行 → 输出报表
    ↓
┌─ 所有 Clear（无 Critical + 无 High）→ 直接通过
├─ 存在 🔴Critical → 阻断提交 → 必须修复 → 重验证
├─ 有可修复项 → 用户决定 "修复" / "修复 #N" / "跳过"
│   ├─ "修复" → P11 → 重验证 → 通过或终止
│   └─ "跳过"（仅无 Critical 时可用）→ 通过（带未修复问题记录）
└─ 使用 --force-review → 跳过 High 级别阻断（Critical 不可跳过）
```

---

## 阶段索引（Stage Index）

> 🎯 = 何时加载 reference | ⏭️ = 可跳过条件 | 📄 = reference 文件路径 | 📦 = 子模块

---

### P0 — 项目检测 + 框架不匹配提示

📄 [references/p0-project-detection.md](references/p0-project-detection.md)
🎯 始终执行（读取 `package.json`）
⏭️ 无
📦 Monorepo检测 · 依赖特征推断框架 · 覆盖端推断 · 框架不匹配提示

---

### P0.5 — 业务行为识别

📄 [references/p0-project-detection.md](references/p0-project-detection.md)（与 P0 同文件）
🎯 P0 确认后自动执行
⏭️ 无（Low 置信度场景需交互确认，Medium 自动化但 `~` 前缀标注）
📦 8场景行为特征检测 · 置信度标记（High/Medium/Low）· `~` 前缀透明化 · 预激活矩阵输出

---

### P1 — 变更扫描

📄 [references/p1-change-scan.md](references/p1-change-scan.md)
🎯 始终执行（`git diff --cached`）
⏭️ Empty diff → 提示 `git add` 后退出
📦 暂存区读取 · 非文本过滤 · Fast Path 分流规则表 · 同目录关联文件推断 · 高风险底层路径名单 · 删除文件风险识别

---

### P2 — 安全扫描

📄 [references/p2-security.md](references/p2-security.md)
🎯 始终执行（P2.7 按需激活：P0.5 检测到 `ai_application` 场景时加载）
⏭️ 无（P2.1~P2.6 始终执行，仅增量扫描 diff 新增行）
📦 P2.1 硬编码密钥/注入 · P2.2 NestJS安全 · P2.3 React XSS · P2.4 依赖CVE · P2.5 日志脱敏 · P2.6 环境配置 · P2.7 AI应用安全

---

### P3 — 平台适配 + 跨端体验一致性

📄 [references/p3-platform.md](references/p3-platform.md)
🎯 始终执行（仅审查 diff 文件）
⏭️ P0 覆盖端仅 Web → 跳过小程序/App 专项 API 规则
📦 API可用性检查 · 条件编译遗漏 · 跨端一致性（上传限制/压缩比/输入长度/授权弹窗）

---

### P4 — 前后端契约 + 错误体系

📄 [references/p4-contract.md](references/p4-contract.md)
🎯 仅当 diff 中前端 API 文件和后端 Controller **同时存在**时加载对比规则
⏭️ 单侧变更（仅前端或仅后端）→ ⏭️ 跳过，附加 🟡 弱提示"建议人工核对"
📦 P4.1 路由对齐 · P4.2 DTO匹配 · P4.3 响应包装 · P4.4 错误体系+UX友好度 · P4.5 状态一致性基线 · P4.6 多租户&权限鉴权（分层：自检✅+对比⏭️）

---

### P5 — 业务场景深度审查

📄 [references/p5-scenarios.md](references/p5-scenarios.md)
🎯 **P0.5 输出驱动**，仅加载已激活场景的规则子集（按需加载，不遍历全部 7 个场景）
⏭️ 跳过条件：P0.5 未激活任何场景 → P5 整体跳过，标注 `⏭️ P5 全部场景未激活`
📦 5.1 交易 · 5.2 数据库锁与事务 · 5.2-ext 消息队列（按需激活）· 5.3 资源抢占 · 5.4 线下履约 · 5.5 权益 · 5.6 信息交互
🔒🧑 每个激活场景分两支：Tech Risk（并发/锁/幂等/数据一致）+ UX Risk（状态持久化/失败分级/弱网兜底/防重/前置校验）

---

### P6+P7+P8 — 性能 + 可访问性 + 异步长任务

📄 [references/p6-p8-quality.md](references/p6-p8-quality.md)
🎯 P6/P7：仅扫描 diff 中的前端组件文件；P8：diff 含队列/任务文件时加载
⏭️ P8：diff 无队列/任务相关代码 → 整体跳过
📦 P6 性能内存（定时器/订阅竞态/重复渲染/虚拟滚动）· P7 a11y/i18n/输入容错/隐私脱敏/通用UX交互6条 · P8 异步长任务体验（进度持久化/失败重试/超时处理）

---

### P9 — AI Reviewer 子智能体

📄 [references/p9-ai-reviewer.md](references/p9-ai-reviewer.md)
🎯 仅前端业务变更 / 全栈变更 / 高风险路径三档触发
⏭️ 纯非代码 / 前端样式 / 前后端工具类档位 → 跳过
📦 UX链路推演 · 逻辑错误检测 · 边界场景遗漏 · 置信度过滤 · `🤖【AI推断-待确认】` 标签 · 资损升级通道（门槛≥0.95）· 大diff分片规则

---

### P10 — 报表生成

📄 [references/p10-report.md](references/p10-report.md)
🎯 始终执行（汇总 P2~P9 所有阶段输出，执行全局去重）
⏭️ 无
📦 总览矩阵（含 `~` 前缀注释）· 双板块拆分（技术故障/用户投诉风险）· 状态列（✅完整/⚠️部分/⏭️跳过）· 排序规则 · CI兜底提示 · 跳过项统计 · AI建议附录

---

### P11 — 按需修复

📄 [references/p11-auto-fix.md](references/p11-auto-fix.md)
🎯 用户发出"修复"指令后加载
⏭️ Full Baseline 模式 / 无可修复项 → 跳过
📦 黑白名单 · 临时目录+回滚策略 · 编译校验（tsc+eslint）· 重验证 · 2轮上限

---

## 与 `requesting-code-review` 的关系

本 Skill 是 `requesting-code-review` 的**服务业全栈增强版**：

| | requesting-code-review | fullstack-review |
|---|---|---|
| 语言 | Python/JS/Go/Rust 通用 | **TS/React/NestJS 深度定制** |
| 平台感知 | 无 | **Web/H5/App/小程序 四端自适应** |
| 前后端交互 | 无 | **API契约+DTO匹配+响应包装+错误体系** |
| 业务场景 | 无 | **7 大服务业场景按需激活** |
| 用户体验 | 无 | **双分支审查（Tech+UX）+ 双标签报表** |
| 多租户鉴权 | 无 | **P4.6 全局基线** |
| 依赖安全 | 无 | **P2.4 增量 CVE 扫描** |
| AI Reviewer | ✅ | ✅（增加置信度 + 分片规则 + 职责边界） |
| Auto-fix | 2轮通用修复 | **2轮 + 黑白名单 + 编译校验 + 重验证** |
| 删除文件检测 | 无 | **✅ 业务源码删除自动提权** |
| 全局去重 | 无 | **✅ file:line 去重** |

---

## 注意事项（Pitfalls）

1. **Empty diff**：`git diff --cached` 为空 → 提示用户 `git add` 或退出
2. **非 git 仓库**：跳过并提示用户
3. **P0.5 低置信度漏检**：多层封装/抽象基类/纯前端场景可能导致行为识别失效 → 必须开放手动覆写 + Low 置信度强制交互确认
4. **P9 AI Reviewer 误报**：置信度 < 0.75 的结论归入附录，避免污染正式报表
5. **P11 修复引入新问题**：每轮修复后自动重验证，兜底发现
6. **全量模式禁止修复**：P11 仅在 Incremental Mode 下启动
7. **文案跨文件不一致**：Incremental Mode 只检查 diff 内的文案；全量文案一致性需使用 Full Baseline Mode
8. **大 diff（>15,000 字符）**：P9 按文件边界自动分片，结果合并去重
9. **P4.5/P8 重复告警**：依赖全局 `file:line` 去重机制，合并为一条保留最高严重度
10. **依赖安全扫描**：先跑全量 `npm audit` 获取漏洞数据，再按 diff 中新增/升级的包名过滤结果（不直接逐包扫描，避免遗漏存量依赖的已知漏洞）
