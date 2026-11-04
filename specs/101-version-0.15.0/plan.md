# 101 — 版本更新：0.15.0（version）跨工程技术方案与执行结果

> **Module:** 101-version-0.15.0 ｜ **Status:** Released（四端构建 + 桌面/鸿蒙/安卓真机验证 + 五子工程 tag v0.15.0 + 父工程收尾） ｜ **Last Updated:** 2026-11-04
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
6. **测试**：四端基本功能（DeepSeek Key / 沙箱工作区 / 文本对话 / 图片分析）+ 能力对照清单相关项。
   **范围变更**：原定「鸿蒙/安卓暂缓」在执行中被解除（用户接通两端真机），故四端均实测。敏感信息零落盘。

---

## 2. 各工程执行步骤与结果

> ✅=完成 ⚠️=有遗留/受限 ⬜=未做（下表为本轮**实测**结果）

### 2.1 pi-desktop（Desktop / Electron）

- 清理：`out/`、`.next/`、`pi-dist/`、`dist/` 等构建中间产物
- 版本：`package.json` / `package-lock.json` → **0.15.0**
- 上游：`ci.yml` / `release.yml` → `PI_WEB_REF=v0.15.0` / `PI_WEB_SHA=f130910abf662b506b2ef2f475fc21a2fe57badc`
  （`PI_REF=v1.5.0` / `PI_SHA=f9a26365…` 不变）；`pin-upstream.mjs` baseline 更新；`upstream.lock.json` 本地刷新
- 补丁：**0 条**（沿用）
- 构建：`prepare-next` → `collect-pi` → `smoke-desktop` → `electron-forge make`
- 结果：✅ **typecheck exit 0**；`prepare-next` BUILD_ID `ZKSPsW51TKpiiPpKaBoj4`（`--force-install`，见下）；**smoke 10 PASS / 0 FAIL**；打包态基本功能 e2e **11 PASS / 0 FAIL**；打包态 `probe-renderer` **7 PASS / 0 FAIL**；产物 `Setup.exe` / `nupkg` / `zip` 均 0.15.0
- **本轮定位的问题**：① `../pi-web/.next` 陈旧残留致 prerender `EPERM`（删后重建通过）；② `node_modules` 陈旧使产物内嵌 **next 16.3.8**（v0.15.0 锁文件要求 **16.4.4**）→ 必须 `--force-install`
- 证据：`pi-desktop/logs/20261104-1/TEST_REPORT.md`

### 2.2 pi-desktop-hos（Electron on HarmonyOS）

- 清理：`electron/build/`、`pi-dist/`、`web_engine/.../app/` 等中间产物
- 版本：`AppScope/app.json5` → `versionCode 1500 / versionName 0.15.0`；`src-main/package.json`、
  `web_engine/.../resfile/resources/app/package.json` → 0.15.0
- 补丁：`patches/v0.15.0/`（0001/0002 由 v0.14.2 逐字节原样拷贝，已 dry-run 验证）；`prepare-next.mjs`
  `PI_WEB_PATCH_TAG=v0.15.0` + `EXPECTED_PI_WEB_VERSION=0.15.0`
- 构建：`collect-runtime` → `prepare-next` → `smoke-node22` → `collect-pi` → `collect-app` → `hvigorw assembleHap`
- 结果：✅ collect-runtime 守卫全通过；`prepare-next` BUILD_ID `CQ3HkkwORYzhRx6oekSVX`（2 补丁应用→build→回退，pi-web 保持 pristine）；**smoke-node22 8 PASS / 2 SKIP / 0 FAIL**；`collect-pi` 归档 **153.5 MB**（SHA `4062e504…`）；签名 HAP **375,236,384 B**，`pack.info` = **code 1500 / name 0.15.0**
- **真机（`3QC0226526001227` / HAD-W32）**：安装 ✅ 启动 ✅（main+gpu+renderer+network）徽标 `web 0.15.0 · pi 1.5.0` ✅；DeepSeek Key ✅（沙箱 `auth.json`，重启后仍在）；沙箱工作区 `e2e-0150-test` ✅；**文本对话 ✅**；**图片分析对话 ✅**（会话 content 类型含 `image`，模型准确识别 PI 文字 + 红圆/蓝方/绿倒三角及叠压）；工具链 ✅（bash `pwd`/写读/`grep`/`ls` + 内置 git `init/add/commit/log`）
- ⚠️ 首次测试因**设备整体离线**（DNS `EAI_AGAIN`）报 `Connection error`，切网后同会话恢复正常 ⇒ 环境问题
- 证据：`pi-desktop-hos/docs/真机验证记录.md` 轮次 8

### 2.3 pi-desktop-ark（HarmonyOS 纯血版 / ArkWeb）

- 清理：`entry/build/`、`payload build 中间目录` 等
- 版本：`AppScope/app.json5` → `versionCode 1500 / versionName 0.15.0`
- 补丁：`patches/v0.15.0/`（0001–0006 原样拷贝）；`assemble-payload.mjs` `PI_WEB_PATCH_TAG=v0.15.0`
- 构建：`assemble-payload --apply`（载荷源 `../pi-desktop-hos/pi-dist`）→ `hvigorw assembleHap`
- 结果：✅ 载荷组装 patch→重编→标记断言→归档 **87.5 MB** 全绿，pi-web 还原 pristine；`hvigorw assembleHap` **BUILD SUCCESSFUL 32 s**；签名 HAP **133,776,606 B**，`pack.info` = **code 1500 / name 0.15.0**
- **真机**：安装 ✅ 启动 ✅（main+gpu+render）徽标 `web 0.15.0 · pi 1.5.0` ✅；设备用例通道 **`model` 套件 2/2 PASS**（真实 DeepSeek 非流式 + 流式 SSE，compat `shimFetch`/`WebAssembly=undefined`）；**`204-unit` 26 PASS / 0 FAIL**
- ⚠️ 图片对话本轮未在 Ark 单独复验（附件桥与 HOS 同形态，HOS 已验证）；⬜ 留待补测
- 证据：`pi-desktop-ark/docs/003-真机验证记录-0.15.0.md`

