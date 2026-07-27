# Fullstack Review

<!-- Keep in sync with README.md -->

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Platform](https://img.shields.io/badge/Platform-React%20%7C%20NestJS%20%7C%20TypeScript-4F46E5)]()

**Automated Pre-commit Review for Service-Industry Fullstack Projects** — React + TypeScript + NestJS, covering Web / H5 / App / Mini Program platforms.

[中文](./README.md)

---

## 📖 Table of Contents

- [Quick Start](#-quick-start)
- [Core Principle](#-core-principle)
- [Core Capabilities](#-core-capabilities)
- [Pipeline Architecture](#-pipeline-architecture)
- [Two-Layer Defense](#-two-layer-defense)
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

Or load in a Hermes conversation:

```
@skill fullstack-review
```

### Usage

```bash
# Local pre-commit [Incremental Mode] (default)
git add .
hermes review

# CI pipeline [Full Baseline Mode], no degradation allowed
hermes review --full
```

---

## 💡 Core Principle

### Diff-Only: Only reviews the files you `git add`'ed

In local pre-commit mode, **every stage only reads files from `git diff --cached`** — no full-repository traversal. If you only changed 2 CSS files, it only reviews those 2 files. Payment modules, auth code, and other unrelated code are left alone.

### Context-Missing → Skip, never force full-scan

When a check needs reference files outside the diff (e.g., route alignment needs both the frontend API file and the backend Controller in diff), it marks `⏭️ Skipped` with a clear reason instead of scanning the entire repo.

### CI as the safety net

Baseline checks skipped locally are enforced in CI via `--full` mode as the final merge gate.

---

## 🔧 Core Capabilities

### 13-Stage Automated Pipeline

| Stage | Name | Scan Scope |
|-------|------|-----------|
| P0 | Project Detection | `package.json` (single file) |
| P0.5 | Behavioral Detection | Diff files only for scenario inference |
| P1 | Change Scan | `git diff --cached`; incl. same-dir related file inference |
| P2 | Security Scan | Diff new lines (incl. CVE/log/env/AI app security) |
| P3 | Platform Adaptation | Diff files only |
| P4 | Frontend-Backend Contract | Both sides in diff → compare; one side missing → ⏭️ |
| P4.5 | State Consistency | Diff files only |
| P4.6 | Multi-tenancy & Auth | Layered: self-check ✅ (always) + cross-file ⏭️ (skip if context missing) |
| P5 | Business Deep Review | Diff files in activated scenarios only; no match → ⏭️ |
| P6 | Performance & Memory | Diff components only |
| P7 | a11y/i18n/UX Interaction | Diff components only; 6 static UX rules (confirm/anti-duplicate/error-wrapping, etc.) |
| P8 | Async Tasks | No queue files in diff → ⏭️ skip |
| P9 | AI Reviewer | Diff + UX path simulation + confidence filter + dual-dimension output |
| P10 | Report | Tech/UX dual panels + status column + skip reasons |
| P11 | On-demand Fix | Allowlist/Blocklist + compile check + re-validation |

### 8 Business Scenarios (incl. AI Application Security)

| Scenario | Detection | Key Checks |
|----------|-----------|------------|
| 💰 Transaction | `pay`/`order` + amount fields | Idempotency, signature, reconciliation, refund |
| 📅 Resource Contention | `capacity`/`slot`/`stock` | Conflict validation, concurrency lock, timeout release |
| 🏠 Location-based | `lat`/`lng`/`geo` | GPS battery, check-in verification, service range |
| 🎫 Benefits | `coupon`/`points`/`voucher` | Concurrent redemption, pre-validation, stacking precision |
| 💬 Info Exchange | Bidirectional ops | Desensitization, permission notice, mutual confirmation |
| ⏳ Async Long-task | State machine + queue/task entities | Progress persistence, timeout, recovery |
| 📨 Message Queue | MQ deps + Queue decorators | Idempotency, DLQ, transactional messages |
| 🤖 AI Application Security | LLM SDK/Vector DB/RAG framework | Fallback chain, prompt injection, output safety (P2.7 on-demand) |

### Dual-Branch Review

```
🔒 Tech Risk: Concurrency/Transaction/Idempotency/Data Integrity/Signature
🧑 UX Risk:  State Persistence/Error Classification/Weak-network Fallback/Dedup/Pre-validation
```

### UX Enhancement (v6.1)

- **6 Static UX Rules**: delete confirmation, submit anti-duplicate, error wrapping, empty state, native alert, input hints
- **AI Path Simulation**: P9 simulates user journeys to catch cross-step UX breakdowns
- **Asset-loss Escalation**: UX flaws in payment/refund/redemption can escalate to 🔴Critical (ux_derived_asset_loss)
- **Dual-Panel Report**: tech faults and user complaints displayed separately

### AI Application Security (v6.2)

P2.7 auto-activates when LLM SDK/vector DB/RAG framework deps detected:
- **Fallback chain**: try-catch + preset responses for LLM failures
- **Output safety**: format validation + XSS sanitization for LLM-generated content
- **Prompt injection**: user input sanitization before prompt assembly
- **Token management**: topK/score/maxTokens limits for retrieval context

### Audit Enhancement (v6.3)

9 fixes from a deep security audit, focused on trustworthiness & coverage:
- **🤖 AI Conclusion Downgrade**: AI-only findings tagged `🤖【AI推断-待确认】` to visually distinguish from static rules; asset-loss escalation threshold 0.9→0.95
- **~ Medium Transparency**: P0.5 Medium-confidence scenarios prefixed with `~` in reports, annotated "P0.5 inference — verify"
- **📎 Related File Hints**: P1 now checks same-directory Controller↔Service pairs, warns when counterpart not in diff
- **🔧 P11 Implementation Details**: temp directory structure + 6-step write flow + 5-scenario rollback strategy
- **📋 Framework Mismatch Hint**: auto-detects non-React/NestJS projects and suggests alternative skills

---

## 🛡️ Two-Layer Defense

```
① Local pre-commit (Incremental)
   git add . → hermes review
   Diff-only → Fast → Skip if context missing
   Goal: Incremental pre-flight, keep development fluid

② CI pipeline (Full Baseline)
   hermes review --full
   Full-repo scan → Slower but complete → No degradation allowed
   Goal: Final merge gate
```

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
| `--full` | Full baseline mode (CI) | `hermes review --full` |
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
┌──────────────────┬────┬────┬────┬────┬──────┬──────────┬─────────────────────────┐
│ Dimension         │ 🔴 │ 🟠 │ 🟡 │ 🔵 │ 🚨UX │ Status   │ Skip Reason              │
├──────────────────┼────┼────┼────┼────┼──────┼──────────┼─────────────────────────┤
│ Security P2       │  2 │  1 │  0 │  0 │   0  │ ✅ Full   │                         │
│ Route Align P4    │  — │  — │  — │  — │   —  │ ⏭️ Skip   │ Backend Controller not   │
│                  │    │    │    │    │      │          │ in diff                 │
│ Auth P4.6         │  0 │  1 │  0 │  0 │   0  │ ⚠️ Partial │ Self-check done; cross   │
│                  │    │    │    │    │      │          │ comparison skipped       │
│ Transaction P5.1  │  0 │  2 │  1 │  0 │   3  │ ✅ Full   │                         │
│ Resource P5.3     │  — │  — │  — │  — │   —  │ ⏭️ Skip   │ Not detected by P0.5    │
│ Async Task P8     │  — │  — │  — │  — │   —  │ ⏭️ Skip   │ No queue code in diff    │
│ ...               │    │    │    │    │      │          │                         │
├──────────────────┼────┼────┼────┼────┼──────┼──────────┼─────────────────────────┤
│ Total             │  3 │  8 │  5 │  1 │   5  │ 4 skipped │                         │
└──────────────────┴────┴────┴────┴────┴──────┴──────────┴─────────────────────────┘

🔧 Auto-fixable: 8 issues | ⚠️ Manual review: 6 issues
→ Reply "fix" to start auto-fix
```

### Status Types

| Status | Meaning | Trigger |
|--------|---------|---------|
| ✅ Full | All checks complete | All within diff scope |
| ⚠️ Partial | Partial execution | P4.6 only: self-check done, comparison skipped |
| ⏭️ Skip | Context insufficient | Reference files not in diff / P0.5 not activated |

Each issue carries four labels:

| Label | Values |
|-------|--------|
| Severity | `Critical` / `High` / `Medium` / `Info` |
| Experience Risk | `high_complaint` / `medium_complaint` / `no_risk` |
| Fix Priority | `block` / `this_iteration` / `next_iteration` / `optional` |
| Source | `【Static Rule】` / `【AI Review】` |

---

## ❓ FAQ

<details>
<summary><b>I only changed 2 CSS files. Will it scan the whole repo?</b></summary>

No. Local mode follows the **Diff-Only Principle** — only `git add`'ed files are reviewed. 2 CSS files → only P1/P2/P3/P10 run, results in seconds. P4/P5/P8 and other irrelevant modules are ⏭️ skipped.

</details>

<details>
<summary><b>What if the backend Controller isn't in the diff?</b></summary>

Route alignment marks `⏭️ Skip: Backend Controller not in diff, cannot compare`. No full-repo scan. This check is enforced in CI `--full` mode.

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

- [ ] v7: Historical baseline comparison (store reports, track defect trends)
- [ ] v7: CI/CD native integration plugins (GitHub Actions / GitLab CI)
- [ ] v7: Custom rule extension API
- [ ] v7: Fast Path intelligent path detection (replace hardcoded high-risk list with import-frequency-based auto-detection)
- [ ] v7: `hermes review --list-suppressions` command to view suppressed alerts

---

## 📄 License

MIT © 2026
