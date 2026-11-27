# 101 — 版本更新：0.19.0（version）跨工程功能规格

> **Module:** version ｜ **Status:** In-progress ｜ **Last Updated:** 2026-11-27
> **上游基线:** `pi` v1.6.1（`ef4a1cfcca62b77ad3558337ff3e1c7f6beeadfe`，Branch_v1.6.1）· `pi-web` v0.19.0（`7b77aa1a01a74597280dd5bbc9221195411bddf4`，Branch_v0.19.0）
> **发布 tag:** `v0.19.0`
> **适用范围:** `pi-desktop` · `pi-desktop-hos` · `pi-desktop-ark` · `pi-desktop-android` · `pi-desktop-website`
> **父/子分工:** 本文写**跨工程版本口径与矩阵**；各子工程的升级步骤、补丁、构建证据写在各自提交与 `patches/v0.19.0/`。

---

## 1. 模块概述

### 1.1 目的

把五个工程统一收敛到 `v0.19.0` 基线，并把上游基线升到 `pi` v1.6.1 + `pi-web` v0.19.0。

### 1.2 解决的问题

- **上游 `pi-web` v0.18.0 → v0.19.0**：284 文件，+8976/−2383，44 提交。功能要点（逐条冲突评估见 plan §2）：
  - **安全 #1827**：页面响应新增 `Content-Security-Policy: frame-ancestors 'self'` 与 `X-Frame-Options: SAMEORIGIN`；
    新增 `PI_WEB_FRAME_ANCESTORS` 供异源门户嵌入时放开（`lib/frame-ancestors.ts`）。
  - **离线 #1828**：`PI_OFFLINE` 判定统一为 `!== undefined`（新 `lib/pi-offline.ts`），影响 app-update / 模型目录 / 插件更新三处。
  - **新增**：`#` 引用其他会话（chip + 有界 transcript）、侧边栏「按时间排序」、聊天行号链接跳行 + diff 双栏、
    设置›通用新增「引导消息 / 后续消息 / 网络代理 / 提示（四开关）」、折叠工具步骤显示总费用。
  - **删除路由**：`/api/cache-miss-notices`（并入 `/api/notices`）。
  - **新增路由**：`/api/http-proxy`、`/api/notices`、`/api/queue-modes`、`/api/file-index`。
  - **内部重构**：抽出 `lib/errors.ts`（`errorMessage`）、`lib/fetch-json.ts`（`readJsonOrThrow`）、
    `lib/content-disposition.ts`、`lib/thinking-levels.ts`、`lib/browser-storage.ts` 等共享件。
- **`pi` v1.6.0 → v1.6.1**：591 文件，+14587/−3999。
  - **破坏性**：`showCacheMissNotices` 设置被拆为 `notices.{cacheMisses,cacheWarming,compactionCosts,reasoningDrops}`
    （`SettingsManager.getShowCacheMissNotices()` / `setShowCacheMissNotices()` 移除）。
  - 新增 codemode `models.classify()`（decision models）、扩展 `ctx.ui.*` 的 `blockedKind`、bash 工具 `full_output_error`。
  - 修复：超大会话（≈12 万条）构建上下文栈溢出、Windows 控制台关闭跳过 `session_shutdown`、
    `!` 命令输出写临时文件失败导致崩溃、GPT-6 Codex 断连后 `prefix_mismatch` 丢弃推理等。
- **补丁 rebase**：4 端 16 条补丁中 **6 条冲突（10 条干净）**，根因仅 2 个（见 plan §2.2）：
  ① 上游抽出 `readJsonOrThrow` / `errorMessage` 正好落在 `DirectoryPicker.tsx` 的导入与错误处理处（HOS `0001`、Ark/Android `0005`）；
  ② 上游给 `app/api/terminal/route.ts` 顶部加 `import { errorMessage }`，与我方能力门导入位置重合（Ark/Android `0001`）；
  ③ 上游给 `components/SettingsPanel.tsx` 增加 `HttpProxyForm` / `readJsonOrThrow` / `errorMessage` 导入，
  落在 pi-desktop `0001` 的导入上下文区间（`ConfigButton` 与 `FontSettings` 之间不再相邻）。

### 1.3 范围

**包含**：五版上游基线一致性（pi v1.6.1、pi-web v0.19.0）；应用版本/versionCode/产物/tag 矩阵；
`patches/v0.19.0/` rebase 与幂等验证；四端清理中间/临时文件；四端构建产物；官网/README/AGENTS 更新；
CI 上游钉住更新；父工程与各子工程 tag `v0.19.0`；**Windows + Android 真机 + 鸿蒙真机三端测试**（含 `docs/能力对照清单.md` 端侧能力）。
**不包含**：修改上游源码（只允许幂等补丁）；Debian/Ubuntu/macOS 验证（暂缓）；签名 HAP 材料变更。

---

## 2. 需求条目

