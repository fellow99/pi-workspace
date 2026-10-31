# STRUCTURE.md — Pi Desktop Workspace 目录与文件结构

> 本文件记录父工程整体目录结构，供规范文档与后续维护参考。
> 扫描范围：仓库根目录与四个子工程 2–3 层；排除 `.git`、`node_modules`、`oh_modules`、`build`、`.gradle`、
> `.hvigor`、`logs`、`out`、`dist`、`pi-dist` 等构建/依赖目录。**生成时间：** 2026-10-31。

---

## 1. 顶层结构

```
pi-workspace/                          # 父工程（submodule 容器）
├── .gitmodules                        # 8 个 submodule 的路径与远端
├── README.md / README_zh.md           # 双语总览：四版定位、布局、协作、能力矩阵、许可
├── LICENSE                            # MIT
├── assets/                            # 品牌物料唯一来源（已入库）
│   ├── logo.svg / logo.png / logo-1024-with-bg.png / logo-200-with-bg.png
│   ├── logo.ico / logo.icns
│   └── screenshots/{pc,hos}/
├── docs/
│   ├── 能力对照清单.md                 # 四版 × 能力矩阵 + 源码核实证据
│   └── 文档实现不一致项.md             # 文档 ↔ 源码差异与回写记录
├── specs/                             # 父工程规范文档（本目录）
│   ├── README.md / STRUCTURE.md / TECH.md / ARCHITECTURE.md
│   ├── constitution.md / overall-spec.md / overall-plan.md
│   ├── 011-version-0.12.1/{spec.md,plan.md}
│   ├── 012-building-workflow/{spec.md,plan.md}
│   └── 013-branding/{spec.md,plan.md}
├── pi/                                # [submodule] 上游 agent harness（MIT）
├── pi-web/                            # [submodule] 上游浏览器界面 + Next 服务（MIT）
├── pi-desktop/                        # [submodule] Electron 桌面壳
├── pi-desktop-hos/                    # [submodule] Electron on HarmonyOS 版
├── pi-desktop-ark/                    # [submodule] HarmonyOS 纯血版
├── pi-desktop-android/                # [submodule] Android 版
├── pi-desktop-website/                # [submodule] 四版产品官网
├── harmonypc-electron/                # [submodule] Electron-on-鸿蒙运行时
└── harmonypc-electron-versions/       # 本地运行时归档（未入库，gitignored）
```

### 1.1 submodule 清单（`.gitmodules`）

| 路径 | 远端 | 被谁消费 |
|---|---|---|
| `pi` | `earendil-works/pi` | 四版（agent 源码 / 依赖） |
| `pi-web` | `agegr/pi-web` | 四版（UI + Next 服务） |
| `pi-desktop` | `fellow99/pi-desktop` | — |
| `pi-desktop-hos` | `fellow99/pi-desktop-hos` | — |
| `pi-desktop-ark` | `fellow99/pi-desktop-ark` | — |
| `pi-desktop-android` | `fellow99/pi-desktop-android` | — |
| `pi-desktop-website` | `fellow99/pi-desktop-website` | — |
| `harmonypc-electron` | `fellow99/harmonypc-electron` | `pi-desktop-hos` |

### 1.2 两个普通目录（非 submodule）

- `assets/`：品牌物料（已入库）；被 `pi-desktop` / `pi-desktop-hos` / `pi-desktop-ark` / `pi-desktop-android` 的品牌脚本读取。
- `harmonypc-electron-versions/`：Electron-on-鸿蒙运行时归档与原生模块编译材料；仅 `pi-desktop-hos` 的
  `collect-runtime.mjs` 使用；未入库。

---

## 2. 上游工程

### 2.1 `pi/` — agent harness

