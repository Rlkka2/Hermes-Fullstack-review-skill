# Fullstack Review

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Platform](https://img.shields.io/badge/Platform-React%20%7C%20NestJS%20%7C%20TypeScript-4F46E5)]()

**Automated Pre-commit Review for Service-Industry Fullstack Projects** — React + TypeScript + NestJS, covering Web / H5 / App / Mini Program platforms.

[中文](./README.md)

---

## 📖 Table of Contents

- [Quick Start](#-quick-start)
- [Core Capabilities](#-core-capabilities)
- [Pipeline Architecture](#-pipeline-architecture)
- [Cross-Platform Coverage](#-cross-platform-coverage)
- [CLI Parameters](#-cli-parameters)
- [Configuration](#-configuration)
- [Report Sample](#-report-sample)
- [FAQ](#-faq)
- [License](#-license)

---

## 🚀 Quick Start

### Prerequisites

- **Stack**: React + TypeScript + NestJS
- **OS**: Windows / macOS / Linux
- **VCS**: Git
- **Node.js**: ≥ 16

### Installation

```bash
cp -r fullstack-review/ ~/AppData/Local/hermes/skills/software-development/fullstack-review/
```

Or load directly in a Hermes conversation:

```
@skill fullstack-review
```

### Usage

```bash
# Incremental mode (default, pre-commit)
git add .
hermes review

# Specify platforms
hermes review --platforms=mini

# Force-enable scenarios
hermes review --include=transaction,message_queue

# Full baseline audit
hermes review --full

# English report
hermes review --lang=en
```

### Commit Flow

```
git add . && hermes review
    ↓
P0~P10 auto-run → Output report
    ↓
┌─ No Critical → Commit directly
├─ Auto-fixable → Reply "fix" or "fix #1 #3"
└─ Critical present → Must fix before commit
```

---

## 🔧 Core Capabilities

### 13-Stage Automated Pipeline

| Stage | Name | Responsibility |
|-------|------|---------------|
| P0 | Project Detection | Framework ID + Monorepo detection |
| P0.5 | Behavioral Detection | Feature-based scenario activation |
| P1 | Change Scan | Code diff + text resources + deleted file risk |
| P2 | Security Scan | 6 sub-modules (secrets/injection/CVE/log/env) |
| P3 | Platform Adaptation | Cross-platform API + UX consistency |
| P4 | Frontend-Backend Contract | API routing + DTO + error system |
| P4.5 | State Consistency | SSOT + intermediate state fallback |
| P4.6 | Multi-tenancy & Auth | Tenant isolation + privilege escalation + guards |
| P5 | Business Deep Review | 7 scenarios on-demand, Tech+UX dual-branch |
| P6 | Performance & Memory | Timer cleanup/race condition/re-render/virtual scroll |
| P7 | a11y & i18n | Accessibility/internationalization/input tolerance/privacy |
| P8 | Async Tasks | Progress persistence/timeout/page-close recovery |
| P9 | AI Reviewer | Independent sub-agent + confidence filtering |
| P10 | Report | Dual-label report + auto fix_priority |
| P11 | On-demand Fix | Allowlist/Blocklist + compile check + re-validation |

### 7 Business Scenarios

| Scenario | Detection | Key Checks |
|----------|-----------|------------|
| 💰 Transaction | `pay`/`order` + amount fields | Idempotency, signature, reconciliation, refund |
| 📅 Resource Contention | `capacity`/`slot`/`stock` | Conflict validation, concurrency lock, timeout release |
| 🏠 Location-based | `lat`/`lng`/`geo` | GPS battery, check-in verification, service range |
| 🎫 Benefits | `coupon`/`points`/`voucher` | Concurrent redemption, pre-validation, stacking precision |
| 💬 Info Exchange | Bidirectional ops | Desensitization, permission notice, mutual confirmation |
| ⏳ Async Long-task | State machine + queue/task entities | Progress persistence, timeout, recovery |
| 📨 Message Queue | MQ deps + Queue decorators | Idempotency, DLQ, transactional messages |

### Dual-Branch Review

```
🔒 Tech Risk: Concurrency/Transaction/Idempotency/Data Integrity/Signature
🧑 UX Risk:  State Persistence/Error Classification/Weak-network Fallback/Dedup/Pre-validation
```

### Smart Auto-fix

- ✅ **Allowed**: styles, imports, conditional compilation, text, TS types
- ❌ **Blocked**: concurrency, transactions, payments, state machines, signatures, DB, auth, MQ, tenants

---

## 🖥️ Cross-Platform Coverage

| Dimension | Web | H5 | App | Mini Program |
|-----------|:---:|:---:|:---:|:---:|
| DOM API | ✅ | ✅ | ❌ | ❌ |
| CSS Units | vw/vh | vw/vh | StyleSheet | rpx |
| Network | WiFi | Weak net | Weak+offline | WeChat proxy |
| Storage | 5MB+ | Limited | Unlimited | 10MB |
| Push | SW | Poor | FCM/APNs | Template msg |
| Auth | Cookie/Token | Token | Token+Biometric | wx.login |
| Payment | Web/QR | App-switch | Native SDK | wx.requestPayment |

---

## ⚙️ CLI Parameters

| Parameter | Purpose | Example |
|-----------|---------|---------|
| `--platforms=<list>` | Override platforms | `--platforms=web,h5,mini` |
| `--include=<list>` | Force-enable scenarios | `--include=transaction` |
| `--exclude=<list>` | Disable scenarios | `--exclude=benefits` |
| `--full` | Full baseline mode | `hermes review --full` |
| `--lang=<zh\|en>` | Report language | `--lang=en` |
| `--force-review` | Skip High blocks | `--force-review` |

---

## 📄 Configuration

`.hermes/review-config.json` (all fields optional):

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

## 📊 Report Sample

```
╔══════════════════════════════════════════════════╗
║              Code Review Report                  ║
║  Project: my-app | Branch: feat/payment          ║
║  Time: 2026-07-27 | Platforms: Web, H5, Mini     ║
╚══════════════════════════════════════════════════╝

📊 Overview
┌──────────────────┬────┬────┬────┬────┬──────┬──────────┐
│ Dimension         │ 🔴 │ 🟠 │ 🟡 │ 🔵 │ 🚨UX │ Status   │
├──────────────────┼────┼────┼────┼────┼──────┼──────────┤
│ Security P2       │  2 │  1 │  0 │  0 │   0  │ ✅ Done   │
│ Platform P3       │  0 │  2 │  3 │  0 │   2  │ ✅ Done   │
│ Transaction P5.1  │  0 │  2 │  1 │  0 │   3  │ ✅ Done   │
│ Resource P5.3     │  — │  — │  — │  — │   —  │ ⏭️ Skipped │
│ ...               │    │    │    │    │      │          │
├──────────────────┼────┼────┼────┼────┼──────┼──────────┤
│ Total             │  4 │  8 │  8 │  1 │   6  │ 3 skipped │
└──────────────────┴────┴────┴────┴────┴──────┴──────────┘

🔧 Auto-fixable: 8 issues | ⚠️ Manual review: 6 issues
→ Reply "fix" to start auto-fix
```

Each issue carries four labels:

| Label | Values | Description |
|-------|--------|-------------|
| Severity | `Critical` / `High` / `Medium` / `Info` | Technical severity |
| Experience Risk | `high_complaint` / `medium_complaint` / `no_risk` | UX complaint risk |
| Fix Priority | `block` / `this_iteration` / `next_iteration` / `optional` | Auto-assigned priority |
| Source | `【Static Rule】` / `【AI Review】` | Detection origin |

---

## ❓ FAQ

<details>
<summary><b>How is this different from <code>requesting-code-review</code>?</b></summary>

| | requesting-code-review | fullstack-review |
|---|---|---|
| Languages | General | TS/React/NestJS deep |
| Platform-aware | No | Web/H5/App/Mini Program |
| Frontend-Backend | No | API contract + DTO + error system |
| Scenarios | No | 7 on-demand |
| UX Review | No | Dual-branch |
| Multi-tenancy/Auth | No | ✅ |
| Dependency security | No | ✅ |

Both can coexist.

</details>

<details>
<summary><b>Will MQ checks slow down projects without message queues?</b></summary>

No. MQ scenarios activate on-demand via P0.5 dual-detection. Zero overhead without MQ dependencies.

</details>

<details>
<summary><b>Will Auto-fix modify my concurrency logic?</b></summary>

**Absolutely not.** Concurrency, transactions, payments, state machines, signatures, database ops, auth, MQ, and tenant isolation are all blocklisted.

</details>

<details>
<summary><b>Monorepo support?</b></summary>

✅ Yes, natively. P0 auto-scans subdirectories and applies correct rules per package.

</details>

---

## 📦 Roadmap

- [ ] v2: Historical baseline comparison
- [ ] v2: CI/CD native integration (GitHub Actions / GitLab CI)
- [ ] v2: Custom rule extension API

---

## 📄 License

MIT © 2026
