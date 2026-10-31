# TECH.md — Pi Desktop Workspace 技术选型

> 记录父工程与四个子工程的技术栈、锁定版本、选型理由与兼容风险。
> 版本以源码/锁文件为准（2026-10-31 扫描）；冲突时以各子工程 `package.json` / `oh-package.json5` /
> `gradle/libs.versions.toml` / `app.json5` / `runtime.lock.json` 为准。

---

## 1. 技术栈总表

### 1.1 上游与被复用组件

| 类别 | 技术 | 版本 | 用途 |
|---|---|---|---|
| Agent harness | `pi`（TypeScript / Node） | **v1.2.0**（`06a46693…`） | agent、工具、会话、MCP、SDK |
| Web UI / 服务 | `pi-web`（Next.js） | **v0.12.1**（`f83b1a50…`） | SPA + `/api/*` + SSE + 进程内 agent |
| Web 框架 | Next.js | **16.3.8** | `pi-web` 生产构建 |
| 语言 | TypeScript | **^5.6**（桌面） | 壳层与脚本 |
| 运行时 | Node.js | 22 / 24（分版） | 宿主与载荷 |
| 包管理 | npm（桌面）/ pnpm（构建）/ ohpm（鸿蒙）/ Gradle（Android） | 见各节 | 依赖与构建 |

### 1.2 四个产品形态

| 维度 | `pi-desktop`（Desktop） | `pi-desktop-hos`（Electron on HarmonyOS） | `pi-desktop-ark`（HarmonyOS） | `pi-desktop-android`（Android） |
|---|---|---|---|---|
| 壳技术 | Electron **43.7.9** | Electron-on-HarmonyOS **37.2.3** + ArkTS | ArkTS + **ArkWeb** | Kotlin + 系统 **WebView** |
| Node | **24.21.0**（Electron 内置） | **22.17.0**（运行时内置） | **24.2.0**（ohpm 包） | **24.22.1**（Maven AAR，small-ICU） |
| 渲染内核 | Chromium **150.0.7871.250** | Chromium **138**（包内） | 系统 ArkWeb | 系统 WebView |
| ABI | x64 / arm64（Electron） | arm64-v8a | arm64-v8a | arm64-v8a only |
| 进程内宿主 | Electron 主进程 `startServer` | Electron 主进程 `require` Next | App 进程 `node::Start` | App 进程 `dlopen` + `node::Start` |
| 鉴权 | 随机端口 + 随机密码 + Basic/Cookie | 随机密码 + HMAC 会话 Cookie | loopback 免密 | loopback 免密 |
| 应用版本 | **0.12.1** | **0.12.1** / `1201` | **0.1.0** / `100` | **0.1.0** / `1` |
| 发布 tag | `v0.12.1` | `v0.12.1` | `v0.12.1` | `v0.12.1` |
| 产物 | Squirrel / zip / dmg / deb / rpm | 签名 HAP | 签名 HAP | 签名 APK |
| 上游补丁 | **0** | **2** | **6** | **6** |

### 1.3 运行时包

| 包 | 版本 | 分发 | 消费方 | 内容 |
|---|---|---|---|---|
| `@fellow99/node-bridge` | **0.1.0** | ohpm 中心仓 | ark | ArkTS API + NAPI 引导件 `libnode_bridge.so` |
| `@fellow99/node-runtime` | **24.2.0** | ohpm 中心仓 | ark | `libnode.so` + `libc++_shared.so` |
| `io.github.fellow99:node-bridge` | **0.2.0** | Maven Central | android | Kotlin API + JNI 桥 |
| `io.github.fellow99:node-runtime` | **24.22.1** | Maven Central | android | `libnode.so`（small-ICU）+ `libc++_shared.so` |

> 运行时源码仓库：`ohos-node-bridge`（鸿蒙）/ `aosp-node-runtime`（Android）；本 workspace 不直接包含其源码。

---

## 2. 分版技术细节

### 2.1 `pi-desktop`

| 类别 | 技术 | 版本 | 用途 |
|---|---|---|---|
| 运行时 | Electron | **43.7.9**（精确钉住） | 桌面容器 |
| Node | Electron 内置 | **24.21.0** | 进程内宿主 |
| Chromium | Electron 内置 | **150.0.7871.250** | 渲染 |
| 构建 | Electron Forge | **^7.11.2** | 打包 / makers / Fuses |
| Vite | `@electron-forge/plugin-vite` + Vite | **^7.3.6** | 三段构建（main/preload/renderer） |
| 类型 | TypeScript | **^5.6** | strict |
| 打包 | Squirrel / DMG / ZIP / DEB / RPM | Forge makers | 三平台产物 |
| CI | GitHub Actions | `windows-latest` / `ubuntu-latest` / `macos-latest` | 构建 / 发布 |
| 原生 | `node-pty` | 1.2.0-beta.15 | 终端 PTY（构建期完整性门禁） |

