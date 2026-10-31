# 012 — 构建工作流（building-workflow）跨工程功能规格

> **Module:** building-workflow ｜ **Status:** Implemented（Desktop / Android） ｜ **Last Updated:** 2026-10-31
> **上游基线:** `pi` v1.2.0 / `pi-web` v0.12.1
> **适用范围:** **`pi-desktop`** 与 **`pi-desktop-android`**。
> **不适用:** **鸿蒙工程暂无此模块** —— `pi-desktop-hos` / `pi-desktop-ark` 使用各自本地 hvigor 构建
> （`collect-runtime` / `assemble-payload` / 签名 / 真机），不建 `012`，其构建说明见子工程 `docs/` 与 `specs/`。
> **父/子分工:** 本文写跨工程构建要求；具体命令、脚本、workflow 写在子工程 `012-building-workflow/`。

---

## 1. 模块概述

### 1.1 目的

让桌面与 Android 的构建、打包、发布**可复现、可门禁、可追溯**：

- 同一份源码 + 上游 tag + 运行时归档 → 等价产物；
- 每一步失败都显式阻断，不产出半成品；
- 产物能追溯到 commit、workflow run、哈希与签名状态。

### 1.2 解决的问题

- 手工构建依赖本机隐式状态（已安装依赖、缓存、旧产物）。
- 上游 tag 漂移导致“昨天能构建、今天不能”。
- 构建产物未经验证就发布。
- 密钥/签名材料误入仓库。
- 发布产物无法追溯到具体 commit / run。

### 1.3 范围

**包含**

- 桌面：npm / Forge / Vite 构建，三平台 CI，tag 发布。
- Android：载荷组装、引擎门禁、Gradle 构建、签名 APK、GitHub Release。
- 通用：上游钉住、清理、门禁、产物命名、校验和、证据。
- 与 `011`（版本）和 `013`（品牌）的衔接。

**不包含**

- 鸿蒙两版的构建（本模块不适用）。
- 运行时包本身的构建（由 `ohos-node-bridge` / `aosp-node-runtime` 仓库负责）。
- 应用商店上架流程（子工程 107）。
- 代码签名证书的申请与保管。

---

## 2. 用户故事

- 作为发布者，我推一个 tag 就能得到可安装的产物与校验和。
- 作为维护者，我在干净机器上按文档能复现同一产物。
- 作为审核者，我能从 Release 追到 commit、run id、上游 tag 与运行时版本。
- 作为开发者，构建失败时我能立刻知道是哪一步、为什么、怎么修。
- 作为安全负责人，我能确认仓库里没有签名材料与密钥。

---

## 3. 功能需求

### 3.1 可复现

- **FR-012-001**（MUST）：构建必须从**清理后的工作树**开始；清理清单必须文档化。
- **FR-012-002**（MUST）：构建只依赖源码 + 上游 tag + 运行时归档 + 锁文件；不得依赖本机缓存或未入库文件。
- **FR-012-003**（MUST）：依赖安装必须使用锁文件（`npm ci` / Gradle 锁 / 版本表）。
- **FR-012-004**（MUST）：相同输入必须产出等价产物（签名材料与时间戳除外）。
- **FR-012-005**（MUST）：构建脚本必须跨平台（Windows / macOS / Linux）或明确平台限制。

### 3.2 上游与运行时钉住

- **FR-012-006**（MUST）：上游必须按 tag 检出，并断言 commit SHA；不一致即失败。
- **FR-012-007**（MUST）：运行时版本必须来自锁文件（`package.json` / `runtime.lock.json` / `libs.versions.toml`）。
- **FR-012-008**（MUST）：Android 引擎必须通过 SHA-256 + ELF 六项门禁后才允许打包。
- **FR-012-009**（MUST）：补丁集必须按 `patches/<pi-web-tag>/` 应用，幂等且可逆。

### 3.3 门禁

- **FR-012-010**（MUST）：桌面必须通过 TypeScript 类型检查与 `smoke-desktop` 四项断言（SPA / 带凭据 API / SSE / AgentSession）。
- **FR-012-011**（MUST）：原生依赖必须做完整性检查（如 `node-pty` 预构建不能是空壳）。
- **FR-012-012**（MUST）：Android 必须通过 `verify-runtime` 与 `compliance-check`。
- **FR-012-013**（MUST）：品牌化必须做 0 残留终检（见 `013`）。
- **FR-012-014**（MUST）：任一门禁失败必须非零退出，阻断打包/发布。

