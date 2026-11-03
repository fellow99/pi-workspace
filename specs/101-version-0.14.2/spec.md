# 101 — 版本更新：0.14.2（version）跨工程功能规格

> **Module:** version ｜ **Status:** In Progress ｜ **Last Updated:** 2026-11-04
> **上游基线:** `pi` v1.5.0（`f9a26365568127461c47a07a37dcb42c140a3ac5`）· `pi-web` v0.14.2（`025e24e74fdbc8f46c4e0cf31c6d6b3fa6cf29d5`）
> **发布 tag:** `v0.14.2`
> **适用范围:** `pi-desktop` · `pi-desktop-hos` · `pi-desktop-ark` · `pi-desktop-android` · `pi-desktop-website`
> **父/子分工:** 本文写**跨工程版本口径与矩阵**；各子工程的升级步骤、补丁、构建证据写在各自提交与 `patches/v0.14.2/`。

---

## 1. 模块概述

### 1.1 目的

把五个工程（Desktop / HOS / Ark / Android / Website）统一收敛到 `v0.14.2` 基线，并把上游基线升到
`pi` v1.5.0 + `pi-web` v0.14.2，回答三个问题：

1. **上游版本**是什么？（`pi` / `pi-web` 的 tag 与 commit）
2. **应用版本**是什么？（各壳的 `versionName` / `versionCode`）
3. **发布 tag** 是什么？（`v0.14.2`，只作发布标识）

### 1.2 解决的问题

- pi-web v0.14.2（2026-11 发布）相对 v0.14.1 的变更：**模型选择器 scoped | all**（`enabledModels`
  下的范围切换）、**Agents 列表删除侧聊**、**手机端 Settings 逐级全屏**（系统返回/滑动动画）、
  **触屏控件尺寸**、**Windows 终端默认 PowerShell**、归档视角空标签修复；内部升级 **pi → 1.5.0**
  （新增固定依赖 `@earendil-works/pi-schema`）、**Next.js 16.4.4 / undici 8.12.3**、Release 工作流合并。
- **冲突评估**：以上均为纯前端 / 纯 JS 层变更，不涉宿主桥、运行时或内核 → 对四端**无明显行为冲突**；
  各壳**不设置 `enabledModels`**，scoped|all 选择器对我们的应用为惰性特性；新增依赖
  `@earendil-works/pi-schema` 为纯 JS schema 包，随 pi-web 的 node_modules（Desktop 整树复制 /
  Android 硬链接 / Ark 裁剪规则仅限 native+wasm）自动纳入。
- **补丁漂移检查**：v0.14.1 → v0.14.2 共改 68 个文件；补丁领域（`app/api/terminal/route.ts`、
  `app/api/cwd/browse/route.ts`、`lib/directory-browser.ts`、`app/api/agent/new/route.ts`、
  `app/api/sessions/[id]/export/route.ts`、`components/DirectoryPicker.tsx`）**全部无变更**；
  唯一交集 `hooks/useAgentSession.ts` 不被任何补丁触及 ⇒ **14 条补丁（0/2/6/6）零漂移，原样拷贝**。

### 1.3 范围

**包含**：五版上游基线一致性；应用版本 / versionCode / 产物 / tag 矩阵；运行时矩阵（Android Node 26.13.0）；
补丁集版本目录（`patches/v0.14.2/`）原样迁移与幂等验证；网站 README/页面更新；父/子 README 更新；
发布 tag；真机基本功能与环境能力测试（本机已接鸿蒙 + 安卓真机）。

**不包含**：修改上游源码（只允许幂等补丁）。

---

## 2. 版本矩阵

### 2.1 上游

| 项 | 版本 | Commit |
|---|---|---|
| `pi` | **v1.5.0** | `f9a26365568127461c47a07a37dcb42c140a3ac5` |
| `pi-web` | **v0.14.2** | `025e24e74fdbc8f46c4e0cf31c6d6b3fa6cf29d5` |

### 2.2 五版应用与运行时

