## 前置依赖

本文件规则依赖以下概念，如未加载请先查阅：

| 依赖 | 位置 |
|------|------|
| 术语定义 | SKILL.md 术语规范 + Glossary |

---


## P0：项目类型检测（Project Detection）

### 目标

自动识别项目使用的框架和覆盖的端，输出给用户确认。

### Monorepo 检测（新增）

扫描仓库根目录及 `packages/`、`apps/`、`server/`、`client/`、`frontend/`、`backend/` 等常见 monorepo 子包目录：

1. 若仓库根目录有 `package.json` 且包含 workspaces 配置 → **Monorepo 项目**
2. 分别进入各子包目录读取各自的 `package.json`
3. 分别识别前端框架（React/Taro/RN/uni-app）和后端框架（NestJS/Express）
4. 前端审查规则只应用于前端子包，后端审查规则只应用于后端子包

### 检测逻辑

读取 `package.json`，分析 `dependencies` 和 `devDependencies`：

| 依赖特征 | 推断框架 | 推断覆盖端 |
|---------|---------|-----------|
| `@tarojs/taro` | Taro 跨端框架 | Web, H5, App, Mini Program |
| `uni-app` / `@dcloudio/uni-app` | uni-app 跨端框架 | Web, H5, App, Mini Program |
| `react-native` | React Native | App |
| `react` + `react-dom`（无 Taro/RN） | React Web | Web, H5 |
| `@nestjs/core` | NestJS 后端 | N/A（后端不区分端） |
| `next` | Next.js | Web（SSR/SSG） |

### 输出

Monorepo 项目示例：
```
项目检测结果：
  架构: Monorepo
  ├── packages/client   → React Web (Web, H5)
  ├── packages/admin    → React Web (Web)
  └── packages/server   → NestJS

推断覆盖端：Web ✅ | H5 ✅ | Mini Program ⏭️

是否准确？可回复修正，或使用 --platforms=web,h5,mini 覆写。
```

> **框架不匹配提示**：若检测到的技术栈非 React/NestJS/Taro/uni-app/Next.js（如 Express、纯 Vite、Fastify 等），输出 `⚠️ 当前项目框架（{检测结果}）不在本 Skill 的深度定制范围内，部分 NestJS/React 专属规则（P2.2/P2.3/P4.2）将跳过。建议配合 requesting-code-review 使用。`

### 交互参数

- `--platforms=web,h5,rn,mini`：手动指定本次审查覆盖的端，覆盖自动推断结果
- 场景：本次提交只改了小程序代码，指定 `--platforms=mini` 可跳过 Web/H5/App 规则，减少扫描耗时

> 确认后进入 P0.5。

---

## P0.5：业务行为识别（Behavioral Pattern Detection）

### 目标

不依赖文件名关键词，通过分析代码**行为特征**推断项目包含哪些服务业场景，输出预激活矩阵。

### 检测方法

**仅解析 diff 内变更文件推断业务场景**，不遍历项目全库。

扫描 diff 文件中 NestJS Controller/Service 的方法签名、Prisma/TypeORM Model 字段类型、前端 API 调用参数，以及本文档「场景标识枚举表」中各场景的依赖特征，组合判断。

> **若无业务特征**：diff 中无任何业务相关代码（如仅修改通用工具函数 `formatDate.ts`）→ 所有场景未激活 → P5 整体跳过，报表标注 `⏭️ P5 全部场景未激活（diff中未检测业务特征源码）`。

