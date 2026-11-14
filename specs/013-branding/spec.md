# 013 — 品牌化（branding）跨工程功能规格

> **Module:** branding ｜ **Status:** Implemented（Desktop / HOS / Android）；Partial（Ark） ｜ **Last Updated:** 2026-11-14（R2）
> **上游基线:** `pi` v1.2.0 / `pi-web` v0.12.1
> **适用范围:** 四个子工程；**每个子工程都必须有 `013-branding/`**
> **物料唯一来源:** 父工程 [`assets/`](../../assets/)（`logo.svg` / `logo.png` / `logo-1024-with-bg.png` / `logo.ico` / `logo.icns`）
> **父/子分工:** 本文写统一品牌口径与四版落点；脚本、资源位、验收证据写在子工程 `013-branding/`。

---

## R2 — 包名变更（013-branding-r2）

> **需求编号:** `013-branding-r2` ｜ **需求名称:** 包名修改 ｜ **Git 分支:** 不改动 ｜ **日期:** 2026-11-14
> **适用范围:** `pi-desktop-android` / `pi-desktop-hos` / `pi-desktop-ark`
> **性质:** 按**产品决策**对技术标识符（包名 / bundleName）做一次性变更；**覆盖**本规格第 §1.1 / §1.3 / FR-013-004 / FR-013-022 中「不改包名」的表述。版本号不变。

### R2.1 新包名

| 子工程 | 旧包名 / bundleName | 新包名 / bundleName |
|---|---|---|
| `pi-desktop-android` | `org.fellow99.pi.PiDesktopAndroid` | `top.fellow99.pi.PiDesktopAndroid` |
| `pi-desktop-hos` | `org.fellow99.pi.PiDesktop` | `top.fellow99.pi.PiDesktopHos` |
| `pi-desktop-ark` | `org.fellow99.pi.PiDesktopArk` | `top.fellow99.pi.PiDesktopArk` |

### R2.2 影响面

- **Android**：`namespace` / `applicationId` 随之变更；源码包目录 `app/src/main/java/{org→top}/fellow99/pi/PiDesktopAndroid/` 迁移，8 个 `.kt` 的 `package` 声明与内部 `import` 同步更新。
- **HOS / ARK**：仅变更 `AppScope/app.json5` 的 `bundleName`；HOS 另需同步内部自引用的 `shortcuts_config.json`（`electron` 与 `runtime-overlays/electron` 两处）及 `web_engine/src/main/ets/adapter/*.ets` 4 处 `bundleName` 常量。
- **规格**：本父规格与三子工程 `specs/013-branding` 同步回写。

### R2.3 约束

- 仅改包名 / bundleName；**不做**其他技术标识符（`pi-web:*`、`PI_WEB_*`、`pi_web_session`、API 路由等）变更。
- **不改版本号**；三版版本号/versionCode 维持现状。
- 改动后各子工程**编译构建验证**，确保产物可安装运行。

---

## 1. 模块概述

### 1.1 目的

四个版别交付给用户的是**同一个产品 Pi Desktop**，不是“上游 Pi Web 的四个壳”。品牌化要保证：

- 应用名、窗口标题、页面标题、PWA 名一致；
- 应用图标、启动图标、托盘/状态栏、favicon、PWA 图标、页面 Logo 来自同一物料；
- 用户可见的 `Pi Web` 清零；
- 技术标识符（`pi-web`、`PI_WEB_*`、存储 key、路由）保持不变；包名 / bundleName 按 §R2 单独变更；
- 物料缺失/尺寸/透明度不符时构建失败，不产出半品牌包。

### 1.2 解决的问题

- 用户看到上游 `Pi Web` 品牌，无法区分产品与上游。
- 四版图标来源不一，出现占位图/旧图/尺寸不符。
- 手工拷贝不可复现；换物料不生效、缺物料不报错。
- 品牌化误改技术标识符，破坏兼容/用户数据。
- 版本检测指向上游 `pi-web`，给用户错误更新提示。

### 1.3 范围

**包含**

- 应用名与用户可见文案：`Pi Web` → `Pi Desktop`。
- 应用图标 / 启动图标 / 窗口图标 / 托盘或状态栏字形 / favicon / PWA / apple-touch-icon / 页面 Logo。
- 物料硬校验：存在、尺寸、透明度、幂等。
- 运行时开关：`PI_WEB_APP_NAME`、`PI_WEB_SKIP_VERSION_CHECK`。
- 构建期终检：0 残留、技术标识符计数不变。
- 四版文档与能力清单回写。

**不包含**