### FR-101-190-001 上游基线一致性
`pi` 工作树必须位于 `Branch_v1.6.1` / tag `v1.6.1`；`pi-web` 必须位于 `Branch_v0.19.0` / tag `v0.19.0`。
四端补丁目录名为 `patches/v0.19.0/`，`patches/README.md`、`scripts/*.mjs` 中的上游 tag 常量同步。

### FR-101-190-002 补丁集 rebase 与幂等
每条补丁必须满足既有 8 条补丁纪律（幂等、冲突即终止、不带镜像噪声、按上游 tag 组织、不落在既有连续上下文、
可往返 SHA-256 还原、只在 detached worktree 生成、登记三要素）。补丁文件一律 **UTF-8 / LF**。
**本轮额外要求**：在 `v0.19.0` 工作树上 `git apply --check` 全绿，且 apply → reverse 后 `git status --porcelain` 为空。

### FR-101-190-003 版本号与产物矩阵
四端应用内版本 `0.19.0`、versionCode `1900`；产物名/tag 与矩阵一致（见 §3.2）。

### FR-101-190-004 清理中间/临时文件
构建前清理各工程构建产物与临时目录（各工程 `.gitignore` 已列明的派生目录），不删除源码、不删除已入库文件、
不删除签名材料与本地配置。

### FR-101-190-005 功能零冲突约束
四端**不得**为适配 0.19.0 而新增除补丁 rebase 之外的源码改动。0.19.0 的功能变更对四端均为继承性受益或与本机无关：
- **#1827 iframe 防嵌套**：四端均为**顶层加载**（Electron `BrowserWindow.loadURL` / ArkWeb `Web({src})` / Android `WebView.loadUrl`），
  宿主无 iframe，`X-Frame-Options: SAMEORIGIN` 不作用于顶层导航；pi-web 自有子文档（DOCX 预览、SVG）由同源页面 frame，`'self'` 足够。
  **不设 `PI_WEB_FRAME_ANCESTORS`**（保持默认 `'self'`，安全面最小）。
- **#1828 `PI_OFFLINE` 统一**：`pi-desktop/src/main/host.ts` 与 `PiDesktopRuntime.kt` 设 `PI_WEB_SKIP_VERSION_CHECK=1`，
  该开关在 v0.19.0 的 `app/api/app-update/route.ts` 仍是 `SKIP_VERSION_CHECK` 的短路条件之一（`|| isPiOffline()`）；
  Ark 与 HOS 均设 `PI_OFFLINE=1`（`src-main/main.js` 中 `if (!process.env.PI_OFFLINE) process.env.PI_OFFLINE = '1'`），
  Desktop 与 Android **不设** `PI_OFFLINE`（尤其不得设为 `0`／空串，否则新规则会判为离线）。值恒为 `'1'`，新判定下语义不变。
- **pi 1.6.1 `showCacheMissNotices` 移除**：四端源码 / 补丁 / `pi-agent-hub` 均无引用（已 grep 全库确认为 0 命中）。
- **沙箱契约零改动**：补丁**触及的上游文件**里，只有 4 个在 `v0.18.0..v0.19.0` 有改动 ——
  `components/DirectoryPicker.tsx`（+7/−8，`loadDirectories`/`createDirectory` 改用 `readJsonOrThrow` + 2 处 `errorMessage`）、
  `app/api/sessions/[id]/export/route.ts`（+2/−13，`Content-Disposition` 抽函数）、`components/AppShell.tsx`（+45/−22）、
  `app/api/terminal/route.ts`（+2/−1）、`components/SettingsPanel.tsx`（+86/−39，**且全 4 条冲突都由这几处引起**）。
  以下上游文件在 `v0.18.0..v0.19.0` **逐文件零改动**（`git diff --numstat` 逐条验证）：
  `app/api/cwd/browse/route.ts`、`app/api/cwd/validate/route.ts`、`lib/directory-browser.ts`、`lib/i18n/messages/{en,zh-CN,zh-TW}.ts`。
  （`app/api/cwd/workspace-permission/route.ts`、`app/api/host/perm/route.ts`、`lib/terminal-capability.ts`、
  `lib/hos-sandbox-path.ts`、`components/SyncConfig.tsx` 是**我们补丁新建的文件，上游根本不存在**，
  「上游零改动」对它们无意义 —— 本节此前把它们混在零改动清单里，已修正。）
- **新增路由**（`/api/http-proxy`、`/api/notices`、`/api/queue-modes`、`/api/file-index`）四端未直接调用；
  **删除路由** `/api/cache-miss-notices` 四端未引用。

### FR-101-190-006 官网与 README 更新
`pi-desktop-website` 两页（`index.html` / `index_zh.html`，含能力矩阵）与父/子工程 `README*.md`、四端 `AGENTS.md` 的
版本号、版本描述、补丁数、上游基线更新到 0.19.0 口径。