```
pi/
├── packages/
│   ├── agent/            # agent 核心
│   ├── ai/               # 模型适配
│   ├── coding-agent/     # 编码 agent（工具/会话/RPC）
│   ├── codemode/         # JS 沙箱 / code mode
│   ├── durable/          # 持久化
│   ├── env/              # 环境
│   ├── mcp/              # MCP
│   ├── telemetry/        # 遥测
│   └── tui/              # 终端 UI
├── scripts/
├── package.json
└── AGENTS.md / README.md / SECURITY.md
```

- 版本：`v1.2.0`；四版共同锁定同一 commit。
- 入口：CLI / RPC / JSON / SDK（由 `pi-web` 进程内调用）。

### 2.2 `pi-web/` — UI + Next 服务

```
pi-web/
├── app/
│   ├── api/              # /api/* 路由（agent / sessions / models / mcp / files / git / terminal / ...）
│   ├── login/            # 登录页（桌面版 Basic/Cookie 注入后通常不可见）
│   └── layout.tsx / page.tsx / manifest.ts
├── components/           # UI 组件（ChatWindow、AppShell、PiWebTitle、...）
├── lib/                  # 服务端逻辑（rpc-manager、sessions、auth、...）
├── public/               # 静态资源、icons、sw.js、offline.html
├── docs/                 # 上游能力文档（agents/*.md）
├── instrumentation-node.ts
├── next.config.ts
└── package.json          # v0.12.1
```

- 一个 Node 进程同时提供 SPA 与 `/api/*`；agent 在该进程内创建。
- 四版均不 fork；桌面版直接进程内 `startServer`，移动端由载荷启动脚本托管。

---

## 3. 子工程结构

### 3.1 `pi-desktop/` — Electron 桌面壳

```
pi-desktop/
├── src/
│   ├── main/             # host / windows / tray / notifications / lifecycle / artifact / settings / update-check
│   ├── preload/          # 最小壳层桥（file:// 守卫）
│   ├── renderer/         # loading / 错误页
│   └── types/
├── scripts/              # prepare-next / collect-pi / smoke-desktop / pin-upstream / brand-assets / probe-renderer
├── config/               # settings.json / mcp.json / trust 模板
├── resources/            # 图标、托盘、安装器资源、brand 镜像
├── specs/                # 子工程规范（011/012/013 + 101–113 + 201）
├── docs/                 # 工程规划 / 发布与更新 / 构建验证记录 / 桌面能力清单
├── .github/workflows/    # ci.yml / release.yml
├── forge.config.ts
├── vite.{main,preload,renderer}.config.ts
└── package.json
```

- 版本：`0.12.1`；Electron 43.7.9 / Node 24.21.0。
- 入口：`src/main/index.ts`；主进程进程内启动 Next。
- 构建：`npm ci → prepare:next → collect:pi → smoke → package/make`。

### 3.2 `pi-desktop-hos/` — Electron on HarmonyOS 版

```
pi-desktop-hos/
├── electron/             # entry 模块（EntryAbility、runtime 投放、resfile）
├── web_engine/           # HAR：ArkUI 承载窗口 + 适配器 + ArkWeb 外链页
├── src-main/             # 主进程 main.js / tar-extract.js / package.json
├── extensions/           # harmony-exec / harmony-search / harmony-runtime / harmony-workspace-roots / harmony-git
├── config/               # settings.json / mcp.json
├── runtime-overlays/     # 对运行时/HAR 的整文件替换
├── skills/               # harmony-runtime-capabilities
├── patches/v0.12.1/      # 两条 pi-web 补丁
├── scripts/              # collect-runtime / prepare-next / smoke-node22 / collect-pi / collect-app / brand-assets / build-hap
├── specs/                # 子工程规范（011/013 + 101/106/108 + 203/204）
├── docs/                 # 工程规划 / 真机验证记录 / 能力清单 / ACL
├── AppScope/app.json5    # bundleName / versionCode 1201 / versionName 0.12.1
└── build-profile.json5
```

- 运行时：Electron 37.2.3 / Node 22.17.0 / ABI 138；HarmonyOS API 17 兼容 / 24 目标。
- 构建：`collect-runtime → prepare-next → smoke-node22 → collect-pi → collect-app → hvigor`。

