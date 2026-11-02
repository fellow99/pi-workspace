# 101 — 版本更新：0.14.1（version）跨工程功能规格

> **Module:** version ｜ **Status:** Implemented ｜ **Last Updated:** 2026-11-02
> **上游基线:** `pi` v1.4.0（`4a080252f06caf63a525d7873c5d94d3c23887e0`）· `pi-web` v0.14.1（`506168bf11dc444c02aea9cf9c85ca63755b8e0f`）
> **发布 tag:** `v0.14.1`
> **适用范围:** `pi-desktop` · `pi-desktop-hos` · `pi-desktop-ark` · `pi-desktop-android` · `pi-desktop-website`
> **父/子分工:** 本文写**跨工程版本口径与矩阵**；各子工程的升级步骤、补丁、构建证据写在各自提交与 `patches/v0.14.1/`。

---

## 1. 模块概述

### 1.1 目的

把五个工程（Desktop / HOS / Ark / Android / Website）统一收敛到 `v0.14.1` 基线，并把上游基线升到
`pi` v1.4.0 + `pi-web` v0.14.1，回答三个问题：

1. **上游版本**是什么？（`pi` / `pi-web` 的 tag 与 commit）
2. **应用版本**是什么？（各壳的 `versionName` / `versionCode`）
3. **发布 tag** 是什么？（`v0.14.1`，只作发布标识）

### 1.2 解决的问题

- pi-web v0.14.1（2026-11 发布）相对 v0.13.0 的变更：**侧对话**（`/side` fork）、**Quote**（选区
  blockquote 引用）、**favicon 角标**（会话需注意时）、**File viewer**（散文默认换行 / Git changes
  Tree/List 视图）、**Docker 支持**、以及若干修复（models provider 选择器、i18n 本地化、subagents、
  窄屏 chat 布局、sandboxed HTML preview 存储）。
- **冲突评估**：以上均为纯前端特性，不涉宿主桥、运行时或内核 → 对四端**无明显行为冲突**；
  唯一需要处理的是 **`components/DirectoryPicker.tsx` 在 v0.14.1 有实质改动**——`#1462`
  `fix(directory-picker): keep the picker open when a text drag ends outside` 新增 backdrop 按下追踪
  （`backdropPressRef` / `createBackdropPressRef`）并把 `canSelect` 并入 `!loading`、移除 `canCreate`。
  该文件是 HOS 0001 / ark/android 0005/0006 的主战场 ⇒ 三条补丁链共 5 条补丁**上下文漂移**，
  必须**重新生成**（见 §3）。
- **保持上游修复语义**：重出的补丁**保留** v0.14.1 的 backdrop 按下追踪（不因我们的补丁回退上游 bugfix），
  在其之上重放我们的目录选择器逻辑。

### 1.3 范围

**包含**：五版上游基线一致性；应用版本 / versionCode / 产物 / tag 矩阵；运行时矩阵（Android Node 26.13.0）；
补丁集与版本目录（`patches/v0.14.1/`）；HOS 0001 与 Ark/Android 0005/0006 共 5 条补丁在 v0.14.1 上的
重出与幂等验证；网站 README/页面更新；发布 tag。

**不包含**：修改上游源码（只允许幂等补丁）；真机验证（本机无鸿蒙真机，鸿蒙两端跳过测试；Android 无设备连接）。

---

## 2. 版本矩阵

### 2.1 上游

| 项 | 版本 | Commit |
|---|---|---|
| `pi` | **v1.4.0** | `4a080252f06caf63a525d7873c5d94d3c23887e0` |
| `pi-web` | **v0.14.1** | `506168bf11dc444c02aea9cf9c85ca63755b8e0f` |

### 2.2 五版应用与运行时

| 版别 | 应用版本 | versionCode | 运行时 | Node | 补丁 | 构建/产物 |
|---|---|---|---|---|---|---|
| `pi-desktop` | **0.14.1** | —（package.json） | Electron 43.7.9 | 24.21.0 | 0 | Setup.exe + zip（Windows），smoke 全绿 |
| `pi-desktop-hos` | **0.14.1** | **1401** | Electron-on-HarmonyOS 37.2.3 | 22.17.0 | 2 | unsigned HAP（无签名材料；无真机） |
| `pi-desktop-ark` | **0.14.1** | **1401** | node-bridge 0.1.0 + node-runtime 24.2.0 | 24.2.0 | 6 | unsigned HAP（无签名材料；无真机） |
| `pi-desktop-android` | **0.14.1** | **1401** | aosp-node-bridge 0.2.0 + node-runtime **26.13.0** | **26.13.0** | 6 | debug + release APK |
| `pi-desktop-website` | **0.14.1** | — | — | — | — | README + index.html + index_zh.html |

> **versionCode 编码**：`major*10000 + minor*100 + patch` ⇒ 0.14.1 → **1401**（HOS/Ark/Android 三端一致）。
> **Android 运行时**：aosp-node-bridge 保持 `0.2.0`，aosp-node-runtime 保持 `26.13.0`（本轮不升级）。