### 2.2 `pi-desktop-hos`

| 类别 | 技术 | 版本 | 用途 |
|---|---|---|---|
| 运行时 | Electron-on-HarmonyOS | **37.2.3** | 鸿蒙上的 Electron |
| Node | 运行时内置 | **22.17.0** | 进程内宿主 |
| ABI | — | **v138** | 原生模块 |
| 壳 | ArkTS + ArkUI | HarmonyOS API 17 兼容 / 24 目标 | entry + web_engine |
| 构建 | hvigor + DevEco | 见 `build-profile.json5` | HAP |
| 脚本 | Node `.mjs` | Node 22.17.0 | collect-runtime / prepare-next / smoke / collect-pi / collect-app |
| 图片 | Python + Pillow / ffmpeg | 构建机 | 品牌图标生成 |

### 2.3 `pi-desktop-ark`

| 类别 | 技术 | 版本 | 用途 |
|---|---|---|---|
| 壳 | ArkTS + ArkWeb | API 17 兼容 / 24 目标 | UIAbility + 页面 |
| 运行时 | `@fellow99/node-bridge` | **0.1.0** | 进程内 Node 桥 |
| 引擎 | `@fellow99/node-runtime` | **24.2.0** | libnode.so |
| 形态 | `NODE_MODE='compat'` | 编译期固定 | `--jitless` |
| 构建 | hvigor + DevEco | 见 `build-profile.json5` | HAP |
| 图像 | ffmpeg 7.1 | 构建机 | 图标生成（无 SVG 光栅化器） |

### 2.4 `pi-desktop-android`

| 类别 | 技术 | 版本 | 用途 |
|---|---|---|---|
| 语言 | Kotlin | **2.2.20** | 壳 |
| 构建 | Android Gradle Plugin | **8.10.1** | APK |
| Gradle | Gradle | **8.12** | 构建 |
| JDK | Java | **17** | 编译 |
| SDK | compileSdk / targetSdk / minSdk | **36 / 36 / 24** | 兼容范围 |
| 运行时桥 | `node-bridge` | **0.2.0** | JNI |
| 引擎 | `node-runtime` | **24.22.1** | `libnode.so`（small-ICU） |
| 协程 | `kotlinx-coroutines-android` | **1.9.0** | 异步 |
| CI | GitHub Actions | `ubuntu-latest` | 构建 / 发布 |

---

## 3. 构建与发布工具链

| 工程 | 构建命令（主路径） | 门禁 | 发布 |
|---|---|---|---|
| desktop | `npm ci → prepare:next → collect:pi → smoke → package/make` | `typecheck` + `smoke-desktop` 四项断言 | `.github/workflows/release.yml`（tag `v*`） |
| hos | `collect-runtime → prepare-next → smoke-node22 → collect-pi → collect-app → hvigor` | `smoke-node22` 0 FAIL + 守卫 | 本地签名 HAP + 真机 |
| ark | `assemble-payload → hvigor` | 载荷 tar / 补丁可逆 / HAP 编译 | 本地签名 HAP + 真机 |
| android | `assemble-payload → verify-runtime → gradle assembleRelease/Debug` | 引擎 SHA-256 + ELF 六项 + `compliance-check` | `.github/workflows/build-release.yml`（tag `v*`） |

通用工具：

- **Node.js 脚本**（`.mjs`）：上游检出、载荷组装、品牌化、门禁、探针。
- **Python + Pillow**：图标缩放/裁圆/背景色采样（desktop / hos / android）。
- **ffmpeg**：图标缩放/合成（ark / hos）。
- **adb / hdc / uitest**：Android / HarmonyOS 真机取证。
- **GitHub Actions**：桌面与 Android 的 CI/Release；鸿蒙两版当前本地构建。

---

## 4. 选型理由

1. **Electron（桌面 / HOS）**：成熟桌面 API、Chromium 一致渲染、PTY/托盘/通知开箱即用；代价是包体大。
2. **ArkWeb + 进程内 Node（ark）**：去掉 Electron/Chromium，包体更小、用系统 WebView；代价是 JIT/ACL/原生能力受限。
3. **系统 WebView + 进程内 Node（Android）**：与 ark 同构，复用 pi-web；代价是 WebView 版本碎片与 SAF 路径模型。
4. **上游零 fork**：四版只装配 `pi` / `pi-web`；补丁按 tag 归档、幂等可逆，降低升级成本。
5. **进程内宿主**：无子进程、无孤儿、就绪判定确定；代价是宿主崩溃 = 应用崩溃（fail-visible 兜底）。
6. **同源 HTTP/SSE**：无 CORS、无自定义协议、无 IPC 数据面；可用 `curl`/`adb forward`/`hdc` 取证。
7. **版本锁定**：上游 tag + commit SHA、运行时 AAR/HAR 版本 + SHA-256；升级必须走版本模块并重跑门禁。

