# 101 — 版本更新：0.14.2（version）跨工程技术方案与执行结果

> **Module:** 101-version-0.14.2 ｜ **Status:** Implemented ｜ **Last Updated:** 2026-11-04
> **对应规格:** [`spec.md`](./spec.md)
> **父/子分工:** 本文写跨工程执行步骤与结果；子工程的补丁、构建与提交在各子仓库。

---

## 1. 执行顺序

1. **冲突评估**：`pi-web` v0.14.1→v0.14.2 diff（68 文件）与 14 条补丁（0/2/6/6）的接触面比对 → **零漂移**。
2. **逐工程执行**：版本号 → 上游钉住 → 补丁目录 `v0.14.2/` → 构建 → 冒烟/产物 → 真机验证 → tag。
3. **父工程收尾**：spec/plan、submodule 指针、README 口径、能力/测试报告。

---

## 2. 各工程结果

### 2.1 pi-desktop（Desktop / Electron）
- 版本：`package.json` / `package-lock.json` → **0.14.2**。
- 上游：`ci.yml` / `release.yml` → `PI_REF=v1.5.0` / `PI_SHA=f9a26365568127461c47a07a37dcb42c140a3ac5`、
  `PI_WEB_REF=v0.14.2` / `PI_WEB_SHA=025e24e74fdbc8f46c4e0cf31c6d6b3fa6cf29d5`；`pin-upstream.mjs` baseline 更新；`upstream.lock.json` 本地刷新（gitignored）。
- 补丁：**0 条**（沿用）。
- 构建：`prepare-next`（pi-web `next build --webpack`）→ `collect-pi` → `smoke-desktop` **10 PASS / 0 FAIL** → `electron-forge make`。
- 产物：`Pi Desktop-0.14.2 Setup.exe`（379 MB）、`pi_desktop-0.14.2-full.nupkg`、`Pi Desktop-win32-x64-0.14.2.zip`。
- 提交/tag：`7c0caa0` + `v0.14.2`。
- **构建约束（本轮实测）**：`next build`（Next 16.4.4）默认堆不足 → **必须** `NODE_OPTIONS=--max-old-space-size=8192`；
  `collect-pi` 品牌化需 **ffmpeg 在 PATH**。

### 2.2 pi-desktop-hos（Electron on HarmonyOS）
- 版本：`AppScope/app.json5` → `versionCode 1402 / versionName 0.14.2`；`src-main/package.json`、`web_engine/.../resfile/resources/app/package.json` → 0.14.2。
- 补丁：`patches/v0.14.2/`（0001/0002 由 v0.14.1 **逐字节原样拷贝**）；`prepare-next.mjs` `PI_WEB_PATCH_TAG=v0.14.2` + `EXPECTED_PI_WEB_VERSION=0.14.2`。
- 构建：`collect-runtime`（全守卫 PASS）→ `prepare-next`（Node 22.17.0，2 补丁应用→build→回退 pristine）→ `smoke-node22` **9 PASS / 1 SKIP / 0 FAIL** → `collect-pi`（179.0 MB，SHA `4f840815…`）→ `collect-app` → `hvigorw assembleHap` → **签名 HAP**（`electron-default-signed.hap` 401.9 MB，pack.info `code=1402/name=0.14.2`）。
- 真机（`3QC0226526001227`）：安装 ✅ 启动 ✅ 进程 ✅ UI 渲染 ✅ 版本徽标 `pi-web 0.14.2 / pi 1.5.0` ✅；设置 DeepSeek Key ✅；选择沙箱工作区 ✅；**文本对话 `1+1=?` → `1+1=2`**（DeepSeek-V4-Pro，2s）✅。
- 图片分析：附件经系统文件选择器成功挂载（缩略图可见），但模型侧未收到图片内容 → 记录为 **HOS 附件桥待查项**（详见测试报告 §4）。
- 提交/tag：`2062168` + `v0.14.2`。
- **构建约束**：`DEVECO_SDK_HOME='D:\oh-workspace\command-line-tools-24\sdk'`；`hvigorw`=`D:\oh-workspace\command-line-tools-24\bin\hvigorw.bat`；签名走 `build-profile.json5` 内 DevEco 自动签名材料（**本地保留、未提交**）。

### 2.3 pi-desktop-ark（HarmonyOS 纯血版 / ArkWeb）
- 版本：`AppScope/app.json5` → `versionCode 1402 / versionName 0.14.2`。
- 补丁：`patches/v0.14.2/`（0001–0006 由 v0.14.1 原样拷贝）；`assemble-payload.mjs` `PI_WEB_PATCH_TAG=v0.14.2`。
- **布局修复**：`assemble-payload` 读取 `<bridge>/payload/`，对齐补齐 `vendor/node-bridge-payload/payload/{bootstrap.js,host.js,lib/tar-extract.js}`（与根目录 vendored 三件逐字节一致，已入库）。
- 构建：`assemble-payload --apply`（载荷源 `../pi-desktop-hos/pi-dist`，6 补丁 + jitless + 归档 83.5 MiB，SHA `fb38be4194b8…`）→ `hvigorw assembleHap` → **签名 HAP**（133.7 MB）。
- 真机：安装 ✅ 启动 ✅（main+gpu+render）UI 渲染 ✅ 版本徽标 `pi-web 0.14.2 / pi 1.5.0` ✅；设置 DeepSeek Key ✅；沙箱工作区 ✅；**文本对话 `1+1=?` → `1+1=2`** ✅。
- 图片分析：未测（Ark 服务 loopback 处独立 netns，hdc 端口转发/设备 shell 均不可达；附件桥同 HOS 形态，未单独复验）。
- 提交/tag：`414103e` + `v0.14.2`。