### 2.4 pi-desktop-android（Android / WebView）

- 清理：`build/pi-dist/`、`app/build/` 等（吸取 0.14.2 教训：陈旧 pi-dist 会导致版本误判，必须强制重建）
- 版本：`app/build.gradle.kts` → `versionCode 1500 / versionName 0.15.0`
- 补丁：`patches/v0.15.0/`（0001–0006 原样拷贝）；`assemble-payload.mjs` `PI_WEB_PATCH_TAG=v0.15.0`；
  `build-release.yml` `PI_WEB_TAG=v0.15.0`
- 构建：`assemble-payload --apply` → `./gradlew :app:assembleDebug :app:assembleRelease` → aapt 版本校验
- 结果：✅ 载荷组装（归档自检 **34881 文件**，pi-web 还原 pristine）；gradle **BUILD SUCCESSFUL 3m21s**；`app-debug.apk` **183,033,670 B** / `app-release.apk` **165,849,568 B**；aapt 两包均 **1500 / 0.15.0**
- **真机（`cd3fac61` / MIX / Android 8.0.0）**：旧签名冲突需先 `uninstall` → 安装 ✅ 启动 ✅ UI 渲染 ✅ 徽标 `web v0.15.0 · pi v1.5.0` ✅；`/api/web-auth` 免密 ✅；DeepSeek Key ✅；**沙箱工作区 `e2e-0150-test` ✅**（经 `POST /api/cwd/browse`，`roots`/`contract` 契约在位）；**文本对话 ✅**（9 s）；**图片分析对话 ✅**（切 **V4.1 Flash**；V4 Pro 不支持图片，模型侧回 `image omitted`）；工具链 ✅（bash `pwd`/写读/`grep`/`ls`；bash 内无 git 二进制，**内置 git 工具** `init/add/commit/log` 全部成功）
- ⚠️ 构建需 `JAVA_HOME=D:\Java\JDK`（Temurin 21）；Studio JBR（JDK 25）下 Gradle 8.12 报 `What went wrong: 25.0.3`
- 证据：`pi-desktop-android/docs/002-真机验证记录.md`（0.15.0 轮）

### 2.5 pi-desktop-website

- 两页 + README：版本/描述 → `v0.15.0` / `versionCode 1500`；上游 `pi v1.5.0 + pi-web v0.15.0`；
  补丁目录 `v0.15.0/`；变更记录新增 **v1.9**（历史行保留）
- 结果：✅ README 43 行 / 两页各 20 行改写，逐项计数校验（`0.14.x` 仅保留在历史条目）；HOS 平台卡产物类型由「APK」纠正为 **签名 HAP**

---

## 3. 验证汇总

> 执行完成后回填。

| 检查 | Desktop | HOS | Ark | Android |
|---|---|---|---|---|
| 上游 tag/SHA | v1.5.0 / v0.15.0 ✅ | ✅ | ✅ | ✅ |
| 应用版本 | 0.15.0 ✅ | 0.15.0/1500 ✅ | 0.15.0/1500 ✅ | 0.15.0/1500 ✅ |
| 补丁数 & 幂等 | 0 | 2 ✅ | 6 ✅ | 6 ✅ |
| 构建 | Setup.exe/nupkg/zip ✅ | 签名 HAP 375.2 MB ✅ | 签名 HAP 133.8 MB ✅ | debug+release APK ✅ |
| 冒烟 | 10 PASS + e2e 11 + probe 7 ✅ | smoke-node22 8P/2S/0F ✅ | 载荷标记断言 + model 2/2 + 204-unit 26/0 ✅ | aapt 版本校验 + 载荷自检 ✅ |
| 真机 | —（本机打包态） | 启动/渲染/徽标/Key/工作区/文本/图片/工具链 ✅ | 启动/渲染/徽标/model 2/2/204-unit ✅（图片对话 ⬜） | 启动/渲染/徽标/Key/工作区/文本/图片/工具链 ✅ |
| tag | v0.15.0 ✅ | v0.15.0 ✅（a85c70d） | v0.15.0 ✅（9be7c15） | v0.15.0 ✅（8b48e38） |

---

## 4. 测试（四端实测；证据见各工程记录）

- 基本功能：设置 DeepSeek API Key（仅运行期注入）→ 创建沙箱工作区 → 文本对话「介绍一下你自己」→ 图片分析对话
- 环境能力：按 `docs/能力对照清单.md` 执行相关用例
- 敏感信息：测试 URL / 用户 / 密码 / 凭证**零落盘**；测试后清理运行期注入的 Key
- 结果：✅ 详见 §2 各端结果与四份记录；环境问题两处（HOS 设备离线、Android 设备无公网出口）均在切网后恢复

---

## 5. 待办 / 已知限制

- Ark 图片对话待补测（⬜）；本轮鸿蒙两端均已完成构建与真机基本验证。
- 五子工程 tag `v0.15.0` 与父工程收尾（submodule 指针、README 中英同步）已完成；官网 tag `v0.15.0`（6b19bbe）。
- HOS/Ark 的 `build-profile.json5` DevEco 自动签名材料仅本地保留、不提交（延续 0.14.2 口径）。
