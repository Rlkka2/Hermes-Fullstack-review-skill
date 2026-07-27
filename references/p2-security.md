## 前置依赖

本文件规则依赖以下概念，如未加载请先查阅：

| 依赖 | 位置 |
|------|------|
| 术语定义 | SKILL.md 术语规范 + Glossary |
| 去重机制 | SKILL.md 全局去重 |
| 严重度框架 | SKILL.md |

---


## P2：安全扫描（Security Scan）

### P2.1 硬编码密钥/注入漏洞

针对本次 diff 新增行执行以下扫描：

```bash
# 硬编码密钥/密码
grep "^+" | grep -iE "(api_key|secret|password|token|passwd|jwt_secret)\s*[:=]\s*['\"][^'\"]{6,}['\"]"

# SQL 注入（拼接查询）
grep "^+" | grep -E "\.execute\(\s*f['\"]|\.query\(\s*f['\"]|\$queryRaw\s*\(\s*`"

# XSS（前端）
grep "^+" | grep -E "innerHTML\s*=|dangerouslySetInnerHTML|document\.write\("

# 路径遍历
grep "^+" | grep -E "\.\./|path\.join\(.*req\.|path\.resolve\(.*req\."

# eval / exec / Function 构造器
grep "^+" | grep -E "\beval\(|\bexec\(|new Function\(|Function\(.*return"
```

### P2.2 NestJS 专项安全

- `@Query()` / `@Param()` 直接拼入 SQL 或 shell 命令（未经验证）
- Guard 缺失：敏感接口（支付/退款/核销/删除）未使用 `@UseGuards(AuthGuard)` 或自定义权限守卫
- DTO 未使用 `class-validator` 装饰器做输入白名单校验

### P2.3 React 前端专项安全

- `dangerouslySetInnerHTML` 使用但未经过 `DOMPurify.sanitize()`
- `localStorage` 直接存储敏感信息（token、用户手机号）
- 第三方 `<script>` 标签无 `integrity` 校验

### P2.4 Dependency Security（依赖 CVE 扫描）— 新增

> **两步流**：①跑全量 `npm audit`（获取项目级漏洞数据）→ ②后处理：仅保留 diff 中**新增/升级**包的结果。不扫全量则会漏报已知漏洞的存量依赖风险。

```
触发条件：git diff --cached 中包含 package.json / package-lock.json 变更
扫描范围：全量 npm audit 获取漏洞数据 → 按 diff 中的包名/版本过滤，仅保留新增或变更的依赖包
报告级别：仅报告 Critical / High 级别 CVE
严重度：🔴Critical（生产环境已知高危漏洞）
```

命令示例：
```bash
# 第一步：获取全量漏洞数据
npm audit --json --only=prod 2>/dev/null | \
  jq '.vulnerabilities | to_entries | map(select(.value.severity == "critical" or .value.severity == "high"))'

# 第二步：交叉对比 diff 中新增/升级的包名，过滤不相关结果
# diff 包名通过 git diff --cached package.json | grep '"@\|"version"' 提取
# 仅当 npm audit 结果的包名匹配 diff 包名时才纳入 P10 报表
```

> **注意**：npm audit 扫描的是当前 `node_modules`，如果本地未执行 `npm install` 可能导致扫描结果与代码声明不一致。此检查在 CI `--full` 模式下按全量结果输出，不做 diff 过滤。

### P2.5 Log Desensitization（日志敏感信息脱敏）— 新增

检测日志输出中是否包含敏感信息：

```
前端：
  console.log 中包含 phone/token/password/idCard/openid 等字段 → 🟠High
后端：
  Logger.log / console.log 中包含手机号/身份证/密码/密钥/accessToken → 🟠High
