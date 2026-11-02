# 101 — 版本更新：0.13.0（version）跨工程功能规格

> **Module:** version ｜ **Status:** Implemented ｜ **Last Updated:** 2026-11-02
> **上游基线:** `pi` v1.4.0（`4a080252f06caf63a525d7873c5d94d3c23887e0`）· `pi-web` v0.13.0（`d128bae47cd3abbdfe7135443ac793f547c14e47`）
> **发布 tag:** `v0.13.0`
> **适用范围:** `pi-desktop` · `pi-desktop-hos` · `pi-desktop-ark` · `pi-desktop-android` · `pi-desktop-website`
> **父/子分工:** 本文写**跨工程版本口径与矩阵**；各子工程的升级步骤、补丁、构建证据写在各自提交与 `patches/v0.13.0/`。

---

## 1. 模块概述

### 1.1 目的

把五个工程（Desktop / HOS / Ark / Android / Website）统一收敛到 `v0.13.0` 基线，并把上游基线升到
`pi` v1.4.0 + `pi-web` v0.13.0，回答三个问题：

1. **上游版本**是什么？（`pi` / `pi-web` 的 tag 与 commit）
2. **应用版本**是什么？（各壳的 `versionName` / `versionCode`）
3. **发布 tag** 是什么？（`v0.13.0`，只作发布标识）

### 1.2 解决的问题

- pi-web v0.13.0（2026-10-29 发布）引入了 Debug 面板、工具参数按行显示、svg 预览、公式复制、音视频内联播放、
  压缩排队、Agents 徽标与模型设置增强（Base URL 覆盖 / adaptive thinking / 图片大小上限 / provider id 搜索）。
- **冲突评估**：以上均为纯前端特性，不涉及宿主桥、运行时或内核 → 对四端**无明显冲突**；
  唯一需要处理的是 ark/android 的 `patches/v0.12.1/` 中 3 条（0003/0005/0006）在 v0.13.0 上
  **上下文漂移**（上游在 `app/api/agent/new/route.ts` 引入 `lib/missing-cwd.ts`，把 `error`
  改写成 `...cwdMissingBody(cwd)`），必须**重新生成**适配。

### 1.3 范围

**包含**：五版上游基线一致性；应用版本 / versionCode / 产物 / tag 矩阵；运行时矩阵（含 Android Node 26.13.0）；
补丁集与版本目录（`patches/v0.13.0/`）；Ark/Android 的 6 条补丁在 v0.13.0 上的重新生成与幂等验证；网站 README/页面更新；发布 tag。

**不包含**：修改上游源码（只允许幂等补丁）；真机验证（本机无鸿蒙真机，鸿蒙两端跳过测试）。

---

## 2. 版本矩阵

### 2.1 上游

| 项 | 版本 | Commit |
|---|---|---|
| `pi` | **v1.4.0** | `4a080252f06caf63a525d7873c5d94d3c23887e0` |
| `pi-web` | **v0.13.0** | `d128bae47cd3abbdfe7135443ac793f547c14e47` |

### 2.2 五版应用与运行时

| 版别 | 应用版本 | versionCode | 运行时 | Node | 补丁 | 构建/产物 |
|---|---|---|---|---|---|---|
| `pi-desktop` | **0.13.0** | —（package.json） | Electron 43.7.9 | 24.21.0 | 0 | Setup.exe + zip（Windows），smoke 10 PASS / 0 FAIL |
| `pi-desktop-hos` | **0.13.0** | **1300** | Electron-on-HarmonyOS 37.2.3 | 22.17.0 | 2 | unsigned HAP（无签名材料；无真机） |
| `pi-desktop-ark` | **0.13.0** | **1300** | node-bridge 0.1.0 + node-runtime 24.2.0 | 24.2.0 | 6 | unsigned HAP（无签名材料；无真机） |
| `pi-desktop-android` | **0.13.0** | **1300** | aosp-node-bridge 0.2.0 + node-runtime **26.13.0** | **26.13.0** | 6 | debug + release APK |
| `pi-desktop-website` | **0.13.0** | — | — | — | — | README + index.html + index_zh.html |