### FR-101-190-007 CI 上游钉住更新
`pi-desktop/.github/workflows/{ci,release}.yml` 的 `PI_REF`/`PI_SHA`/`PI_WEB_REF`/`PI_WEB_SHA`；
`pi-desktop-android/.github/workflows/build-release.yml` 的 `PI_WEB_TAG`（及 pi 钉住项，如存在）更新到 v1.6.1 / v0.19.0。

### FR-101-190-008 提交与 tag 纪律
提交信息使用 **Conventional Commits 表内 type**（`feat`/`fix`/`docs`/`style`/`refactor`/`perf`/`test`/`build`/`ci`/`chore`/`revert`），
**禁止表外前缀**；只暂存本会话改动文件（严禁 `git add -A`）；各子工程与父工程打 tag `v0.19.0`；**不改动分支**。

### FR-101-190-009 三端测试
测试范围：**Windows x64 本机**（注意：被测进程可能即当前宿主 `pi-desktop`，禁止误杀）、**Android 真机**（`adb install -r` 覆盖安装，
**禁止卸载**）、**鸿蒙真机**（hos + ark 两版）。基本功能：DeepSeek Key 配置 → 沙箱工作区 → 文本对话 → 图片分析对话；
端侧能力按 `docs/能力对照清单.md` 执行。Debian / Ubuntu arm64 暂缓。

### FR-101-190-010 敏感信息不落盘
模型 API Key、测试账号口令等**只允许运行期注入**，不得写入任何文档、日志、spec、测试用例或提交内容。

---

## 3. 版本矩阵

### 3.1 上游

| 上游 | 版本 | SHA / Branch |
|---|---|---|
| `pi` | v1.6.1 | `ef4a1cfcca62b77ad3558337ff3e1c7f6beeadfe` / Branch_v1.6.1 |
| `pi-web` | v0.19.0 | `7b77aa1a01a74597280dd5bbc9221195411bddf4` / Branch_v0.19.0（annotated tag `e5192a75`） |

### 3.2 应用

| 工程 | 版本 | versionCode | 构建产物 |
|---|---|---|---|
| pi-desktop | 0.19.0 | — | Setup.exe / nupkg / zip |
| pi-desktop-hos | 0.19.0 | 1900 | HAP（签名依本机材料） |
| pi-desktop-ark | 0.19.0 | 1900 | HAP（签名依本机材料） |
| pi-desktop-android | 0.19.0 | 1900 | debug / release APK |
| pi-desktop-website | 0.19.0 | — | 静态站 |

### 3.3 补丁矩阵（目标）

| 工程 | 补丁数 | 目录 | 本轮冲突处置 |
|---|---|---|---|
| pi-desktop | 1 | `patches/v0.19.0/` | `0001` 导入块重定位（上游新增 `HttpProxyForm`/`readJsonOrThrow`/`errorMessage`） |
| pi-desktop-hos | 3 | `patches/v0.19.0/` | `0001`：`DirectoryPicker.tsx` 重放并采纳上游 `readJsonOrThrow`/`errorMessage`；`0002`/`0003` 干净 |
| pi-desktop-ark | 6 | `patches/v0.19.0/` | `0001` 导入位置后移；`0005`：同 HOS 的 `DirectoryPicker` 重放 |
| pi-desktop-android | 6 | `patches/v0.19.0/` | 同 Ark（`0001` 的 `lib/terminal-capability.ts` 与 Ark 不同，Android 有 PTY 支持分支） |

> 补丁**编号只增不改、不复用**：本轮沿用 v0.18.0 的编号（同一补丁的升级版），不新占全局编号；
> `0006` 为空号（v0.17.0 起合并进 `0005`），`0007` 已退役，保留空号不复用。

---

## 4. 非功能要求

| 项 | 要求 |
|---|---|
| 上游零改动 | `../pi`、`../pi-web` 在构建窗口期之外必须 pristine（`git status` clean） |
| 补丁可往返 | 应用 → 反向 → `git status --porcelain` 为空（受影响文件 SHA-256 逐条还原） |
| 敏感信息 | 不落盘任何 Key / 口令 / 私钥 / keystore |
| 构建可复现 | 补丁按文件名排序应用，结果确定性；`BUILD_ID` 写入日志 |
| 文档一致 | 官网/README/AGENTS/CI 四处版本口径一致，无残留 0.18.0 当轮口径 |
| 分支不动 | 五个子工程与父工程均保持各自现有分支（`main`），不新建、不切换 |

## 5. 边界与假设

- 假设 1：`pi` / `pi-web` 分支已由用户固化（已核实：`Branch_v1.6.1` @ v1.6.1、`Branch_v0.19.0` @ v0.19.0）。
- 假设 2：本机具备 Electron / DevEco(hvigor) / Gradle 工具链；签名材料按各自 `signing.*.local.json` / `keystore.properties` 现状处理。
- 假设 3：Android 真机与鸿蒙真机已连数据线（`adb devices` / `hdc list targets` 可达）；Windows 测试在本机执行。