---

## 5. 兼容性与风险

| # | 风险 | 影响 | 现状 / 缓解 |
|---|---|---|---|
| R1 | Electron 43 的 `start-server.js` 属上游内部入口 | 升级可能破坏进程内宿主 | ✅ 已核实；`smoke-desktop` 门禁；候选回退 sidecar 仅登记 |
| R2 | `node-pty` ABI / 预构建完整性 | 终端不可用 | ✅ Windows 已修复 + 构建期完整性门禁；mac/Linux 待验证 |
| R3 | Electron-on-鸿蒙运行时版本落后（Electron 37 / Node 22.17） | 部分 API 缺失 | 固定 v37.2.3；能力清单标注降级 |
| R4 | 鸿蒙 JIT / ACL 限制 | code-mode/WASM 不可用 | ark 固定 `compat`（`--jitless`）；android 默认 JIT |
| R5 | 鸿蒙系统 WebView/ArkWeb 差异 | 外链/选择器/通知行为不同 | 桥 + 补丁 + 真机矩阵；缺口如实登记 |
| R6 | Android WebView 版本碎片 | 旧内核缺 `Intl` 等 | `polyfill-webview.js`；minSdk 24 |
| R7 | 移动端无系统 git / fd / rg | `git` / `find` / `grep` 缺失 | 纯 JS 同名顶替扩展（isomorphic-git / 自研搜索） |
| R8 | 运行时 Node 版本与上游 `engines` 不一致 | 启动失败 | 四版均 ≥22.19；android 24.22.1 / ark 24.2.0 / hos 22.17.0 |
| R9 | 上游 tag 重打 / 分支漂移 | 构建不可复现 | CI 断言 commit SHA；`runtime.lock.json` 锁 SHA-256 |
| R10 | 签名材料 / API Key 泄露 | 安全与合规 | 不入库；日志脱敏；启动前扫描 |
| R11 | 品牌物料缺失 | 无品牌包 | 构建硬失败（桌面/HOS/Android/ark） |
| R12 | 鸿蒙应用名 / 页面 Logo 未统一 | 品牌不一致 | ark 013 登记为 ⬜；待产品定名与 SVG 方案 |
| R13 | 文档与源码漂移 | 误导维护 | 源码即真理；`docs/文档实现不一致项.md` 跟踪回写 |
| R14 | 四版能力口径不一 | 验收争议 | `docs/能力对照清单.md` 统一状态图标 + 证据 |

---

## 6. 运行环境基线

| 项 | 要求 |
|---|---|
| 构建机 Node（desktop） | **24.21.0**（与 Electron 内置一致） |
| 构建机 Node（hos） | **22.17.0**（`prepare-next.mjs --node`） |
| 构建机 Node（ark/android 脚本） | Node 24（脚本）/ 与本机兼容 |
| JDK（android） | **17** |
| Python | 3.x + Pillow（品牌脚本） |
| ffmpeg | 7.1（ark / hos 图标） |
| 鸿蒙 SDK | API 17 兼容 / 24 目标；hdc / DevEco |
| Android SDK | compileSdk 36；adb；签名工具 |
| 磁盘 | 四版构建产物与运行时归档较大（libnode.so 约 121 MB / 81.5 MB） |

---

## 7. 版本锁定文件索引

| 文件 | 作用 |
|---|---|
| `pi-desktop/package.json` / `package-lock.json` | Electron / Forge / Node 版本 |
| `pi-desktop/upstream.lock.json`（生成物） | 上游 commit + baseline |
| `pi-desktop-hos/AppScope/app.json5` | 应用版本 / bundleName |
| `pi-desktop-hos/oh-package.json5` / `electron/oh-package.json5` | 鸿蒙依赖 |
| `pi-desktop-ark/AppScope/app.json5` | 应用版本 / bundleName |
| `pi-desktop-ark/oh-package.json5` | node-bridge / node-runtime |
| `pi-desktop-ark/patches/v0.12.1/` | 上游补丁集 |
| `pi-desktop-android/app/build.gradle.kts` | 应用版本 / 签名 / ABI |
| `pi-desktop-android/gradle/libs.versions.toml` | AGP / Kotlin / AAR 版本 |
| `pi-desktop-android/runtime.lock.json` | Node / bridge / SHA-256 / ELF 门禁 |
| `pi-desktop-android/patches/v0.12.1/` | 上游补丁集 |