### 3.3 `pi-desktop-ark/` — HarmonyOS 纯血版

```
pi-desktop-ark/
├── entry/src/main/
│   ├── ets/
│   │   ├── app/PiDesktopRuntime.ets
│   │   ├── common/{BuildConfig,Constants,DeviceTest,NodeMode,Ports,WindowState}.ets
│   │   ├── entryability/EntryAbility.ets
│   │   ├── host/HostCapabilities.ets
│   │   └── pages/Index.ets
│   └── resources/{rawfile,resfile}/
├── AppScope/app.json5    # org.fellow99.pi.PiDesktopArk / 0.1.0 / 100
├── payload/
│   ├── config/           # settings.json / mcp.json
│   ├── devtest/          # 设备自检套件
│   ├── extensions/       # harmony-exec / search / runtime / workspace-roots / git
│   ├── jitless/          # compat 载荷（WASM 裁剪/纯 JS 垫片）
│   └── skills/           # harmony-runtime-capabilities
├── patches/v0.12.1/      # 六条 pi-web 补丁
├── scripts/              # assemble-payload / brand-assets / sync-git-extension / verify-runtime
├── specs/                # 子工程规范（011/013 + 101–108 + 204）
├── docs/                 # 工程规划 / 能力复刻评估 / JITless 决策 / ACL
├── oh-package.json5      # node-bridge 0.1.0 + node-runtime 24.2.0
└── build-profile.json5
```

- 运行时：`@fellow99/node-bridge` 0.1.0 + `@fellow99/node-runtime` 24.2.0；Node v24.2.0；`compat`（`--jitless`）。
- 构建：`assemble-payload → hvigor`；HAP 签名后真机安装。

### 3.4 `pi-desktop-android/` — Android 版

```
pi-desktop-android/
├── app/src/main/
│   ├── java/org/fellow99/pi/PiDesktopAndroid/
│   │   ├── MainActivity.kt
│   │   ├── PiApplication.kt
│   │   ├── host/HostCapabilities.kt
│   │   ├── payload/PayloadArchive.kt
│   │   ├── runtime/{PiDesktopRuntime,EngineFacts}.kt
│   │   └── service/PiForegroundService.kt
│   ├── res/              # strings / mipmap 五密度 / adaptive / network_security_config
│   └── assets/           # polyfill-webview.js 等
├── payload/
│   ├── config/
│   ├── extensions/       # android-exec / search / runtime / workspace-roots / saf / git / pnpm / pty
│   ├── jitless/
│   └── skills/
├── patches/v0.12.1/      # 六条 pi-web 补丁
├── scripts/              # assemble-payload / brand-assets / generate-icons / verify-runtime / devtest / compliance-check
├── specs/                # 子工程规范（011/012/013 + 101–107）
├── docs/                 # 工程规划 / 真机验证记录 / 能力清单 / 隐私与 AI 标识
├── .github/workflows/build-release.yml
├── gradle/libs.versions.toml
├── runtime.lock.json     # Node 24.22.1 / bridge 0.2.0 / SHA-256 / ELF 门禁
└── build.gradle.kts
```

- 运行时：`io.github.fellow99:node-bridge` 0.2.0 + `node-runtime` 24.22.1（small-ICU）；Node 24.22.1。
- 版本：`0.1.0` / `versionCode` 1；发布 tag `v0.12.1`。
- 构建：`assemble-payload → verify-runtime → gradle assembleRelease/Debug → GitHub Release`。

### 3.5 其它

- `pi-desktop-website/`：`index.html` / `index_zh.html`，纯静态、零外部请求，GitHub Pages 部署。
- `harmony*`：运行时与归档，不属于产品壳。

---

## 4. 关键配置与入口

