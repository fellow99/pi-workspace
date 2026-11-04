# 101 — 版本更新：0.15.0（version）跨工程技术方案与执行结果

> **Module:** 101-version-0.15.0 ｜ **Status:** In Progress ｜ **Last Updated:** 2026-11-05
> **对应规格:** [`spec.md`](./spec.md)
> **父/子分工:** 本文写跨工程执行步骤与结果；子工程的补丁、构建与提交在各子仓库。

---

## 1. 执行顺序

1. **前置修正**：五个子工程本地 main 停留在旧开发提交（落后 origin/main 7~43 个提交），已按
   `git merge --ff-only origin/main` 快进到 v0.14.2 发布线顶端（不换分支）。
2. **冲突评估**：`pi-web` v0.14.2→v0.15.0 diff（75 文件）与 14 条补丁（0/2/6/6）的接触面比对 +
   **累计顺序 dry-run 实测** → **零漂移**（详见 spec §3）。
3. **逐工程执行**：清理中间产物 → 版本号 → 上游钉住（pi-web v0.15.0 + SHA）→ 补丁目录 `v0.15.0/` →
   构建 → 冒烟/产物 → 提交。
4. **Code Review**（requesting-code-review / receiving-code-review）→ 修复 → **五个子工程打 tag v0.15.0**。
5. **父工程收尾**：spec/plan、submodule 指针、README 口径 → 提交 + tag v0.15.0。
6. **测试**：桌面端基本功能（DeepSeek Key / 工作区 / 文本对话 / 图片分析）+ 能力对照清单；
   鸿蒙 / 安卓暂缓（用户指定）。敏感信息零落盘。

---

## 2. 各工程执行步骤与结果

> 结果列在执行完成后回填；⬜=待执行 ✅=完成 ⚠️=有遗留

### 2.1 pi-desktop（Desktop / Electron）

- 清理：`out/`、`.next/`、`pi-dist/`、`dist/` 等构建中间产物
- 版本：`package.json` / `package-lock.json` → **0.15.0**
- 上游：`ci.yml` / `release.yml` → `PI_WEB_REF=v0.15.0` / `PI_WEB_SHA=f130910abf662b506b2ef2f475fc21a2fe57badc`
  （`PI_REF=v1.5.0` / `PI_SHA=f9a26365…` 不变）；`pin-upstream.mjs` baseline 更新；`upstream.lock.json` 本地刷新
- 补丁：**0 条**（沿用）
- 构建：`prepare-next` → `collect-pi` → `smoke-desktop` → `electron-forge make`
- 结果：⬜

### 2.2 pi-desktop-hos（Electron on HarmonyOS）

- 清理：`electron/build/`、`pi-dist/`、`web_engine/.../app/` 等中间产物
- 版本：`AppScope/app.json5` → `versionCode 1500 / versionName 0.15.0`；`src-main/package.json`、
  `web_engine/.../resfile/resources/app/package.json` → 0.15.0
- 补丁：`patches/v0.15.0/`（0001/0002 由 v0.14.2 逐字节原样拷贝，已 dry-run 验证）；`prepare-next.mjs`
  `PI_WEB_PATCH_TAG=v0.15.0` + `EXPECTED_PI_WEB_VERSION=0.15.0`
- 构建：`collect-runtime` → `prepare-next` → `smoke-node22` → `collect-pi` → `collect-app` → `hvigorw assembleHap`
- 结果：⬜

### 2.3 pi-desktop-ark（HarmonyOS 纯血版 / ArkWeb）

- 清理：`entry/build/`、`payload build 中间目录` 等
- 版本：`AppScope/app.json5` → `versionCode 1500 / versionName 0.15.0`
- 补丁：`patches/v0.15.0/`（0001–0006 原样拷贝）；`assemble-payload.mjs` `PI_WEB_PATCH_TAG=v0.15.0`
- 构建：`assemble-payload --apply`（载荷源 `../pi-desktop-hos/pi-dist`）→ `hvigorw assembleHap`
- 结果：⬜

### 2.4 pi-desktop-android（Android / WebView）

- 清理：`build/pi-dist/`、`app/build/` 等（吸取 0.14.2 教训：陈旧 pi-dist 会导致版本误判，必须强制重建）
- 版本：`app/build.gradle.kts` → `versionCode 1500 / versionName 0.15.0`
- 补丁：`patches/v0.15.0/`（0001–0006 原样拷贝）；`assemble-payload.mjs` `PI_WEB_PATCH_TAG=v0.15.0`；
  `build-release.yml` `PI_WEB_TAG=v0.15.0`
- 构建：`assemble-payload --apply` → `./gradlew :app:assembleDebug :app:assembleRelease` → aapt 版本校验
- 结果：⬜

### 2.5 pi-desktop-website

- 两页 + README：版本/描述 → `v0.15.0` / `versionCode 1500`；上游 `pi v1.5.0 + pi-web v0.15.0`；
  补丁目录 `v0.15.0/`；变更记录新增 **v1.9**（历史行保留）
- 结果：⬜

---

## 3. 验证汇总

> 执行完成后回填。

| 检查 | Desktop | HOS | Ark | Android |
|---|---|---|---|---|
| 上游 tag/SHA | ⬜ | ⬜ | ⬜ | ⬜ |
| 应用版本 | ⬜ | ⬜ | ⬜ | ⬜ |
| 补丁数 & 幂等 | 0 | ⬜ | ⬜ | ⬜ |
| 构建 | ⬜ | ⬜ | ⬜ | ⬜ |
| 冒烟 | ⬜ | ⬜ | ⬜ | ⬜ |
| tag | ⬜ | ⬜ | ⬜ | ⬜ |

---

## 4. 测试（桌面端全量；鸿蒙/安卓暂缓）

- 基本功能：设置 DeepSeek API Key（仅运行期注入）→ 创建沙箱工作区 → 文本对话「介绍一下你自己」→ 图片分析对话
- 环境能力：按 `docs/能力对照清单.md` 执行相关用例
- 敏感信息：测试 URL / 用户 / 密码 / 凭证**零落盘**；测试后清理运行期注入的 Key
- 结果：⬜

---

## 5. 待办 / 已知限制

- 鸿蒙 / 安卓真机测试暂缓（用户指定），下一轮补测。
- HOS/Ark 的 `build-profile.json5` DevEco 自动签名材料仅本地保留、不提交（延续 0.14.2 口径）。
