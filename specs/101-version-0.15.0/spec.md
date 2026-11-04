# 101 — 版本更新：0.15.0（version）跨工程功能规格

> **Module:** version ｜ **Status:** Verified（四端构建 + 桌面/鸿蒙/安卓真机验证） ｜ **Last Updated:** 2026-11-04
> **上游基线:** `pi` v1.5.0（`f9a26365568127461c47a07a37dcb42c140a3ac5`，**与 0.14.2 持平不变**）· `pi-web` v0.15.0（`f130910abf662b506b2ef2f475fc21a2fe57badc`）
> **发布 tag:** `v0.15.0`
> **适用范围:** `pi-desktop` · `pi-desktop-hos` · `pi-desktop-ark` · `pi-desktop-android` · `pi-desktop-website`
> **父/子分工:** 本文写**跨工程版本口径与矩阵**；各子工程的升级步骤、补丁、构建证据写在各自提交与 `patches/v0.15.0/`。

---

## 1. 模块概述

### 1.1 目的

把五个工程（Desktop / HOS / Ark / Android / Website）统一收敛到 `v0.15.0` 基线，并把上游基线升到
`pi` v1.5.0（持平）+ `pi-web` v0.15.0（升级），回答三个问题：

1. **上游版本**是什么？（`pi` / `pi-web` 的 tag 与 commit）
2. **应用版本**是什么？（各壳的 `versionName` / `versionCode`）
3. **发布 tag** 是什么？（`v0.15.0`，只作发布标识）

### 1.2 解决的问题

- pi-web v0.15.0（2026-11 发布）相对 v0.14.2 的变更（13 个提交，75 文件，+1775/−446）：
  **运行中切换模型**（#1527）、**侧标签随主会话恢复**（#1512）、**右面板标签拖拽排序**（#1511）、
  **自定义 provider 模型拖拽/快捷键/A–Z 排序**、**扩展 UI 取 pi 调色板主题**（新增
  `lib/extension-ui-theme.ts`）、**provider 显示名称而非 id**（#1514）、**完成通知去重**（#1518）、
  Chrome 粘贴补插修复（#1517）、流式 token/速度指示稳定化（#1520）、登出 provider 的
  enabledModels 计数修复（#1529）、OAuth 登录传设备 ID（#1526）、Docker 依赖安装修复（#1525）、
  **删除被 pi 1.5.0 废弃的 HTML 导出字符串补丁**（1fbf6c6）。
- **冲突评估**：以上均为纯前端 / 纯 JS 层变更，不涉宿主桥、运行时或内核 → 对四端**无明显行为冲突**；
  `pi` 本轮**持平 v1.5.0**（0.14.2 已消费 pi 1.5.0，本轮零内核变更，Node/Electron 运行时矩阵全部不动）。
- **补丁漂移检查（实测）**：v0.14.2 → v0.15.0 共改 75 个文件；补丁领域 6 文件中唯一交集
  `app/api/sessions/[id]/export/route.ts` —— 上游 1fbf6c6 仅**删除** `patchExportHtml()`（139 行，
  pi 1.5.0 导出模板已改迭代实现，字符串补丁过时）并把调用点改为直返 `html`；我们两条 export 补丁的
  改动区域（`exportSession()` 的 spawn 守卫 + `webpackIgnore` 动态 import）与被删函数体**不重叠**。
  `hooks/useAgentSession.ts` / `lib/i18n/messages/*` 虽有变更但不在补丁 hunk 上下文内。
  **累计顺序 dry-run 实测**：14 条补丁（0/2/6/6）在 pi-web v0.15.0 纯净树上按构建时顺序
  **全部干净应用**（与 v0.14.2 基线行为一致）⇒ **零漂移，原样拷贝**。

### 1.3 范围

**包含**：五版上游基线一致性（pi 钉住不变、pi-web 钉住升至 v0.15.0）；应用版本 / versionCode / 产物 /
tag 矩阵；补丁集版本目录（`patches/v0.15.0/`）原样迁移与幂等验证；各工程构建产物（Desktop Setup.exe +
zip；HOS 签名 HAP；Ark 签名 HAP；Android debug/release APK）；网站 README/页面更新；父/子 README 更新；
发布 tag；**桌面端**基本功能与环境能力测试。

**不包含**：修改上游源码（只允许幂等补丁）。

> **范围变更（执行中）**：原计划「鸿蒙 / 安卓真机测试暂缓」在执行期间被解除——用户先后接通鸿蒙
> （`3QC0226526001227`）与安卓（`cd3fac61`）真机并要求一并测试，故本轮**四端均做了真机验证**，
> 证据见 `pi-desktop/logs/20261104-1/TEST_REPORT.md`、`pi-desktop-hos/docs/真机验证记录.md`（轮次 8）、
> `pi-desktop-ark/docs/003-真机验证记录-0.15.0.md`、`pi-desktop-android/docs/002-真机验证记录.md`（0.15.0 轮）。

---

## 2. 版本矩阵

### 2.1 上游