| 版别 | 应用版本 | versionCode | 运行时 | Node | 补丁 | 构建/产物 |
|---|---|---|---|---|---|---|
| `pi-desktop` | **0.14.2** | —（package.json） | Electron 43.7.9 | 24.21.0 | 0 | Setup.exe + zip（Windows），smoke 全绿 |
| `pi-desktop-hos` | **0.14.2** | **1402** | Electron-on-HarmonyOS 37.2.3 | 22.17.0 | 2 | HAP（本机有签名材料 + 鸿蒙真机） |
| `pi-desktop-ark` | **0.14.2** | **1402** | node-bridge 0.1.0 + node-runtime 24.2.0 | 24.2.0 | 6 | HAP（本机有签名材料 + 鸿蒙真机） |
| `pi-desktop-android` | **0.14.2** | **1402** | aosp-node-bridge 0.2.0 + node-runtime **26.13.0** | **26.13.0** | 6 | debug + release APK（真机） |
| `pi-desktop-website` | **0.14.2** | — | — | — | — | README + index.html + index_zh.html |

> **versionCode 编码**：`major*10000 + minor*100 + patch` ⇒ 0.14.2 → **1402**（HOS/Ark/Android 三端一致）。
> **Android 运行时**：aosp-node-bridge 保持 `0.2.0`，aosp-node-runtime 保持 `26.13.0`（本轮不升级）。

### 2.3 发布 tag 与产物

| 版别 | tag | 产物 |
|---|---|---|
| Desktop | `v0.14.2` | `out/make/…/Pi Desktop-0.14.2 Setup.exe`、`Pi Desktop-win32-x64-0.14.2.zip` |
| HOS | `v0.14.2` | `electron/build/…/electron-default-*.hap` |
| Ark | `v0.14.2` | `entry/build/…/entry-default-*.hap` |
| Android | `v0.14.2` | `app-debug.apk`、`app-release.apk`（versionCode 1402 / versionName 0.14.2） |
| Website | `v0.14.2` | README + 两页静态站点 |

---

## 3. 冲突评估（pi-web 0.14.1 → 0.14.2）

| 关注点 | 评估 | 结论 |
|---|---|---|
| 模型选择器 scoped\|all（#1486/#1503） | 纯前端 UI；各壳未设 `enabledModels` → 惰性 | 无冲突 |
| 删除侧聊（#1507）/ 手机 Settings 全屏（#9afc508/#1420b81/#1615347）/ 触屏控件（#57078bb）/ 归档空标签（#1505） | 纯前端 UI / 响应式 | 无冲突 |
| Windows 终端默认 PowerShell（#1394/#1508） | 进程内 pi 行为改进 | 无冲突 |
| 升级 pi→1.5.0 + 固定 `@earendil-works/pi-schema`（ff8de36） | 纯 JS 包；随 node_modules 自动纳入；需 `npm ci` 重装 | 无冲突（构建前置） |
| Next.js 16.4.4 / undici 8.12.3（#1506/#1509） | JS 层升级随包走；engines `>=22.19.0` 未变 | 无冲突 |
| **补丁漂移** | 补丁领域 6 文件 v0.14.2 全未变；仅 `hooks/useAgentSession.ts` 交集且无补丁触及 | **零漂移，原样拷贝** |

---

## 4. 非功能

- **可复现**：同一 tag + 锁文件产出等价包。
- **可追溯**：tag → 上游 SHA → 运行时 → 产物。
- **安全**：API Key / 签名 / 测试凭证不落盘。
- **可维护**：补丁按上游 tag 分目录，历史 `v0.11.1`/`v0.12.1`/`v0.13.0`/`v0.14.1` 保留。

---

## 5. 假设与约束

- 上游 tag/分支在各仓库可用（本机已固化 `Branch_v1.5.0` / `Branch_v0.14.2`）。
- 本机已接鸿蒙真机（`3QC0226526001227`）与安卓真机（`cd3fac61`）→ 两端做真机部署与基本功能测试；
  测试遇问题**记录并继续**（自动模式，用户次日查看结果）。
- 桌面端测试可本机执行（构建/冒烟以独立进程跑，不杀自身）。
- 测试用的 DeepSeek API Key 由用户给定，**不落盘**。
- HOS/Ark 的 `build-profile.json5` 本地签名材料由 DevEco 自动签名写入 → **仅保留本地，不提交**。
