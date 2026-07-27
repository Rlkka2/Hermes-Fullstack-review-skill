## 前置依赖

本文件规则依赖以下概念，如未加载请先查阅：

| 依赖 | 位置 |
|------|------|
| 项目检测结果（P0覆盖端） | references/p0-project-detection.md |
| 术语定义 | SKILL.md Glossary |

---


## P3：平台适配 + 跨端体验一致性（Platform Adaptation & Cross-platform UX）

### 3.1 API 可用性检查（根据 P0 覆盖端）

| 检查项 | 影响端 | 检测方式 |
|-------|-------|---------|
| `window` / `document` 全局对象直接引用 | 小程序、App | grep `window\.` `document\.` 在组件文件中 |
| `localStorage` / `sessionStorage` 直接调用 | 小程序 | grep 后对比 `@tarojs/taro` 的 `Taro.setStorage` |
| `navigator.geolocation` 直接调用 | 小程序 | 应使用 `Taro.getLocation` |
| `fetch` / `axios` 无超时设置 | H5、小程序 | 弱网环境默认超时过长 |

### 3.2 条件编译遗漏

Taro 项目中，平台差异代码必须包裹条件编译。检测：若代码中存在明显的平台特定 API 但无对应条件编译包裹，标记 🟠High。

### 3.3 跨端体验一致性

**仅审查 diff 文件内的跨端一致性问题。**

服务业高频跨端差异事故：

| 检查项 | 规则 |
|-------|------|
| 文件上传大小限制 | 四端 `maxFileSize` 阈值是否统一（同一接口同一定义） |
| 图片压缩比例 | Web/H5/App/小程序压缩比是否一致 |
| 表单输入长度限制 | `maxLength` 四端是否统一 |
| 授权弹窗逻辑 | 位置/相机/相册权限请求流程四端是否一致 |

**核心规则**：相同业务行为，四端交互逻辑允许降级，但**不允许出现规则冲突**；降级场景必须显式提示用户。

---
