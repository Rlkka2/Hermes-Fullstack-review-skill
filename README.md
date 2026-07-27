# Fullstack Review

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Platform](https://img.shields.io/badge/Platform-React%20%7C%20NestJS%20%7C%20TypeScript-4F46E5)]()

**服务业全栈自动化代码审查 Skill** — React + TypeScript + NestJS 项目提交前自动审查，覆盖 Web / H5 / App / 小程序四端。

[English](./README.en.md)

---

## 📖 目录

- [快速开始](#-快速开始)
- [核心能力](#-核心能力)
- [流水线架构](#-流水线架构)
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
# 增量模式（默认，提交前）
git add .
hermes review

# 指定覆盖端
hermes review --platforms=mini

# 强制激活业务场景
hermes review --include=transaction,message_queue

# 全量基线审计
hermes review --full

# 英文报表
hermes review --lang=en
```

### 提交流程

```
git add . && hermes review
    ↓
P0~P10 自动运行 → 输出审查报表
    ↓
┌─ 无 Critical → 直接 commit
├─ 有可修复项 → 回复 "修复" 或 "修复 #1 #3"
└─ 有 Critical → 必须修复后才能 commit
```

---

## 🔧 核心能力

### 13 阶段自动审查流水线

| 阶段 | 名称 | 职责 |
|------|------|------|
| P0 | 项目检测 | 框架识别 + Monorepo 检测 |
| P0.5 | 行为识别 | 业务特征识别 → 自动激活场景规则 |
| P1 | 变更扫描 | 代码变更 + 文案 + 删除文件风险 |
| P2 | 安全扫描 | 6 个子模块（密钥/注入/CVE/日志/环境） |
| P3 | 平台适配 | 四端 API 可用性 + 跨端体验一致性 |
| P4 | 前后端契约 | API 路由 + DTO + 响应包装 + 错误体系 |
| P4.5 | 状态一致性 | 全局状态可信源 + 中间态兜底 |
| P4.6 | 多租户鉴权 | 租户隔离 + 水平/垂直越权 + 接口守卫 |
| P5 | 业务深度审查 | 7 大场景按需激活，技术+体验双分支 |
| P6 | 性能内存 | 定时器清理/竞态/重渲染/虚拟滚动 |
| P7 | 无障碍国际化 | a11y/i18n/输入容错/隐私脱敏 |
| P8 | 异步任务 | 进度持久化/超时/页面关闭恢复 |
| P9 | AI 审查 | 独立子智能体 + 置信度过滤 |
| P10 | 报表 | 双标签报表 + fix_priority 自动赋值 |
| P11 | 按需修复 | 黑白名单 + 编译校验 + 重验证（最多2轮） |

### 7 大业务场景

| 场景 | 检测关键词 | 审查重点 |
|------|---------|---------|
| 💰 交易 | `pay`/`order` + 金额字段 | 支付幂等、签名验证、对账、退款 |
| 📅 资源抢占 | `capacity`/`slot`/`stock` | 时间冲突双验、并发锁、超时释放 |
| 🏠 线下履约 | `lat`/`lng`/`geo` | GPS 省电、签到双验、服务范围 |
| 🎫 权益 | `coupon`/`points`/`voucher` | 并发核销、条件前置校验、叠加精度 |
| 💬 信息交互 | 用户间双向操作 | 脱敏、权限提示、双向确认 |
| ⏳ 异步长任务 | 状态机 + queue/task 实体 | 进度持久化、超时、恢复 |
| 📨 消息队列 | MQ 依赖 + Queue 装饰器 | 幂等、死信、事务消息 |

### 双分支审查

```
🔒 Tech Risk：并发/事务/幂等/数据一致/签名
🧑 UX Risk：  状态持久化/失败分级/弱网兜底/防重/前置校验
```

### 智能修复

- ✅ **可自动修复**：样式、导入、条件编译、文案、TS 类型
- ❌ **禁止自动修复**：并发、事务、支付、状态机、签名、DB、鉴权、MQ、租户

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
| `--full` | 全量基线模式 | `hermes review --full` |
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
┌──────────────────┬────┬────┬────┬────┬──────┬──────────┐
│ 检查维度           │ 🔴 │ 🟠 │ 🟡 │ 🔵 │ 🚨UX │ 状态      │
├──────────────────┼────┼────┼────┼────┼──────┼──────────┤
│ 安全 P2           │  2 │  1 │  0 │  0 │   0  │ ✅ 已检   │
│ 平台适配 P3        │  0 │  2 │  3 │  0 │   2  │ ✅ 已检   │
│ 交易场景 P5.1      │  0 │  2 │  1 │  0 │   3  │ ✅ 已检   │
│ 资源抢占 P5.3      │  — │  — │  — │  — │   —  │ ⏭️ 跳过   │
│ ...               │    │    │    │    │      │          │
├──────────────────┼────┼────┼────┼────┼──────┼──────────┤
│ 合计              │  4 │  8 │  8 │  1 │   6  │ 3模块跳过  │
└──────────────────┴────┴────┴────┴────┴──────┴──────────┘

🔧 可自动修复: 8 issues | ⚠️ 需人工判断: 6 issues
→ 回复 "修复" 开始自动修复
```

每条缺陷含四标签：

| 标签 | 可选值 | 说明 |
|------|-------|------|
| Severity | `Critical` / `High` / `Medium` / `Info` | 技术严重度 |
| Experience Risk | `high_complaint` / `medium_complaint` / `no_risk` | 体验风险度 |
| Fix Priority | `block` / `this_iteration` / `next_iteration` / `optional` | 修复优先级（自动赋值） |
| Source | `【静态规则】` / `【AI评审】` | 缺陷来源 |

---

## ❓ FAQ

<details>
<summary><b>和 <code>requesting-code-review</code> 有什么区别？</b></summary>

| | requesting-code-review | fullstack-review |
|---|---|---|
| 语言 | 通用 | TS/React/NestJS 深度定制 |
| 平台感知 | 无 | Web/H5/App/小程序 |
| 前后端交互 | 无 | API契约+DTO+错误体系 |
| 业务场景 | 无 | 7大场景按需激活 |
| 用户体验 | 无 | 双分支审查 |
| 多租户/鉴权 | 无 | ✅ |
| 依赖安全 | 无 | ✅ |

两者可共存。

</details>

<details>
<summary><b>MQ 检查会拖慢没有消息队列的项目吗？</b></summary>

不会。MQ 场景通过 P0.5 双特征识别按需激活，无 MQ 依赖的项目零扫描开销。

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

- [ ] v2：历史基线对比
- [ ] v2：CI/CD 原生集成（GitHub Actions / GitLab CI）
- [ ] v2：自定义规则扩展接口

---

## 📄 License

MIT © 2026
