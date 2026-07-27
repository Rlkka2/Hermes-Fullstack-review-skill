## 前置依赖

本文件规则依赖以下概念，如未加载请先查阅：

| 依赖 | 位置 |
|------|------|
| Diff-Only 原则 | SKILL.md |
| 暂存区读取规范 | SKILL.md |
| 非文本过滤 | SKILL.md |
| Fast Path 概念 | SKILL.md（六档权重链+优先级） |

---


## P1：变更扫描（Change Scan）

### 扫描内容

1. **代码变更**：`git diff --cached` 获取本次提交涉及的所有文件
2. **文案资源变更**：diff 中涉及的文案文件，但**仅限以下后缀的文件**纳入文案扫描：`.tsx` `.ts` `.jsx` `.js` `.json`（i18n 文件）。`.md`、测试文件、注释文件不纳入文案扫描
3. **删除文件风险识别**：本次 diff 中删除的文件，若为业务源码且文件名匹配以下关键词，自动提权至 🔴Critical + 🚨high_complaint：
   - `pay` / `payment` / `callback` / `notify` / `refund` / `transaction`
   - `order` / `booking` / `reservation` / `appointment`
   - `cron` / `schedule` / `job` / `task`
   - `lock` / `mutex` / `semaphore`
   - `coupon` / `points` / `voucher`

### 删除文件风险排除（路径匹配优先级）

路径匹配规则：**任一目录层级**（非仅根目录）匹配以下模式即排除，不触发提权：

- `test/` / `tests/` / `__tests__/`
- `docs/` / `documentation/`
- `*.test.*` / `*.spec.*`
- `*.example.*` / `*.template.*` / `*.sample.*`
- 静态资源（`*.svg`、`*.png`、`*.jpg`、`*.gif`）

> 例如 `src/modules/payment/__tests__/payment.service.ts` 删除不触发风险。
> 例如 `src/modules/payment/test/payment.service.ts` 删除不触发风险。
> 例如 `src/modules/payment/payment.service.ts` 删除触发 🔴Critical。

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

## Fast Path 六档分流（Change Type Prejudgment）

P1 之后根据变更文件类型执行分流判定，避免"改了 2 个 CSS 文件跑全套流水线"。

### 分流规则

| 档位 | 命中条件 | 执行范围 | 跳过 |
|------|---------|---------|------|
| 🟢 纯非代码 | 仅 `.md` `.txt` `.lock` `.gitignore` 等 | P0 → P1 → P2(仅2.5/2.6) → P10 | P0.5/P3~P9/P11 |
| 🟢 前端样式 | 仅 `.css` `.scss` `.less` `.stylus` | P0 → P1 → 基础样式合规 → P10 | P0.5/P3~P9/P11 |
| 🟢 前端工具 | 仅 `utils/` 通用组件，无路由/权限/状态/业务页面 | P2安全 → P6组件 → P7(仅7.6:删除确认/防重/错误透传) → P10 | P0.5/P3/P4/P4.5/P4.6/P5/P7其余/P8/P9 |
| 🟢 后端工具 | 仅 `.ts`，无 Controller/Service/DB 特征 | P2安全 → P10 | P0.5/P3/P4/P4.5/P4.6/P5/P8/P9 |
| 🔴 高风险路径 | 命中前后端默认名单 | **叠加**当前档位 + 强制 P2 + P9 | — |
| 🟡 全栈变更 | 含业务代码（前后端同时或任一含业务特征） | P0~P11 完整 | 无 |

### 优先级规则

1. **就高不就低**：变更中包含更高权重文件 → 升级至对应路径，不按占比多数判定
2. **高风险路径最高优先级**：命中高风险名单即强制升级。行为是**叠加**（当前档位基础检查 + 强制 P2 + P9），不是替换。例如前端工具文件命中高风险路径 → 执行 P2 + P6 + P9
3. **CI `--full` 强制禁用 Fast Path**：全量模式不执行任何短路，必须完整扫描

**权重**：纯非代码 < 前端样式 < 前端工具/后端工具 < 全栈 < 高风险路径

---

## 同目录关联文件推断（Same-directory Related File Inference）— v6.3 新增

> 解决 Diff-Only 的盲区：改了 Controller 但没改对应的 Service，本地 pre-commit 静默通过 → CI 才发现问题。

### 规则

P1 扫描 diff 文件列表后，对每个后端文件执行**零成本同目录推断**：

```
输入：diff 文件路径（如 src/modules/payment/payment.controller.ts）
操作：检查同目录下是否存在关联文件且不在 diff 中
关联规则：
  X.controller.ts  →  X.service.ts
  X.service.ts     →  X.controller.ts
  X.resolver.ts    →  X.service.ts
  X.gateway.ts     →  X.service.ts
输出：不在 diff → ⚠️ 标注，非阻断
```

### 输出

在 P1 结束后追加关联文件提示：

```
📎 关联文件检查（同目录推断）：
  ⚠️ payment.controller.ts 已修改，但同目录 payment.service.ts 不在本次变更中，建议一并审查
  ⚠️ auth.service.ts 已修改，但同目录 auth.controller.ts 不在本次变更中，建议一并审查
  ✅ order.controller.ts + order.service.ts 均在本次变更中

> 提示：以上仅做同目录文件名推断，不跨目录追踪。跨端关联（前端 ↔ 后端）将在 CI --full 模式补全。
```

### 重要约束

- **不具阻断性**：仅提示，不产生 Critical/High 告警，不进入 P10 缺陷统计
- **仅限同目录**：不跨目录解析 import 链或路由映射（性能约束）
- **空 diff / 纯非代码档位**：不触发此检查

---

## 高风险底层路径名单（High-Risk Path List）

修改底层通用基础设施文件，影响面广但 P0.5 难以识别业务特征。命中后强制激活 P2 + P9。

### 后端默认名单

| 路径特征 | 风险说明 |
|---------|---------|
| `auth` / `guards` / `middleware` | 鉴权/中间件改动影响所有接口 |
| `request` / `interceptor` | 全局请求拦截器 |
| `crypto` / `encrypt` | 加密/签名逻辑 |
| `database` / `prisma` / `typeorm` 基类 | 数据库基础层 |

### 前端默认名单

| 路径特征 | 风险说明 |
|---------|---------|
| `src/router/` | 路由配置、路由守卫 |
| `src/utils/request` / `src/api/` 基类 | 全局请求拦截器 |
| `src/permission/` | 全局权限控制 |
| `src/store/` / `src/stores/` | 全局状态管理入口 |
| `src/middleware/` | 前端中间件 |

### 配置与行为

可在 `.hermes/review-config.json` 中覆盖或扩展：

```json
{
  "high_risk_paths": {
    "backend": ["auth", "guards", "request", "crypto"],
    "frontend": ["src/router/", "src/permission/", "src/store/"]
  }
}
```

命中后行为：
- 不激活 P5 业务场景（业务场景不相关）
- 强制 P2 安全基础扫描
- 强制 P9 AI 通用安全评审
- **叠加** 当前档位已有检查项（不替换）

---