### 3.4 产物

- **FR-012-015**（MUST）：桌面产物必须包含安装包/归档、图标、`pi-dist.tar.gz` + marker；Forge makers 配置必须可复现。
- **FR-012-016**（MUST）：Android 产物必须为签名 APK（secrets 齐全）或 debug 兜底（明确标注）。
- **FR-012-017**（MUST）：产物必须带 SHA-256 校验和。
- **FR-012-018**（MUST）：产物命名必须包含版本与平台/架构，便于区分。
- **FR-012-019**（MUST）：不得把构建产物、日志、`node_modules` / `oh_modules`、密钥提交入库。

### 3.5 CI / 发布

- **FR-012-020**（MUST）：桌面 CI 必须覆盖 Windows / macOS / Linux 三平台，各自构建。
- **FR-012-021**（MUST）：发布必须由 tag（`v*`）触发；`workflow_dispatch` 只做构建演练，不发布。
- **FR-012-022**（MUST）：发布前必须校验 tag 与 `package.json.version`（桌面）/发布版本一致。
- **FR-012-023**（MUST）：签名材料只能通过 secrets 注入；缺失时签名构建必须显式失败或降级为 debug 并标注。
- **FR-012-024**（MUST）：Release 说明必须包含上游 tag、运行时版本、补丁数、run id。
- **FR-012-025**（MUST）：发布产物必须可从 Release 下载并校验哈希。

### 3.6 证据

- **FR-012-026**（MUST）：每次发布必须保留构建日志/run id/产物哈希。
- **FR-012-027**（MUST）：真机/安装验证记录必须与产物版本对应。
- **FR-012-028**（MUST）：历史发布记录不得回改。

---

## 4. 关键实体

| 实体 | 说明 | 关键属性 |
|---|---|---|
| 构建输入 | 源码 + 上游 + 运行时 | tag、SHA、锁文件 |
| 门禁 | 阻断条件 | 类型、smoke、引擎、品牌、合规 |
| 产物 | 安装包 / HAP / APK | 版本、平台、哈希 |
| 发布 | Release | tag、run id、附件、说明 |
| 证据 | 日志/记录 | 时间、设备、结果 |

---

## 5. 验收场景

### 场景：干净构建

- Given 清理后的工作树与正确锁文件
- When 按文档执行桌面/Android 构建
- Then 产物生成，门禁全绿，哈希可复现

### 场景：上游漂移

- Given 上游 tag 被重打，commit 变化
- When 构建
- Then SHA 断言失败，构建停止

### 场景：引擎门禁

- Given 篡改的 AAR
- When Android 构建
- Then `verify-runtime` 失败，不产出 APK

### 场景：发布

- Given 推送 tag `v0.12.1`
- When CI 完成
- Then Release 含产物 + `.sha256`，说明含上游/运行时/补丁/run id

### 场景：密钥保护

- Given 仓库历史与工作树
- When 扫描签名/密钥
- Then 无 keystore、口令、API Key 入库

---

## 6. 非功能需求

- **耗时**：Android 全量构建 ≤ 90 min（CI timeout）；桌面三平台并行。
- **稳定性**：构建失败可重试；网络依赖有重试/镜像。
- **可读性**：日志分阶段，错误含修法。
- **安全**：最小权限 token；Release 只上传必要文件。
- **可维护**：脚本集中、参数化、无隐式全局状态。

---

## 7. 假设与约束

- 桌面 CI 可访问 GitHub 上游；Android CI 可访问 Maven Central。
- 签名 secrets 由发布者配置；缺失时允许 debug 兜底（Android）。
- 鸿蒙两版不在本模块范围。
- macOS 公证/Windows 签名当前未启用（`011` / 发布文档登记）。

---

## 8. 依赖

- 父工程：`011-version-0.12.1`、`013-branding`、`constitution.md`。
- 子工程：`pi-desktop/012-building-workflow/`、`pi-desktop-android/012-building-workflow/`。
- 外部：GitHub Actions、npm registry、Maven Central、上游仓库。
