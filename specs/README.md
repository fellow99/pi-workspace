# Pi Desktop Workspace 规范文档索引

**项目名称：** Pi Desktop Workspace（父工程 `pi-workspace`）
**定位：** 把上游 [Pi](https://github.com/earendil-works/pi) 与 [Pi Web](https://github.com/agegr/pi-web) 封装为桌面应用的**聚合工程**
**产品形态：** 四种 —— Pi Desktop（Electron）· Pi Desktop · Electron on HarmonyOS 版 · Pi Desktop · HarmonyOS 版 · Pi Desktop · Android 版
**上游基线：** `pi` **v1.2.0**（`06a466935536f8c9bcacb2d5efc0fc021be1bdff`）· `pi-web` **v0.12.1**（`f83b1a502f8e434555a7153fe08e1c135dd65ca9`）
**当前发布基线：** `v0.12.1`
**文档生成时间：** 2026-10-31
**最后更新：** 2026-10-31

---

## 一、这是什么

`pi-workspace` 是一个 **submodule 容器**，把 Pi Desktop 四个版别所需的全部工程汇聚到同一父目录，使各封装构建脚本期望的兄弟目录布局（`../pi`、`../pi-web`、`../assets`、`../harmony*`）在 clone 后天然成立：

| 工程 | 角色 | 产物 |
|---|---|---|
| `pi-desktop` | Electron 桌面壳（Windows / macOS / Linux） | 安装包 / zip / dmg / deb / rpm |
| `pi-desktop-hos` | Electron-on-HarmonyOS 版（ArkTS + Electron 运行时） | 签名 HAP |
| `pi-desktop-ark` | HarmonyOS 纯血版（ArkTS + ArkWeb + 进程内 Node） | 签名 HAP |
| `pi-desktop-android` | Android 版（Kotlin + 系统 WebView + 进程内 Node） | 签名 APK |
| `pi` / `pi-web` | 上游 agent harness 与浏览器界面 / Next 服务 | 被四版共同消费 |
| `harmonypc-electron` | Electron-on-鸿蒙运行时 | 仅 `pi-desktop-hos` |
| `pi-desktop-website` | 四版产品官网 | 静态站点 |
| `assets/` | 品牌物料唯一来源 | 图标 / Logo / 截图 |

四个版别的共同点：**UI 100% 复用上游 `pi-web`，业务在进程内 Node 中运行，壳只做装配**；没有 CORS、没有自定义协议、没有登录页、不 fork 上游源码。差异集中在运行时、打包形态、平台能力缺口与网络绑定。

---

## 二、文档总览

| 层级 | 分类 | 文档数量 | 说明 |
|---|---|---|---|
| 整体 | 项目级顶层文档 | 5 | README · STRUCTURE · TECH · ARCHITECTURE · constitution |
| 整体 | 整体规格 / 方案 | 2 | overall-spec · overall-plan |
| 模块 | 公共模块（`011` / `012` / `013`） | 3 | 版本更新 · 构建工作流 · 品牌化 |
| **合计** | **4 个目录 / 13 份文档** | | 7 份项目级 + 3 个模块目录（各 2 份 = 6 份）→ 共 **13 份 `.md`** |

> **本父工程规格集刻意不含** `API.md`、`overall-api.md`、`overall-data-model.md`、`overall-test-cases.md`：
> 数据面（`/`、`/api/*`、SSE）由上游 `pi-web` 提供，父工程不自有 API；数据模型与测试用例落在各子工程 `specs/`。
> 这是父工程的**明确取舍**，不是缺漏。

---

## 三、项目级顶层文档

| 文档 | 路径 | 说明 |
|---|---|---|
| **项目结构** | [STRUCTURE.md](./STRUCTURE.md) | 整个 workspace 的目录文件结构、四个子工程布局、关键配置与入口 |
| **技术选型** | [TECH.md](./TECH.md) | 四版技术栈、版本锁定、运行时包、构建工具链、兼容风险 |
| **整体架构** | [ARCHITECTURE.md](./ARCHITECTURE.md) | 四版共同架构、进程模型、数据流、鉴权差异、载荷/补丁、构建发布拓扑 |
| **宪法原则** | [constitution.md](./constitution.md) | 模块编号规则、文档分工、源码即真理、上游零改动、提交与评审纪律 |
| **整体规格** | [overall-spec.md](./overall-spec.md) | 技术无关的系统级需求（`FR-000-*`）、四版能力矩阵、非功能需求、边界 |
| **整体方案** | [overall-plan.md](./overall-plan.md) | 系统级技术方案、四版实现策略、跨切面、构建发布、验证与风险 |

---

## 四、公共模块（`0XX`）

> `0XX` 是四个子工程**公共、共同**的需求模块。父工程写**跨工程规范**，子工程写**本工程实现**。
> 编号规则见 [constitution.md](./constitution.md) 第一条。

### 011 — 版本更新：0.12.1（version）

> 上游 `pi` v1.2.0 / `pi-web` v0.12.1 的统一收敛；四版应用版本、运行时版本、补丁数、发布 tag 与产物的对照。

| 文档 | 链接 | 说明 |
|---|---|---|
| 功能规格 | [011-version-0.12.1/spec.md](./011-version-0.12.1/spec.md) | 跨工程版本口径、四版版本矩阵、发布与证据要求（`FR-011-*`） |
| 技术方案 | [011-version-0.12.1/plan.md](./011-version-0.12.1/plan.md) | 各子工程升级步骤、验证矩阵、回滚与历史保留 |

### 012 — 构建工作流（building-workflow）

> 可复现、可门禁、可追溯的构建与发布流程。**适用范围：`pi-desktop`、`pi-desktop-android`**；
> 按父工程规则，**鸿蒙工程暂无此模块**（`pi-desktop-hos` / `pi-desktop-ark` 使用各自本地 hvigor 构建，见子工程文档）。

| 文档 | 链接 | 说明 |
|---|---|---|
| 功能规格 | [012-building-workflow/spec.md](./012-building-workflow/spec.md) | 构建可复现、上游钉住、门禁、产物与 CI 要求（`FR-012-*`） |
| 技术方案 | [012-building-workflow/plan.md](./012-building-workflow/plan.md) | 桌面 npm/Forge 与 Android Gradle/GHA 两条流水线、产物命名与验证 |

### 013 — 品牌化（branding）

> 四版统一的品牌口径：`Pi Desktop` 名称、父工程 `assets/` 唯一物料、图标 / Logo / favicon / PWA、
> 用户可见 `Pi Web` 清零、技术标识符不变、物料缺失即构建失败。

| 文档 | 链接 | 说明 |
|---|---|---|
| 功能规格 | [013-branding/spec.md](./013-branding/spec.md) | 跨工程品牌要求与四版落点（`FR-013-*`） |
| 技术方案 | [013-branding/plan.md](./013-branding/plan.md) | 各版品牌化脚本/资源/overlay 的实现与验收 |

---

## 五、模块编号与文档分工

### 5.1 编号规则（摘要，完整版见宪法）

| 段位 | 含义 | 本 workspace 已用编号 |
|---|---|---|
| `0XX` | 各子工程公共、共同的需求模块 | `011-version-<VERSION>`、`012-building-workflow`、`013-branding` |
| `1XX` | 各工程自己的基础功能模块（尽量对齐 `pi-desktop-android` 基线） | 101 运行时接入 · 102 载荷流水线 · 103 渲染壳 · 104 宿主能力桥 · 105 鉴权 · 106 工具扩展 · 107 合规 · 108+ 平台特有 |
| `2XX` | hub-console 前端模块 | 201 前端壳 · 202 session 上下文查看器 |

### 5.2 文档分工

- **父工程 `specs/`**：整体规划、整体设计、跨工程公共需求、版本矩阵、构建/品牌总规范。
- **子工程 `specs/`**：本工程具体技术实现、独立需求、模块 `spec.md` + `plan.md`、真机证据。
- **`docs/`**：各工程的操作手册、工程规划、能力清单、真机验证记录；`specs/` 只收敛结论并回引，不复制论证。

### 5.3 编号迁移（已执行）

- 子工程 `101-version-<VERSION>` → **`011-version-<VERSION>`**；
- 子工程 `012-branding` / `103-branding` → **`013-branding`**；
- 子工程 `001-building-workflow` / `010-build-pipeline` → **`012-building-workflow`**；
- 基础模块由 `00X` 迁移到 `1XX`，与 Android 基线对齐；迁移对照表见各子工程 `specs/README.md`。

---

## 六、快速导航

| 目标读者 | 推荐阅读顺序 |
|---|---|
| 先看全貌 | [README_zh.md](../README_zh.md) → 本文件 → [ARCHITECTURE.md](./ARCHITECTURE.md) |
| 新加入开发者 | [constitution.md](./constitution.md) → [STRUCTURE.md](./STRUCTURE.md) → [overall-spec.md](./overall-spec.md) → 对应子工程 `specs/` |
| 架构师 / 评审 | [ARCHITECTURE.md](./ARCHITECTURE.md) → [TECH.md](./TECH.md) → [overall-plan.md](./overall-plan.md) → 子工程 `constitution.md` |
| 构建 / 发布 | [012-building-workflow](./012-building-workflow/) → [011-version-0.12.1](./011-version-0.12.1/) → 子工程构建文档 |
| 品牌 / 物料 | [013-branding](./013-branding/) → [assets/](../assets/) → 子工程品牌化模块 |
| 能力 / 差异 | [overall-spec.md](./overall-spec.md) §4 → [docs/能力对照清单.md](../docs/能力对照清单.md) |
| 测试 / QA | [overall-plan.md](./overall-plan.md) §8 → 子工程 `overall-test-cases.md` / 真机记录 |

---

**文档维护者：** Pi Desktop Workspace 开发团队