| 文件 | 工程 | 作用 |
|---|---|---|
| `pi-desktop/package.json` | desktop | 版本 `0.12.1`、脚本、Electron 依赖 |
| `pi-desktop/forge.config.ts` | desktop | Forge 打包、Fuses、makers、`extraResource` |
| `pi-desktop/.github/workflows/{ci,release}.yml` | desktop | 三 OS 构建 / tag 发布、上游 tag + SHA 断言 |
| `pi-desktop-hos/AppScope/app.json5` | hos | bundleName、`0.12.1` / `1201` |
| `pi-desktop-hos/build-profile.json5` | hos | 产品、SDK 17/24、模块 |
| `pi-desktop-hos/src-main/main.js` | hos | 主进程宿主、鉴权、窗口、品牌、能力桥 |
| `pi-desktop-ark/AppScope/app.json5` | ark | bundleName、`0.1.0` / `100` |
| `pi-desktop-ark/oh-package.json5` | ark | node-bridge / node-runtime 版本 |
| `pi-desktop-ark/entry/src/main/ets/app/PiDesktopRuntime.ets` | ark | 启动编排 |
| `pi-desktop-ark/patches/v0.12.1/` | ark | 六条补丁 |
| `pi-desktop-android/app/build.gradle.kts` | android | applicationId、版本、签名、ABI |
| `pi-desktop-android/runtime.lock.json` | android | 引擎单一事实源 |
| `pi-desktop-android/.github/workflows/build-release.yml` | android | tag 触发构建/发布 |
| `pi-web/app/api/**` | 上游 | 数据面路由（约 90 条 + SSE） |
| `pi-web/lib/rpc-manager.ts` | 上游 | 进程内 agent 创建 |

---

## 5. 页面 / 路由概览

数据面全部由上游 `pi-web` 提供，四版同源加载，**不新增路由**。主要面：

| 面 | 路径 | 说明 |
|---|---|---|
| SPA | `/` | 主界面（会话、模型、工具、设置） |
| 登录 | `/login` | 上游登录页（桌面版注入凭据后通常跳过） |
| Agent | `/api/agent/**` | 会话创建、消息、SSE 事件 |
| Sessions | `/api/sessions/**` | 列表、搜索、导出、UI 状态 |
| Models | `/api/models/**`、`/api/models-config/**` | 模型 / Provider 配置 |
| MCP | `/api/mcp/**` | MCP 服务器与 OAuth |
| Files / Git / Terminal | `/api/files/**`、`/api/git/**`、`/api/terminal/**` | 文件、Git、PTY |
| Auth / Trust | `/api/auth/**`、`/api/web-auth`、`/api/project-trust` | 鉴权与项目信任 |
| Update | `/api/app-update` | 上游版本检查（桌面版运行时关闭，移动端亦关闭） |

> 父工程**不写 API.md**：以上仅用于说明结构；完整路由以上游 `pi-web/app/api/**` 为准。

---

## 6. 构建/运行期目录（不入库）

| 目录/文件 | 工程 | 说明 |
|---|---|---|
| `node_modules/` | desktop / 上游 | 依赖 |
| `oh_modules/` | hos / ark | ohpm 依赖 |
| `build/`、`.gradle`、`.kotlin` | android | Gradle 构建 |
| `out/`、`.vite/`、`dist/` | desktop | Forge/Vite 产物 |
| `pi-dist/`、`pi-dist.tar.gz`、`pi-dist.version` | 各版 | 组装载荷 |
| `logs/` | 各版 | 验证日志 |
| `payload/pi-web/`（或 resfile） | ark / android | 组装后的载荷（gitignored） |
| `.hvigor/` | hos / ark | hvigor 缓存 |
| `harmonypc-electron-versions/` | 父工程 | 本地运行时归档 |

---

## 7. 维护规则

1. 本文件随目录结构调整更新；新增顶层目录或子工程必须同步 §1 与 §3。
2. 结构说明只记录**事实路径**；模块职责与需求写在 `spec.md` / `plan.md`。
3. 不入库目录不展开；构建产物只列类型，不列具体文件。
4. 路径以仓库根为基准；跨工程引用用相对路径（如 `../pi-web/app/api`）。