```

规则：禁止在任何日志（包括 `console.log`、`Logger.log`、`logger.debug`）中直接打印以下类型数据：
- 手机号（完整）
- 身份证号
- 密码/密钥/Token
- 银行卡号
- 用户地址（完整）
- 微信 openid/unionid

### P2.6 Env Config Isolation（环境配置隔离）— 新增

检测环境相关硬编码：

| 检查项 | 严重度 | 说明 |
|-------|-------|------|
| 硬编码线上域名/IP | 🔴Critical | 代码中出现 `https://api.prod.com` / `https://xxx.com` 等非 `localhost` 的完整 URL |
| 第三方密钥硬编码 | 🔴Critical | `wxAppId = 'wx123...'` / `secret = 'sk-xxx'` 直接写在代码中 |
| 测试开关硬编码 | 🟠High | `MOCK = true` / `DEBUG = true` / `BYPASS_AUTH = true` 等未通过 `process.env` 控制 |

### P2.7 AI Application Security（AI 应用安全与可靠性）— 按需激活

要求 P0.5 检测到 `ai_application` 行为特征。不激活则完全跳过此模块。

> **覆盖范围**：P2.5（日志脱敏）同步覆盖 AI 调用路径——LLM 调用时用户问题可能包含个人信息，禁止在日志中原样输出用户 prompt。

#### 规则清单

| # | 规则 | 严重度 | 检测内容 | 说明 |
|---|------|:---:|---------|------|
| 1a | **LLM 降级兜底** | 🔴Critical | LLM 调用是否有 `try-catch` 包裹；`catch` 分支是否有预设话术常量，而非直接 `throw error` | AI 调用失败后用户看到的应是"我需要进一步核实"而非技术错误堆栈 |
| 1b | **完整四层降级链** | 🟡Medium | 知识库命中→直接返回 / 未命中→LLM兜底 / 输出校验 / 预设话术兜底，四层是否完整 | 含运行时行为，静态代码仅能做部分检测。初版可选 |
| 2 | **LLM 输出安全渲染** | 🔴Critical | **格式层**：`JSON.parse()` 是否有 `try-catch` 包裹（LLM 可能返回非标准 JSON）；**安全层**：LLM 输出渲染前是否经过 `DOMPurify.sanitize()` 或使用 `textContent`。原规则2+5合并，同源扫描，dedup 合并 | AI 应用前端崩溃+新型 XSS 的一体两面 |
| 3 | **输出可信度兜底** | 🟠High | RAG 场景下 LLM 回答是否附带 `source`/`citations`/`references` 字段；前端是否展示来源引用 | 初版可选，标记建议不强制阻断 |
| 4 | **Prompt 注入防线** | 🔴Critical | 用户输入拼入 prompt 模板前是否经过 `sanitize`/`replace`/关键词过滤；是否有系统 prompt 和用户 prompt 的分隔标记 | **仅在 diff 包含 prompt 模板文件时触发**。若 sanitize 逻辑在 diff 外 → ⏭️ 跳过，标注原因，同 P4.6 降级口径 |
| 5 | **Token 管理** | 🟠High | 检索结果是否有 `topK` 限制和相似度阈值 `score` 过滤；拼接后的上下文是否有 `maxTokens` 截断；是否使用 tokenizer 计数而非字符数估算 | 无限制拼接→上下文过长稀释关键信息+超时风险 |

#### 与其他阶段的边界

| 其他阶段 | 与 P2.7 的关系 |
|---------|---------------|
| P2.3 React 前端安全 | P2.3 检测开发者代码中的 XSS（硬编码 `dangerouslySetInnerHTML`）；P2.7 检测 **LLM 动态生成内容**的渲染安全。互补，不重复 |
| P2.5 日志脱敏 | P2.5 同步覆盖 AI 调用路径——用户 prompt 可能含个人信息，禁止原样输出到日志 |
| P4.4 错误体系 | P4.4 管通用接口错误的友好封装；P2.7 专门管 LLM 调用失败时的降级话术。场景不同 |
| P8 异步长任务 | P8 管任务的进度/超时/恢复；P2.7 管 LLM 同步调用失败时的降级链 |

#### 不纳入的理由（备查）

| 想法 | 不纳入原因 |
|------|-----------|
| Token 成本监控 | RAG 架构下仅兜底走 LLM，成本可控 |
| LLM 调用频率限制 | 属于业务策略非安全基线，前端防重点击 P7 已覆盖 |
| Prompt 版本管理 | 属于工程实践建议，不属于检查规则 |

---