| 项 | 版本 | Commit | 相对 0.14.2 |
|---|---|---|---|
| `pi` | **v1.5.0** | `f9a26365568127461c47a07a37dcb42c140a3ac5` | **持平（不变）** |
| `pi-web` | **v0.15.0** | `f130910abf662b506b2ef2f475fc21a2fe57badc` | **升级**（v0.14.2 → v0.15.0） |

### 2.2 五版应用与运行时

| 版别 | 应用版本 | versionCode | 运行时 | Node | 补丁 | 构建/产物 |
|---|---|---|---|---|---|---|
| `pi-desktop` | **0.15.0** | —（package.json） | Electron 43.7.9 | 24.21.0 | 0 | Setup.exe + zip（Windows），smoke 全绿 |
| `pi-desktop-hos` | **0.15.0** | **1500** | Electron-on-HarmonyOS 37.2.3 | 22.17.0 | 2 | HAP（本机签名材料） |
| `pi-desktop-ark` | **0.15.0** | **1500** | node-bridge 0.1.0 + node-runtime 24.2.0 | 24.2.0 | 6 | HAP（本机签名材料） |
| `pi-desktop-android` | **0.15.0** | **1500** | aosp-node-bridge 0.2.0 + node-runtime 26.13.0 | 26.13.0 | 6 | debug + release APK |
| `pi-desktop-website` | **0.15.0** | — | — | — | — | README + index.html + index_zh.html |

> **versionCode 编码**：`major*10000 + minor*100 + patch` ⇒ 0.15.0 → **1500**（HOS/Ark/Android 三端一致）。
> **运行时不变**：Electron-on-HarmonyOS / node-runtime / aosp-node 全部维持 0.14.2 轮次版本（pi 持平）。

### 2.3 发布 tag 与产物

| 版别 | tag | 产物 |
|---|---|---|
| Desktop | `v0.15.0` | `out/make/…/Pi Desktop-0.15.0 Setup.exe`、`Pi Desktop-win32-x64-0.15.0.zip` |
| HOS | `v0.15.0` | `electron/build/…/electron-default-signed.hap`（versionCode 1500） |
| Ark | `v0.15.0` | `entry/build/…/entry-default-signed.hap`（versionCode 1500） |
| Android | `v0.15.0` | `app-debug.apk`、`app-release.apk`（versionCode 1500 / versionName 0.15.0） |
| Website | `v0.15.0` | README + 两页静态站点 |
| 父工程 | `v0.15.0` | specs / README / submodule 指针 |

---

## 3. 冲突评估（pi-web 0.14.2 → 0.15.0）

| 关注点 | 评估 | 结论 |
|---|---|---|
| 运行中切模型 / provider 名称 / 侧标签恢复 / 标签拖拽排序 / 模型排序 | 纯前端 UI / JS | 无冲突 |
| 扩展 UI 调色板主题（`lib/extension-ui-theme.ts` 新增） | 纯前端主题层 | 无冲突 |
| 完成通知去重 / 粘贴补插 / 流式指示稳定化 / OAuth 设备 ID | 前端 + Next API 层 | 无冲突 |
| **删除 `patchExportHtml()`**（1fbf6c6，−139 行） | 与两条 export 补丁改动区域（`exportSession()` 守卫 + `webpackIgnore`）不重叠；pi 1.5.0 模板已迭代化，删除正确 | 无冲突（dry-run 实测 OK，offset −136） |
| **补丁漂移** | 补丁领域 6 文件仅 export 1 文件变更且不接触 hunk；累计顺序 dry-run 14/14 全绿 | **零漂移，原样拷贝** |

---

## 4. 非功能

- **可复现**：同一 tag + 锁文件产出等价包。
- **可追溯**：tag → 上游 SHA → 运行时 → 产物。
- **安全**：API Key / 签名 / 测试凭证不落盘（DeepSeek 测试 Key 仅运行期注入；测试后清理）。
- **可维护**：补丁按上游 tag 分目录，历史 `v0.11.1`/`v0.12.1`/`v0.13.0`/`v0.14.1`/`v0.14.2` 保留。

---

## 5. 假设与约束

- 上游 tag/分支在各仓库可用（本机已固化 `Branch_v1.5.0` / `Branch_v0.15.0`，用户手动完成）。
- 各子工程本地 main 已 fast-forward 到 origin/main（= v0.14.2 发布线顶端）后开工；本轮发布提交
  继续落在 main 上（**Git 分支不改动**）。
- 鸿蒙真机（HAD-W32/API 26）与安卓真机（MIX/Android 8.0.0）**本轮均可用并已实测**；两端无头自动化
  限制（原生文件选择器不可达、WebView/ArkWeb 需经调试通道）已在各自验证记录中说明。
- 桌面端测试本机执行（构建/冒烟以独立进程跑，不杀自身）。
- 测试用的 DeepSeek API Key 由用户给定，**不落盘**。
- HOS/Ark 的 `build-profile.json5` 本地签名材料由 DevEco 自动签名写入 → **仅保留本地，不提交**。