### 2.4 pi-desktop-android（Android / WebView）
- 版本：`app/build.gradle.kts` → `versionCode 1402 / versionName 0.14.2`。
- 运行时：`aosp-node-bridge 0.2.0` + `node-runtime 26.13.0`（不升级）。
- 补丁：`patches/v0.14.2/`（0001–0006 原样拷贝）；`assemble-payload.mjs` `PI_WEB_PATCH_TAG=v0.14.2`；`build-release.yml` `PI_WEB_TAG=v0.14.2`。
- **关键修正**：首轮 `build/pi-dist/.next` 残留 **v0.14.1** 旧构建，补丁标记与版本无关 → 误判「在位」跳过重编；已 `rm -rf build/pi-dist` 强制重建为 v0.14.2（BUILD_ID 11-04 02:01，标记 `PI_ANDROID_ROOTS`/`picker-contract`/`terminal.pty` 全在位）。
- 构建：`assemble-payload --apply` → `./gradlew :app:assembleDebug :app:assembleRelease` → **debug 189 MB / release 172 MB APK**（aapt `versionCode=1402 / versionName=0.14.2`）。
- 真机（`cd3fac61`）：安装 ✅ 启动 ✅（Node v26.13.0、PTY ready、JIT 启用）；`/api/web-auth={enabled:false,authenticated:true}`（免密）；**文本对话 617 字** ✅；**视觉图片对话（deepseek-flash）准确识别文字+四形状+布局+配色** ✅；工具链 bash/read/write/grep/ls **5/5** ✅；git init/add/commit/log ✅。
- 提交/tag：`3780810` + `v0.14.2`。

### 2.5 pi-desktop-website
- 两页 + README：版本/描述 → `v0.14.2` / `versionCode 1402`；上游 `pi v1.5.0 + pi-web v0.14.2`；补丁目录 `v0.14.2/`；变更记录新增 **v1.8**（历史行保留）。
- 提交/tag：`4884bef` + `v0.14.2`。

---

## 3. 验证汇总

| 检查 | Desktop | HOS | Ark | Android |
|---|---|---|---|---|
| 上游 tag/SHA | v1.5.0 / v0.14.2 ✅ | ✅ | ✅ | ✅ |
| 应用版本 | 0.14.2 | 0.14.2/1402 | 0.14.2/1402 | 0.14.2/1402 |
| 补丁数 & 幂等 | 0 | 2 ✅ | 6 ✅ | 6 ✅ |
| 构建 | Setup.exe + nupkg + zip | 签名 HAP 401.9MB | 签名 HAP 133.7MB | debug+release APK |
| 冒烟 | 10 PASS/0 FAIL | 9 PASS/1 SKIP/0 FAIL | 载荷归档自检通过 | aapt 版本校验通过 |
| 真机启动/渲染/版本 | —（本机） | ✅ | ✅ | ✅ |
| 文本对话 | ✅（API e2e） | ✅（UI） | ✅（UI） | ✅（UI/API） |
| 视觉图片对话 | ✅（API, flash） | ⚠️ 附件桥未送达（已记录） | ⬜ 未测（netns 限制） | ✅（API, flash） |
| 工具链 | ✅ bash+写入 | — | — | ✅ bash/read/write/grep/ls/git |
| tag | v0.14.2 | v0.14.2 | v0.14.2 | v0.14.2 |

---

## 4. 待办 / 已知限制

- **HOS 图片附件桥**：文件选择器挂载成功、UI 缩略图可见，但模型侧收到 `[Unsupported Image]`/空附件（`deepseek-v4-pro` 另见不支持图片输入提示）。**待查**：Electron 渲染层读取 SAF 文件的路径/权限，或上传编码环节。
- **Ark 未做图片与工具链真机验证**：其服务绑定 `127.0.0.1` 于应用独立 netns，`hdc fport` 与设备 `wget` 均不可达；只能经 UI 验证（文本对话已过）。
- 桌面 macOS / Linux 未本机构建（依赖 CI）。
- 鸿蒙两端 `build-profile.json5` 的 DevEco 自动签名材料**未提交**（按规范仅本地）。
- 测试用 DeepSeek API Key：仅运行期注入；**文档零落盘**；设备侧测试后清理（见测试报告 §5）。