### 2.3 发布 tag 与产物

| 版别 | tag | 产物 |
|---|---|---|
| Desktop | `v0.14.1` | `out/make/…/Pi Desktop-0.14.1 Setup.exe`、`Pi Desktop-win32-x64-0.14.1.zip` |
| HOS | `v0.14.1` | `electron/build/…/electron-default-unsigned.hap` |
| Ark | `v0.14.1` | `entry/build/…/entry-default-unsigned.hap` |
| Android | `v0.14.1` | `app-debug.apk`、`app-release.apk`（versionCode 1401 / versionName 0.14.1） |
| Website | `v0.14.1` | README + 两页静态站点 |

---

## 3. 冲突评估（pi-web 0.13.0 → 0.14.1）

| 关注点 | 评估 | 结论 |
|---|---|---|
| 功能特性（side chat / Quote / favicon / File viewer / Docker 等） | 纯前端 UI，不涉宿主桥/运行时 | 无冲突 |
| 运行时 Node `>=22.19.0`（engines 未变） | Electron 24.x / host Node 22.17（HOS，smoke 门禁降级） / Android 26.13.0 | 兼容 |
| `app/api/agent/new/route.ts` 等补丁 0003 相关文件 | v0.13.0 → v0.14.1 **无变更** | ✅ 干净沿用 |
| ark/android `patches/v0.13.0/` 0001/0002/0003/0004 | 在 v0.14.1 上按序应用干净 | ✅ 原样拷贝为 `v0.14.1/` |
| ark/android `patches/v0.13.0/` 0005/0006 | **漂移**：`DirectoryPicker.tsx` #1462 backdrop 改动 | ⚠️ **重出**（保住 #1462） |
| hos `patches/v0.13.0/` 0001-workspace-picker-roots | **漂移**：`DirectoryPicker.tsx` #1462 backdrop 改动 | ⚠️ **重出**（保住 #1462） |
| hos `patches/v0.13.0/` 0002-session-export | 涉及文件 v0.13.0→v0.14.1 无变更 | ✅ `v0.14.1/` 同内容副本 |

### 3.1 补丁重出方法（实测证据）

对 ark / android / hos，在 `pi-web@v0.14.1` **独立 detached worktree** 上重出：

```
git worktree add --detach <scratch> v0.14.1
# ark/android：先顺次应用拷贝版 0001-0004（干净）；hos：先应用 0001 非 DirectoryPicker 部分（干净）
# 再以「保留 v0.14.1 backdropPressRef/createBackdropPressRef + canSelect(含 !loading)」为前提，
# 手工在 v0.14.1 的 DirectoryPicker.tsx 上重放：
#   hos 0001：工作区主目录 Tab + 面包屑 + 权限归因（EACCES/EPERM → /api/cwd/workspace-permission）；
#   ark/and 0005：主目录 Tab 契约（roots/rootId/truncated）+ selectRoot + localStorage；
#   ark/and 0006：BrowseError + /api/host/perm 权限流 + "请求权限/刷新"。
# 逐条 git add + commit，git diff <前一 stage> 生成补丁；
# 全栈在 pristine v0.14.1 上按序 `git apply --check` + 应用 + 反向往返回 pristine，均 PASS。
```

- **ark 0005/0006**（`patches/v0.14.1/`）与 android **0005** 的唯一差异是
  `normalizeHosSandboxPath`/`@/lib/hos-sandbox-path`/`PI_HOS_ROOTS` ↔
  `normalizeAndroidSandboxPath`/`@/lib/android-sandbox-path`/`PI_ANDROID_ROOTS`（sed 替换派生）；
  android **0006** 与 ark **0006** 逐字节相同（同上游文件，无平台差异）。
- **验证汇总**：ark 0001-0006 / android 0001-0006 / hos 0001-0002 三条链在 pristine v0.14.1 上
  **按序 `git apply` 全 PASS**，**反向 6→1/2→1 往返回 pristine 全 PASS**；
  合并后 `DirectoryPicker.tsx` 经 TS transpile 语法校验 PASS，`backdropPressRef`/`createBackdropPressRef`
  各保留 6 处（v0.14.1 修复语义在位），`canSelect` 行保留 `&& !loading`。

---

## 4. 非功能

- **可复现**：同一 tag + 锁文件产出等价包。
- **可追溯**：tag → 上游 SHA → 运行时 → 产物。
- **安全**：API Key / 签名 / 测试凭证不落盘。
- **可维护**：补丁按上游 tag 分目录，历史 `v0.11.1`/`v0.12.1`/`v0.13.0` 保留。

---

## 5. 假设与约束

- 上游 tag/分支在各仓库可用。
- 本机无鸿蒙真机 → HOS / Ark 只构建（unsigned HAP），**跳过真机测试**（用户指示）。
- 桌面端测试可本机执行（本智能体即 Pi Desktop Windows 版，构建/冒烟以独立进程跑，不杀自身）。
- 测试用的 DeepSeek API Key 由用户给定，**不落盘**。