> **Android 运行时升级**：aosp-node-bridge 保持 `0.2.0`，aosp-node-runtime 从 `24.22.1` 升到 **`26.13.0`**（small-ICU）；
> `runtime.lock.json`、`gradle/libs.versions.toml` 在上一轮已对齐（`libs.versions.toml` nodeRuntime=26.13.0）；
> 本次 workflow `build-release.yml` 的 `PDA_NODE_RUNTIME_VER` 同步为 `26.13.0`。

### 2.3 发布 tag 与产物

| 版别 | tag | 产物 |
|---|---|---|
| Desktop | `v0.13.0` | `out/make/…/Pi Desktop-0.13.0 Setup.exe`、`Pi Desktop-win32-x64-0.13.0.zip` |
| HOS | `v0.13.0` | `electron/build/…/electron-default-unsigned.hap` |
| Ark | `v0.13.0` | `entry/build/…/entry-default-unsigned.hap` |
| Android | `v0.13.0` | `app-debug.apk`、`app-release.apk`（versionCode 1300 / versionName 0.13.0） |
| Website | `v0.13.0` | README + 两页静态站点 |

---

## 3. 冲突评估（pi-web 0.12.1 → 0.13.0）

| 关注点 | 评估 | 结论 |
|---|---|---|
| 功能特性（Debug 面板 / svg / 公式复制 / 音视频 / 模型设置等） | 纯前端 UI，不涉宿主桥/运行时 | 无冲突 |
| 运行时 Node ≥22.19.0 | Electron 24.x / host Node 22.17（HOS，smoke 门禁降级） / Android 26.13.0 | 兼容 |
| ark/android `patches/v0.12.1/` 0001/0002/0004 | 在 v0.13.0 上应用干净 | ✅ 原样沿用 |
| ark/android `patches/v0.12.1/` 0003（cwd 路径别名归一） | v0.13.0 `agent/new/route.ts` 改为 `...cwdMissingBody()` → 上下文漂移 | ⚠️ **重新生成** |
| ark/android `patches/v0.12.1/` 0005/0006（目录选择器契约 / 权限流） | 依赖 0003 的链式上下文 | ⚠️ **随 0003 重出后干净应用** |
| hos `patches/v0.12.1/` 0001/0002 | 涉及文件 v0.12.1→v0.13.0 无变更 | ✅ `patches/v0.13.0/` 同内容副本，幂等验证通过 |

### 3.1 补丁重出方法

对 ark / android：
```
git worktree add <tmp> v0.13.0
git apply 0001, 0002            # （v0.12.1 版，干净）
git commit -m base
git apply --reject 0003         # 仅 agent/new/route.ts reject
# 手动合并：`...cwdMissingBody(cwd)` 后补 `hint: describePathForms(...)`，并补齐 hos/android-sandbox-path import
git add -A && git diff --cached > patches/v0.13.0/0003-…
# 顺序验证 0001..0006 全部 `git apply` OK
```

---

## 4. 非功能

- **可复现**：同一 tag + 锁文件产出等价包。
- **可追溯**：tag → 上游 SHA → 运行时 → 产物。
- **安全**：API Key / 签名 / 测试凭证不落盘。
- **可维护**：补丁按上游 tag 分目录，历史 `v0.11.1`/`v0.12.1` 保留。

---

## 5. 假设与约束

- 上游 tag/分支在各仓库可用。
- 本机无鸿蒙真机 → HOS / Ark 只构建（unsigned HAP），**跳过真机测试**（用户指示）。
- 桌面端测试可本机执行（本智能体即 Pi Desktop Windows 版，构建/冒烟以独立进程跑，不杀自身）。
- 测试用的 DeepSeek API Key 由用户给定，**不落盘**。