- 修改上游 `pi` / `pi-web` 源码。
- 改包名 / bundleName（品牌名与应用名一致，包名保留渠道标识）——**R2 起按产品决策变更**，见 §R2。
- UI 交互设计 / i18n 文案重写（除品牌名）。
- 官网物料（`pi-desktop-website`）。
- 代码签名 / 公证。

---

## 2. 用户故事

- 作为用户，我在任务栏/桌面/应用列表/浏览器标签看到的是 **Pi Desktop**。
- 作为用户，我在登录页、空会话、侧栏看到统一 Logo，而不是上游图标。
- 作为用户，我不会看到“pi-web 有新版本”的错误提示，但能看到当前版本。
- 作为维护者，我换 `assets/` 物料并重跑流水线，四版全链更新。
- 作为维护者，物料缺失或透明通道不符时构建立即失败。
- 作为审核者，我能用脚本证明产物中用户可见 `Pi Web` 为 0、技术标识符未变。

---

## 3. 功能需求

### 3.1 品牌名与文案

- **FR-013-001**（MUST）：四版用户可见主品牌名必须为 **Pi Desktop**；允许渠道后缀（如 Ark），但 UI 主名必须一致。
- **FR-013-002**（MUST）：用户可见 `Pi Web` 必须清零；覆盖页面元数据、登录页、hero、侧栏、`document.title`、离线页、PWA manifest、i18n 品牌串、无障碍标签。
- **FR-013-003**（MUST）：改名必须落在**交付产物**中，不能只改源码或构建脚本。
- **FR-013-004**（MUST NOT）：不得改技术标识符：`pi-web`、`@agegr/pi-web`、`pi-web:*`、`PI_WEB_*`、`pi_web_session`、`PiWeb`、API 路由、SW 缓存前缀。包名 / bundleName 由 §R2 单独变更管控（FR-013-004 不再涵盖）。
- **FR-013-005**（MUST）：运行时必须置位 `PI_WEB_APP_NAME=Pi Desktop`，并按平台置位 `PI_WEB_SKIP_VERSION_CHECK=1`（保留当前版本显示）。

### 3.2 物料与图标

- **FR-013-006**（MUST）：物料唯一来源为父工程 `assets/`；不得从上游/参考工程拷贝图标充当品牌图标。
- **FR-013-007**（MUST）：应用级图标必须为 **1024×1024 且不透明**（alpha 全 255）；窗口/托盘/状态栏图标允许透明。物料可用 `logo-1024-with-bg.png`，或由 `logo.png` 放大并合成白底（Ark 采用后者）。
- **FR-013-008**（MUST）：每一个被引用的图标位必须来自同一物料来源（`assets/` 集合）：应用图标、启动图标、窗口图标、托盘/状态栏、快捷方式、PWA 192/512、apple-touch-icon、favicon、页面 Logo。允许按平台选择同一集合中的不同文件（如 Ark 用 `logo.png` 合成白底）。
- **FR-013-009**（MUST）：页面 Logo 必须使用矢量 `logo.svg` 或 ≥ 显示尺寸的清晰版本；不得把 200² 位图放大当大图标。
- **FR-013-010**（MUST）：物料缺失、尺寸不符、透明度不符 ⇒ 构建失败；不得静默降级。
- **FR-013-011**（SHOULD）：图标生成必须幂等且确定性（同一物料 → 同一字节）。

### 3.3 四版落点

- **FR-013-012**（MUST）：Desktop 必须品牌化 `pi-dist` 副本（文案/Logo/web 位图/favicon）与 `resources/*`（应用/窗口/托盘/安装器）。
- **FR-013-013**（MUST）：HOS 必须品牌化 pi-dist 副本、`AppScope` 图标、`runtime-overlays` 状态栏字形、窗口/启动图标。
- **FR-013-014**（MUST）：Ark 必须品牌化载荷文案、应用/启动图标；页面 Logo 与应用名统一为待收口项（见 §3.4）。
- **FR-013-015**（MUST）：Android 必须品牌化载荷文案/Logo/favicon/PWA 与五密度 launcher/adaptive 图标。

### 3.4 未闭合项（Ark）

- **FR-013-016**（SHOULD）：Ark 页面 Logo 应改用 `logo.svg`；当前 **⬜**（无 SVG 光栅化器）。
- **FR-013-017**（SHOULD）：Ark 应用名应从 `Pi Desktop Ark` 统一为 `Pi Desktop`；当前 **⬜**（待产品定名）。
- **FR-013-018**（SHOULD）：Ark 首屏标记探测应与品牌化产物一致；当前 **⬜**。

### 3.5 判据与回归

