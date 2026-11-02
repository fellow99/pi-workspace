# 101 — 版本更新：0.14.1（version）跨工程技术方案

> **Module:** 101-version-0.14.1 ｜ **Status:** Implemented ｜ **Last Updated:** 2026-11-02
> **对应规格:** [`spec.md`](./spec.md)
> **父/子分工:** 本文写跨工程执行步骤与结果；子工程的补丁、构建与提交在各子仓库。

---

## 1. 执行顺序

1. **补丁冲突评估**：把 `patches/v0.13.0/*.patch` 按序应用到 pi-web v0.14.1，定位漂移补丁（见 spec §3）。
2. **重新生成补丁**：hos / ark / android 各重建 `patches/v0.14.1/`（hos 0001、ark/and 0005/0006 重出，
   其余拷贝），在 pristine v0.14.1 上逐条 `git apply` 幂等 + 往返回 pristine 验证。
3. **逐工程执行**：版本号 → 补丁目录/tag → 构建 → 冒烟/产物 → tag。
4. **父工程收尾**：spec 编写、submodule 指针更新、能力清单/README 口径回写。

---

## 2. 各工程结果

### 2.1 pi-desktop（Desktop / Electron）

- 版本：`package.json` / `package-lock.json` → `0.14.1`。
- 上游：`ci.yml` / `release.yml` `PI_REF=v1.4.0` `PI_SHA=4a080252f…`、`PI_WEB_REF=v0.14.1` `PI_WEB_SHA=506168b…`；
  `pin-upstream.mjs` baseline → `v1.4.0 (Branch_v1.4.0)` + `v0.14.1 (Branch_v0.14.1)`；`upstream.lock.json` 刷新。
- 补丁：无（0 条，沿用）。
- 构建：`npm ci` → `prepare-next`（pi-web next build）→ `collect-pi`（pi-dist.tar.gz）→
  `smoke-desktop` **全 PASS** → `electron-forge make`（`Pi Desktop-0.14.1 Setup.exe` + zip）。
- 中间产物清理：`out/ pi-dist* .vite` 清空后重建。
- 提交 + tag `v0.14.1`。

### 2.2 pi-desktop-hos（Electron on HarmonyOS）

- 版本：`AppScope/app.json5` `versionCode 1401 / versionName 0.14.1`；`src-main/package.json` → `0.14.1`；
  `web_engine/…/resources/app/package.json` → `0.14.1`。
- 补丁：`patches/v0.14.1/`（0001 重出 + 0002 拷贝），`prepare-next.mjs` `PI_WEB_PATCH_TAG=v0.14.1`；
  pristine 幂等验证通过。
- 构建：`collect-runtime`（全部守卫 PASS，注入 libc++_shared）→ `prepare-next`（2 条补丁应用 + next build + 回退 pristine）
  → `smoke-node22`（Node 门禁降级）→ `collect-pi`（pi-dist.tar.gz）→ `collect-app` → `hvigorw assembleHap`
  → **unsigned HAP**（无签名材料）。
- 真机：**跳过（无鸿蒙真机）**。
- 提交 + tag `v0.14.1`。

### 2.3 pi-desktop-ark（HarmonyOS 纯血版 / ArkWeb）

- 版本：`AppScope/app.json5` `versionCode 1401 / versionName 0.14.1`。
- 补丁：`patches/v0.14.1/`（0001-0004 拷贝、0005/0006 因 #1462 backdrop 重出），
  `assemble-payload.mjs` `PI_WEB_PATCH_TAG=v0.14.1`。
- 构建：`ohpm install`（拉 node-bridge 0.1.0 + node-runtime 24.2.0）→ `assemble-payload --apply`
  （6 条补丁按序应用 → next build → 裁剪/jitless → 归档 → 回退 pristine）→ `hvigorw assembleHap`
  → **unsigned HAP**。
- 真机：**跳过（无鸿蒙真机）**。
- 提交 + tag `v0.14.1`。

### 2.4 pi-desktop-android（Android / WebView）

- 版本：`app/build.gradle.kts` `versionCode 1401 / versionName 0.14.1`。
- 运行时：aosp-node-bridge **0.2.0**（不变）+ aosp-node-runtime **26.13.0**（不变）；
  `libs.versions.toml` / `runtime.lock.json` 已对齐；`build-release.yml`
  `PDA_NODE_RUNTIME_VER=26.13.0`、`PI_WEB_TAG=v0.14.1`。
- 补丁：`patches/v0.14.1/`（0001-0004 拷贝、0005 派生自主 ark + android 化替换、0006 与 ark 相同），
  `assemble-payload.mjs` `PI_WEB_PATCH_TAG=v0.14.1`。
- 构建：`assemble-payload --apply`（6 条补丁 → next build → 归档 → 回退 pristine）→ `gradlew :app:assembleDebug`
  → **debug + release APK**（versionCode 1401 / versionName 0.14.1）。
- 提交 + tag `v0.14.1`。

### 2.5 pi-desktop-website

- 版本：README + `index.html` / `index_zh.html` 全部 `v0.14.1`；Android Node `26.13.0`；HOS/Ark versionCode `1401`；
  上游 `pi v1.4.0 + pi-web v0.14.1`；补丁数 0/2/6/6；发布 tag `v0.14.1`；changelog 增 v1.7。
- 提交 + tag `v0.14.1`。

---

## 3. 验证汇总

| 检查 | Desktop | HOS | Ark | Android |
|---|---|---|---|---|
| 上游 tag/SHA | v1.4.0 / v0.14.1 ✅ | ✅ | ✅ | ✅ |
| 应用版本 | 0.14.1 | 0.14.1/1401 | 0.14.1/1401 | 0.14.1/1401 |
| 运行时 | Electron 43.7.9 | Electron 37.2.3 | bridge 0.1.0 / rt 24.2.0 | bridge 0.2.0 / rt **26.13.0** |
| 补丁数 | 0 | 2 | 6 | 6 |
| 补丁 v0.14.1 幂等 | — | ✅（0001 重出） | ✅（0005/0006 重出） | ✅（0005 派生/0006 重出） |
| #1462 backdrop 保留 | — | ✅ | ✅ | ✅ |
| 构建 | Setup.exe + zip | unsigned HAP | unsigned HAP | APK（debug+release） |
| 冒烟 | 全 PASS | smoke-node22 PASS/SKIP | （payload 归档自检通过） | APK 产出 |
| 真机 | — | ⏸ 跳过（无真机） | ⏸ 跳过（无真机） | —（无设备） |
| tag | v0.14.1 | v0.14.1 | v0.14.1 | v0.14.1 |

---

## 4. 待办 / 已知限制

- 鸿蒙两端（HOS / Ark）**未做真机测试**（本机无真机），HAP 为 unsigned（无签名材料）。
- Android 未做设备端会话/对话测试（无设备连接）。
- 桌面端（Windows）冒烟/端到端验证通过；macOS / Linux 未在本机构建（依赖 CI）。
- 测试用 DeepSeek API Key 未落盘文档。