| 行为特征 | 检测信号（满足2+即命中） | 激活的场景规则集 |
|---------|----------------------|----------------|
| Transaction（交易行为） | Controller 含 `pay`/`order` 状态变更方法；Model 含 `amount`/`paid`/`refund`/`payment_status` 字段；存在 `*pay*.service` 或 `*payment*` 模块 | `transaction` |
| Resource Contention（资源抢占行为） | Model 含 `capacity`/`quota`/`slot`/`stock` 字段；存在 `checkAvailability` / `isSlotAvailable` 类方法；存在预约类实体 | `resource_contention` |
| Location-based（地理位置行为） | Model 含 `lat`/`lng`/`latitude`/`longitude`/`geo` 字段；存在签到/范围校验逻辑；前端含地图组件调用 | `location_based` |
| Benefits（权益体系行为） | Model 含 `coupon`/`points`/`voucher`/`discount`/`reward` 实体；存在核销/兑换逻辑 | `benefits` |
| Async Long-task（异步长任务行为） | Model 含状态机 `pending→processing→completed→failed` 流转；存在 `queue`/`task`/`job`/`process` 实体；使用 Bull/BullMQ/Agenda 等队列库 | `async_long_task` |
| Info Exchange（信息交换行为） | 存在用户间消息/查看/联系方式交换等双向操作；存在聊天/私信/咨询模块 | `info_exchange` |
| Message Queue（消息队列行为） | 依赖包：`bull`/`bullmq`/`amqplib`/`kafkajs`/`@nestjs/bull`/`@nestjs/microservices`；代码特征：`@Queue()`/`@Process()`/`@MessagePattern()` 装饰器、`sendQueue`/`publish`/`consume`/`emit` 方法调用 | `message_queue` |
| AI Application（AI 应用行为） | 依赖包：`openai`/`langchain`/`@langchain/core`/`anthropic`/`llamaindex`/`@azure/openai`/`chromadb`/`pinecone-client`/`weaviate-client`/`pgvector`/`tiktoken`/`gpt-tokenizer`；代码特征：`chatWithAI`/`generateAnswer`/`askLLM`/`RAG`/`vectorStore`/`embedding` 等函数调用 | `ai_application` |

### 补充说明（MQ 与 AI Application 激活逻辑）

> **MQ 双重判定逻辑**：①检测项目是否引入 `bull` / `bullmq` / `amqplib` / `kafkajs` / `@nestjs/bull` 等 MQ 依赖包；②代码中是否存在 `@Queue()` 装饰器、`sendQueue` / `publish` / `consume` 等业务方法调用。满足**任意一类**即提升置信度，两类同时命中判定为 **High 置信自动激活**，无需人工确认。
>
> **AI Application 激活逻辑**：检测到 LLM SDK（openai/langchain 等）或向量数据库（chromadb/pinecone 等）或 RAG 框架（+ tokenizer）中**任意一项** → Medium 置信度激活；满足**两项及以上** → High 置信自动激活。

### 置信度标记（Confidence Marking）

| 置信度 | 条件 | 处理方式 |
|-------|------|---------|
| **High** | 检测到 3+ 特征信号 | 自动激活该场景规则集 |
| **Medium** | 检测到 2 个特征信号 | 展示给用户确认，默认激活 |
| **Low** | 检测到 1 个特征信号 | **⚠️ 仅提示，强制要求用户交互确认，不确认则流水线暂停。不允许直接跳过** |

### 输出

```
场景激活矩阵：
  交易场景 (transaction)               → ✅ High 置信度，已激活
  资源抢占 (resource_contention)        → ~⚠️ Medium 置信度，已激活（P0.5推断，建议核实）
  线下履约 (location_based)             → ⏭️ 未检测到
  权益场景 (benefits)                   → ⏭️ 未检测到
  异步长任务 (async_long_task)          → 🔍 Low 置信度，需要你确认是否激活 [Y/n]
  信息交互 (info_exchange)              → ⏭️ 未检测到
  消息队列 (message_queue)              → ✅ High 置信度，已激活

回复 --include=transaction,async_long_task --exclude=benefits 可手动覆写。
Low 置信度场景必须明确回复 Y/n 后才能继续。
```

### 重要约束

- 识别结果仅作为**预激活矩阵（建议）**，不是强制开关
- 用户可随时通过 `--include` / `--exclude` 参数手动覆写
- **Low 置信度场景必须交互确认，不能跳过**
- **Medium 置信度场景自动激活但标注 `~` 前缀**：表示"P0.5 推断结果，非确定激活"。在 P10 报表中，Medium 激活的场景名加 `~` 前缀（如 `~资源抢占 P5.3`），表头注释 `~ 标记为P0.5推断，建议核实`，提醒开发者可能存在误激活
- 多层封装（公共 Service、抽象基类、纯前端场景）可能导致漏检，依赖人工覆写补充

> 确认后进入 P1。

---