- **FR-013-019**（MUST）：品牌化后产物（排除 `node_modules/`）用户可见 `Pi Web` 必须为 0；`Pi Desktop` 必须出现。
- **FR-013-020**（MUST）：技术标识符替换前后计数必须逐项不变（机器断言）。
- **FR-013-021**（MUST）：品牌化不得回归启动、会话、对话、工具、文件上传、Git。
- **FR-013-022**（MUST）：品牌化不得改变版本号/包名。**R2 例外**：013-branding-r2 已按产品决策变更包名 / bundleName（见 §R2），版本号不变。
- **FR-013-023**（MUST）：构建必须提供可单独运行的终检命令（`--check` 或等价）。

---

## 4. 关键实体

| 实体 | 说明 | 关键属性 |
|---|---|---|
| 物料 | 父工程 `assets/` | SVG / 200² 透明 PNG / 1024² 不透明 PNG / ICO / ICNS |
| 应用图标 | 各版资源目录 | 1024² 不透明；五密度/多尺寸 |
| Web 图标 | 载荷 `public/icons/` | 192/512/180/favicon |
| 文案 | 用户可见品牌 | `Pi Web` → `Pi Desktop` |
| 运行时开关 | 环境变量 | `PI_WEB_APP_NAME` / `PI_WEB_SKIP_VERSION_CHECK` |

---

## 5. 四版落点矩阵

| 项 | Desktop | HOS | Ark | Android |
|---|---|---|---|---|
| 应用名 | ✅ | ✅ | ⚠️ `Pi Desktop Ark` | ✅ |
| 应用图标 | ✅ | ✅ | ✅ | ✅ |
| 启动/窗口图标 | ✅ | ✅ | ✅ | ✅ |
| 托盘/状态栏 | ✅ | ✅ | ➖ | ➖ |
| 页面 Logo | ✅ `logo.svg` | ✅ `logo.svg` | ⬜ 上游 PNG | ✅ `logo.svg` |
| favicon/PWA | ✅ | ✅ | ⬜ | ✅ |
| UI 文案 0 残留 | ✅ | ✅ | ✅ | ✅ |
| 技术标识符不变（包名除外，R2 变更） | ✅ | ✅ | ✅ | ✅ |
| 版本检测关闭 | ✅ | ✅ | ✅ | ✅ |
| 物料缺失硬失败 | ✅ | ✅ | ✅ | ✅ |
| 幂等/终检 | ✅ | ✅ | ✅ | ✅ |

---

## 6. 验收场景

### 场景：文案清零

- Given 品牌化后的产物
- When 搜索用户可见 `Pi Web`（排除 `node_modules/`）
- Then 0 处；`Pi Desktop` > 0

### 场景：图标一致

- Given 父工程物料
- When 生成并检查各图标位
- Then 尺寸/透明度符合规则；页面 Logo 为 `logo.svg` 或等价矢量

### 场景：技术标识符不变

- Given 品牌化前/后
- When 统计 `pi-web`、`PI_WEB_`、`pi_web_session`、`PiWeb`
- Then 计数逐项相等

### 场景：物料缺失

- Given 移走物料
- When 构建/生成
- Then 失败并提示物料路径

### 场景：四版一致

- Given 四版安装包/HAP/APK
- When 检查应用名/图标/首屏
- Then 主品牌名为 `Pi Desktop`，Logo 同一物料

---

## 7. 非功能需求

- **确定性**：同一物料重复生成字节稳定。
- **性能**：构建期完成，不增加运行时开销。
- **安全**：品牌脚本不读取凭据；不修改签名材料。
- **可维护**：物料路径可覆盖（`--assets` / `PI_ASSETS_DIR`）。
- **可审计**：终检命令输出残留文件/计数。

---

## 8. 假设与约束

- 父工程 `assets/` 在构建环境可用（CI sparse checkout / 本地）。
- 图像工具可用：Python + Pillow（Desktop/HOS/Android）、ffmpeg（Ark/HOS）。
- Ark 无 SVG 光栅化器；页面 Logo 需换方案。
- 应用名 `Pi Desktop Ark` 是当前定名，统一需产品决策。

---

## 9. 依赖

- 父工程：`assets/`、`constitution.md`、`011`、`012`。
- 子工程：各版 `013-branding/`、构建脚本、资源目录。
- 外部：Pillow / ffmpeg / 图像库。

---

## 10. 版本与变更

| 版本 | 日期 | 变更 |
|---|---|---|
| 1.0 | 2026-10-31 | 初版：统一四版品牌口径、落点矩阵与 Ark 未闭合项 |
| 1.1 | 2026-11-14 | R2：包名变更（android/hos/ark 迁移至 `top.fellow99.pi.*`），覆盖原「不改包名」约束；版本号不变 |
