# Fullstack Review

<!-- 修改时请同步更新 README.en.md -->

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Platform](https://img.shields.io/badge/Platform-React%20%7C%20NestJS%20%7C%20TypeScript-4F46E5)]()

**服务业全栈自动化代码审查 Skill** — React + TypeScript + NestJS 项目提交前自动审查，覆盖 Web / H5 / App / 小程序四端。

[English](./README.en.md)

---

## 📖 目录

- [快速开始](#-快速开始)
- [核心原则](#-核心原则)
- [核心能力](#-核心能力)
- [流水线架构](#-流水线架构)
- [两层防御](#-两层防御)
- [四端覆盖](#-四端覆盖)
- [命令行参数](#-命令行参数)
- [配置文件](#-配置文件)
- [报表样例](#-报表样例)
- [FAQ](#-faq)
- [License](#-license)

---

## 🚀 快速开始

### 前提条件

- **技术栈**：React + TypeScript + NestJS
- **操作系统**：Windows / macOS / Linux
- **版本管理**：Git
- **Node.js**：≥ 16

### 安装

```bash
cp -r fullstack-review/ ~/AppData/Local/hermes/skills/software-development/fullstack-review/
```

或在 Hermes 对话中加载：

```
@skill fullstack-review
```

### 使用

```bash
# 本地 pre-commit【增量模式】（默认）
git add .
hermes review

# CI 流水线【全基线模式】，不允许降级
hermes review --full
```

---

## 💡 核心原则

### Diff-Only：只审查你 `git add` 的文件

本地 pre-commit 模式下，**所有阶段只读取 `git diff --cached` 中的文件**，不遍历项目全库。如果你只改了 2 个 CSS 文件，就只审查这 2 个文件，不会去扫支付模块、鉴权代码等无关内容。

### 上下文不足则跳过，不强行全库扫描

当某个检查需要 diff 之外的参照文件才能完成时（如对比前后端路由需要两端文件都在 diff 中），标记 `⏭️ 跳过` 并写明原因，不做全库扫描。

### CI 兜底

本地跳过的基线校验，在 CI 流水线 `--full` 模式下强制完整扫描，作为代码入库最后一道闸门。

---

## 🔧 核心能力

### 13 阶段自动审查流水线

| 阶段 | 名称 | 扫描范围 |
|------|------|---------|
| P0 | 项目检测 | `package.json`（单文件） |
| P0.5 | 行为识别 | 仅 diff 文件推断业务场景 |
| P1 | 变更扫描 | `git diff --cached`；含同目录关联文件推断 |
| P2 | 安全扫描 | diff 文件新增行（含依赖CVE/日志脱敏/环境配置/AI应用安全） |
| P3 | 平台适配 | 仅 diff 文件中的跨端问题 |
| P4 | 前后端契约 | diff 中前后端同时存在才比对；缺一端则 ⏭️ 跳过 |
| P4.5 | 状态一致性 | 仅 diff 文件中的状态管理代码 |
| P4.6 | 多租户鉴权 | 分层：自检 ✅（始终执行）+ 跨文件对比 ⏭️（上下文不足跳过） |
| P5 | 业务深度审查 | 仅 diff 中归属已激活场景的文件；无匹配则 ⏭️ |
| P6 | 性能内存 | 仅 diff 组件 |
| P7 | 无障碍/国际化/交互体验 | 仅 diff 组件；含 6 条通用 UX 规则（删除确认/防重/错误封装等） |
| P8 | 异步任务 | diff 无队列文件则 ⏭️ 跳过 |
| P9 | AI 审查 | diff 喂给 AI + UX 链路推演 + 置信度过滤 + 双维度输出 |
| P10 | 报表 | 技术故障/用户投诉双板块 + 状态列 + 跳过原因 |
| P11 | 按需修复 | 黑白名单 + 编译校验 + 重验证 |

### 8 大业务场景（含 AI 应用安全）

| 场景 | 检测关键词 | 审查重点 |
|------|---------|---------|
| 💰 交易 | `pay`/`order` + 金额字段 | 支付幂等、签名验证、对账、退款 |
| 📅 资源抢占 | `capacity`/`slot`/`stock` | 时间冲突双验、并发锁、超时释放 |
| 🏠 线下履约 | `lat`/`lng`/`geo` | GPS 省电、签到双验、服务范围 |
| 🎫 权益 | `coupon`/`points`/`voucher` | 并发核销、条件前置校验、叠加精度 |
| 💬 信息交互 | 用户间双向操作 | 脱敏、权限提示、双向确认 |
| ⏳ 异步长任务 | 状态机 + queue/task 实体 | 进度持久化、超时、恢复 |
| 📨 消息队列 | MQ 依赖 + Queue 装饰器 | 幂等、死信、事务消息 |
| 🤖 AI 应用安全 | LLM SDK/向量数据库/RAG 框架 | 降级兜底、Prompt注入、输出安全渲染（P2.7按需激活） |

### 双分支审查

```
🔒 Tech Risk：并发/事务/幂等/数据一致/签名
🧑 UX Risk：  状态持久化/失败分级/弱网兜底/防重/前置校验
```

### UX 增强（v6.1）

- **6 条静态 UX 规则**：删除二次确认、提交防重、错误封装、空状态兜底、原生弹窗、输入提示
- **AI 链路推演**：P9 模拟用户操作路径，检测全链路体验崩坏
- **资损升级通道**：支付/退款/核销类 UX 缺陷达三要素门槛可升 🔴Critical（ux_derived_asset_loss）
- **双板块报表**：技术故障和用户投诉风险分开展示，UX 问题不被淹没

### AI 应用安全（v6.2）

检测到 LLM SDK/向量数据库/RAG 框架依赖时自动激活 P2.7：
- **降级兜底**：LLM 调用是否有 try-catch + 预设话术
- **输出安全渲染**：LLM 动态内容是否经过格式校验 + XSS 清洗
- **Prompt 注入防线**：用户输入是否在拼入 prompt 前经过清洗
- **Token 管理**：检索上下文是否有 topK/score/maxTokens 限制

### 审计增强（v6.3）

深度审计后集中修复 9 项，提升可信度与覆盖率：
- **🤖 AI 结论降权**：AI 单独命中的缺陷标注 `🤖【AI推断-待确认】` 标签，与静态规则视觉区分；资损升级门槛 0.9→0.95
- **~ Medium 透明化**：P0.5 Medium 置信度场景在报表中加 `~` 前缀，标注"P0.5推断建议核实"
- **📎 关联文件提示**：P1 自动检查同目录 Controller↔Service 是否同步变更，不在 diff 则标注提醒
- **🔧 P11 实现细节**：临时目录 + 6 步写入流程 + 5 场景回滚策略
- **📋 框架不匹配提示**：非 React/NestJS 项目自动提示引导

---

## 🛡️ 两层防御

```
① 本地 pre-commit（Incremental）
   git add . → hermes review
   仅审 diff → 快 → 上下文不足则跳过
   目标：增量前置拦截，保持开发流畅

② CI 流水线（Full Baseline）
   hermes review --full
   全库扫描 → 慢但完整 → 不允许降级
   目标：代码入库最后一道闸门
```

---

## 🖥️ 四端覆盖

| 维度 | Web | H5 | App | 小程序 |
|------|:---:|:---:|:---:|:---:|
| DOM API | ✅ | ✅ | ❌ | ❌ |
| CSS 单位 | vw/vh | vw/vh | StyleSheet | rpx |
| 网络 | WiFi | 弱网 | 弱网+离线 | 微信代理 |
| 存储 | 5MB+ | 有限 | 无限制 | 10MB |
| 推送 | SW | 兼容差 | FCM/APNs | 模板消息 |
| 鉴权 | Cookie/Token | Token | Token+生物 | wx.login |
| 支付 | 网页/扫码 | 唤起 | 原生SDK | wx.requestPayment |

---

## ⚙️ 命令行参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `--platforms=<list>` | 手动指定覆盖端 | `--platforms=web,h5,mini` |
| `--include=<list>` | 强制激活场景 | `--include=transaction` |
| `--exclude=<list>` | 关闭场景 | `--exclude=benefits` |
| `--full` | 全量基线模式（CI 用） | `hermes review --full` |
| `--lang=<zh\|en>` | 报表语言 | `--lang=en` |
| `--force-review` | 跳过 High 阻断 | `--force-review` |

---

## 📄 配置文件

`.hermes/review-config.json`（所有字段可选）：

```json
{
  "fullstack_review": {
    "platforms": ["web", "h5", "rn", "mini"],
    "scenarios": {
      "transaction": "auto",
      "resource_contention": "auto",
      "location_based": "auto",
      "benefits": "auto",
      "async_long_task": "auto",
      "info_exchange": "auto",
      "message_queue": "auto"
    },
    "ai_reviewer": { "confidence_threshold": 0.75 },
    "auto_fix": { "max_rounds": 2 },
    "report": { "lang": "zh", "output_format": ["terminal", "markdown"] }
  }
}
```

---

## 📊 报表样例

```
╔══════════════════════════════════════════════════╗
║              Code Review Report                  ║
║  项目: my-app | 分支: feat/payment               ║
║  时间: 2026-07-27 | 覆盖端: Web, H5, Mini        ║
╚══════════════════════════════════════════════════╝

📊 总览
┌──────────────────┬────┬────┬────┬────┬──────┬──────────┬─────────────────────────┐
│ 检查维度           │ 🔴 │ 🟠 │ 🟡 │ 🔵 │ 🚨UX │ 状态      │ 跳过原因                  │
├──────────────────┼────┼────┼────┼────┼──────┼──────────┼─────────────────────────┤
│ 安全 P2           │  2 │  1 │  0 │  0 │   0  │ ✅ 完整   │                         │
│ 路由对齐 P4       │  — │  — │  — │  — │   —  │ ⏭️ 跳过   │ 后端Controller不在diff    │
│ 鉴权基线 P4.6     │  0 │  1 │  0 │  0 │   0  │ ⚠️ 部分   │ 自检完成；对比跳过         │
│ 交易场景 P5.1      │  0 │  2 │  1 │  0 │   3  │ ✅ 完整   │                         │
│ 资源抢占 P5.3      │  — │  — │  — │  — │   —  │ ⏭️ 跳过   │ P0.5未检测到该特征         │
│ 异步任务 P8        │  — │  — │  — │  — │   —  │ ⏭️ 跳过   │ diff未检测队列代码          │
│ ...               │    │    │    │    │      │          │                         │
├──────────────────┼────┼────┼────┼────┼──────┼──────────┼─────────────────────────┤
│ 合计              │  3 │  8 │  5 │  1 │   5  │ 4模块跳过  │                         │
└──────────────────┴────┴────┴────┴────┴──────┴──────────┴─────────────────────────┘

🔧 可自动修复: 8 issues | ⚠️ 需人工判断: 6 issues
→ 回复 "修复" 开始自动修复
```

### 状态枚举

| 状态 | 含义 | 触发条件 |
|------|------|---------|
| ✅ 完整 | 所有检查均完成 | diff 范围内全部可执行 |
| ⚠️ 部分 | 部分执行 | P4.6 专属：自检完成，对比跳过 |
| ⏭️ 跳过 | 上下文不足 | 参照文件不在 diff 中 / P0.5 未激活 |

每条缺陷含四标签：

| 标签 | 可选值 |
|------|-------|
| Severity | `Critical` / `High` / `Medium` / `Info` |
| Experience Risk | `high_complaint` / `medium_complaint` / `no_risk` |
| Fix Priority | `block` / `this_iteration` / `next_iteration` / `optional` |
| Source | `【静态规则】` / `【AI评审】` |

---

## ❓ FAQ

<details>
<summary><b>只改了两个CSS文件，会扫全库吗？</b></summary>

不会。本地模式遵循 **Diff-Only 原则**，只审查 `git add` 的文件。改了 2 个 CSS → 只跑 P1/P2/P3/P10，秒级出报表。P4/P5/P8 等与 CSS 无关的模块直接 ⏭️ 跳过。

</details>

<details>
<summary><b>如果后端 Controller 不在 diff 里，路由对齐会怎么处理？</b></summary>

标记 `⏭️ 跳过：后端Controller不在diff，无法比对`。不会为了找 Controller 去扫全库。该检查会在 CI `--full` 模式下补全。

</details>

<details>
<summary><b>Auto-fix 会自动改我的并发逻辑吗？</b></summary>

**绝对不会。** 并发、事务、支付、状态机、签名、数据库、鉴权、MQ、租户隔离代码全部在黑名单中。

</details>

<details>
<summary><b>支持 Monorepo 吗？</b></summary>

✅ 原生支持，P0 自动扫描子包并分别识别前后端框架。

</details>

---

## 📦 Roadmap

- [ ] v7：历史基线对比（存储历史报表，追踪缺陷趋势）
- [ ] v7：CI/CD 原生集成插件（GitHub Actions / GitLab CI）
- [ ] v7：自定义规则扩展接口
- [ ] v7：Fast Path 智能路径检测（替代硬编码高风险路径名单，基于 import 频率自动识别）

---

## 📄 License

MIT © 2026
