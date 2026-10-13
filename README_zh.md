[English](./README.md) | 中文

---

# Pi Desktop Workspace

> 本仓库是一个聚合工程，把将 [Pi](https://github.com/earendil-works/pi)（可扩展的 agent harness）与其浏览器界面
> [Pi Web](https://github.com/agegr/pi-web) 封装为桌面应用所需的全部工程汇聚到一处：Windows / macOS / Linux 端由
> Electron 桌面壳承载，鸿蒙 PC / 平板端由 "Electron-on-鸿蒙" 运行时承载。
> 上游仓库、运行时与品牌物料以 **git submodule + 同级普通目录**并列，使本 workspace 的目录结构恰好还原各封装工程
> 构建脚本所期望的兄弟目录布局（`../pi`、`../pi-web`、`../assets`、`../harmonypc-electron`）。

---

## 这是什么

`Pi` 是上游开源的可扩展 agent harness（MIT，[earendil-works/pi](https://github.com/earendil-works/pi)），提供
CLI / RPC / JSON / SDK 四个集成入口；`Pi Web` 是它的社区浏览器界面（MIT，[agegr/pi-web](https://github.com/agegr/pi-web)），
**一个 Node 进程同时提供 SPA 与 `/api/*`**，agent 就在该进程内创建。两个封装工程把这一整套搬进桌面：

- **`pi-desktop`** —— Electron 桌面壳（Windows / macOS / Linux），Electron **43.7.9**：主进程**进程内**承载
  `pi-web` 的 Next 服务器（含 agent 与 `/api/*`），渲染层同源加载官方 Pi Web UI。
- **`pi-desktop-hos`** —— 鸿蒙 PC / 平板移植版，基于 `harmonypc-electron` 的 Electron-on-鸿蒙运行时
  （Electron **37.2.3** / Node **22.17.0**），打包为可签名分发的 **HAP**。

两个壳都只做**装配**：UI 100% 复用上游，必需补丁 **0 条**，不 fork、不改上游源码
（依据：`pi-desktop/docs/工程规划.md` §1.1–§1.2、`pi-desktop-hos/docs/工程规划.md` §1.1–§1.2）。

本仓库自身不含产品代码 —— 它是承载这些工程的**容器**，让所有兄弟目录引用在 clone 下来时天然成立。

## 仓库布局

```
pi-workspace/                       # 本仓库 —— submodule 容器（父工程）
├── pi/                             # [submodule] 上游 agent harness（宿主 + SDK，MIT）
├── pi-web/                         # [submodule] 上游浏览器界面 + Next 服务端（MIT）
├── pi-desktop/                     # [submodule] Electron 桌面壳（Windows / macOS / Linux）
├── pi-desktop-hos/                 # [submodule] 鸿蒙 PC / 平板移植壳（HAP）
├── pi-desktop-website/             # [submodule] 两个壳的产品官网（纯静态双页站点）
├── harmonypc-electron/             # [submodule] Electron-on-鸿蒙运行时（Electron 37.2.3 / Node 22.17.0）
├── assets/                         # （普通目录）品牌物料唯一来源：logo.* + screenshots/
├── harmonypc-electron-versions/    # （普通目录，未入库）运行时发行版归档 + 原生模块编译指导
└── LICENSE · README.md · README_zh.md
```

### 各角色一览

| 目录 | 角色 | 被谁消费 |
|---|---|---|
| `pi` | 上游 agent harness（宿主 + SDK） | 两个壳（源码引用与上游版本锁定基准） |
| `pi-web` | 上游浏览器界面 + Next 服务端 | 两个壳（构建期取其生产产物） |
| `pi-desktop` | Electron 桌面壳（Windows / macOS / Linux） | — |
| `pi-desktop-hos` | 鸿蒙 PC / 平板移植壳（HAP） | — |
| `pi-desktop-website` | 两个壳的产品官网 | — |
| `harmonypc-electron` | Electron-on-鸿蒙运行时（原生 SO + ArkTS 外壳） | 仅鸿蒙壳 |
| `assets` | **品牌物料的唯一来源**：两个壳的应用图标 / 窗口图标 / 状态栏字形 / 页面 Logo，以及官网的 Logo 与截图 | 两个壳 + 官网 |
| `harmonypc-electron-versions` | 运行时发行版归档（v34 / v37）+ 原生模块重编译材料 | 仅鸿蒙壳工具链 |

### 两个普通目录（非 submodule）

- **`assets/`** —— 品牌物料：`logo.svg`、`logo.png`、`logo-1024-with-bg.png`、`logo-200-with-bg.png`、`logo.ico`、
  `logo.icns`，外加 `screenshots/`（`pc/` 与 `hos/` 两个子目录；`pc/` 现有 `1.png`–`6.png`，`hos/` 目前为空 ⬜）。
  它不随任一壳分发，而是被两个壳的品牌资源生成脚本与官网当作**唯一的物料来源**读取，因此必须与两个壳同级。
- **`harmonypc-electron-versions/`** —— 鸿蒙运行时的各版本归档与原生模块编译指导：
  `v34.8.5-*.zip`、`v37.2.3-*.zip`、`electron144_ohos_hap/`、`libelectron/`、`better-sqlite3编译指导（Electron37）/`，
  以及两份《Electron34 / Electron37 的 node 头文件（Headers）使用指导》压缩包。用途是：当需要固定 submodule 之外的
  运行时版本，或在鸿蒙侧重编译原生模块（如 `better-sqlite3`）时取用。
  该目录**不在版本库里**（`.gitignore` 中有 `/harmonypc-electron-versions` 条目）；`assets/` 目前同样未被 git 跟踪。

## 各部分如何协作

两个壳的构建脚本一律以「本工程根目录的上一级」推导上游与物料路径，脚本内**没有**安装目录回退：

| 需要的同级目录 | 文件级依据 |
|---|---|
| `../pi-web` | `pi-desktop/scripts/prepare-next.mjs`、`collect-pi.mjs`、`smoke-desktop.mjs`（`REPO_PARENT = resolve(ROOT, '..')` 后 `join(REPO_PARENT, 'pi-web')`）；`pi-desktop-hos/scripts/prepare-next.mjs`、`collect-pi.mjs`、`smoke-node22.mjs` |
| `../pi` | `pi-desktop/scripts/pin-upstream.mjs`（把 `../pi`、`../pi-web` 的 commit / describe / 分支写入 `upstream.lock.json`）；`pi-desktop-hos/scripts/prepare-next.mjs`（存在性硬校验：`pi-web` 的 `@earendil-works/*` 由同级 `../pi` 解析）；`pi-desktop/.github/workflows/ci.yml` |
| `../assets` | `pi-desktop/scripts/brand-assets.mjs` 与 `pi-desktop-hos/scripts/brand-assets.mjs`（物料目录默认为父工程 `../assets`，缺失时才回退仓内镜像 `resources/brand/`） |
| `../harmonypc-electron` | `pi-desktop-hos/scripts/collect-runtime.mjs`（从同级 `../harmonypc-electron/ohos_hap` 拷贝运行时外壳） |
| `../harmonypc-electron-versions` | `pi-desktop-hos/scripts/collect-runtime.mjs`（该阶段从同名目录解压 v37 归档取原生 SO 与 Chromium 资源） |

因此这些工程**必须**处在同一父目录下、且目录名不能改：单独 clone 某一个壳、或把上游放到别处，构建会直接失败
（`pi-desktop-hos/scripts/prepare-next.mjs` 找不到 `../pi-web` / `../pi` 即 `fail`；`pi-desktop/scripts/brand-assets.mjs`
在父工程物料与仓内镜像都不存在时同样硬失败）。CI 也照此假设工作：`pi-desktop/.github/workflows/ci.yml` 在文件头写明
「CI 只 checkout 本仓库，而构建脚本按 `../pi`、`../pi-web` 解析上游源码」，于是用 `git clone` 把两者拉到与 `pi-desktop` 平级。

**数据面是共享的**：两个壳都采用「主进程内托管 `pi-web` 的 Next 服务器 + 渲染层同源加载」的形态 —— agent 与
`/api/*` 就在 Electron 主进程内，UI 复用未改动的官方前端，因此没有 CORS、没有自定义协议、没有登录页、上游零改动。
差异集中在运行时与平台接入（运行时版本、打包形态、网络绑定、平台能力缺口），以两个壳各自的 README 与
`docs/工程规划.md` 为准。

## Submodule 清单

全部条目记录在 [`.gitmodules`](.gitmodules)：

| 路径 | 远端 |
|---|---|
| `pi` | https://github.com/earendil-works/pi.git |
| `pi-web` | https://github.com/agegr/pi-web.git |
| `pi-desktop` | https://github.com/fellow99/pi-desktop.git |
| `pi-desktop-hos` | https://github.com/fellow99/pi-desktop-hos.git |
| `pi-desktop-website` | https://github.com/fellow99/pi-desktop-website.git |
| `harmonypc-electron` | https://github.com/fellow99/harmonypc-electron.git |

```bash
git clone --recursive https://github.com/fellow99/pi-workspace.git
cd pi-workspace
git submodule update --init --recursive   # 拉取/刷新全部固定版本的 submodule
```

> `assets/` 与 `harmonypc-electron-versions/` 不在版本库里，clone 后不会出现；需要构建品牌资源或鸿蒙运行时归档时，
> 按「两个普通目录」一节的说明自行准备（前者缺失时，桌面壳会回退到仓内镜像 `resources/brand/`；鸿蒙壳的
> `collect-runtime` 则需要 `../harmonypc-electron-versions` 中的 v37 归档）。

如需推进固定版本，可执行 `git submodule update --remote`，随后提交新的指针。

## 现状

| 项 | 现状 |
|---|---|
| 上游基线 | `pi` **v1.1.0**（`Branch_v1.1.0`）+ `pi-web` **v0.11.1**（`Branch_v0.11.1`），commit 锁定记录在 `pi-desktop/upstream.lock.json` |
| `pi-desktop` | 版本 **0.11.1**；Windows 打包态端到端已实测（进程内宿主 / 首屏 / 产物引导 / 品牌化 / 终端 PTY / 原生通知 / 安装-启动-卸载 ✅，见 `docs/构建验证记录.md` 轮次 1–2）；**macOS / Linux 待验证**；系统托盘的视觉呈现 ⬜ 未专项取证（`docs/桌面能力清单.md`） |
| `pi-desktop-hos` | 版本 **0.11.1** / `versionCode` **1101**；**已构建、已签名、已真机验证**（HarmonyOS 6.1.0.135 / API 24，见 `docs/真机验证记录.md`）；能力边界见 `docs/鸿蒙环境能力清单-v0.11.1.md` |
| 上游改动 | 两个壳均为**必需补丁 0 条**，上游源码零改动（品牌化只作用于各自的产物副本） |
| `pi-desktop-website` | 纯静态双页站点（`index.html` / `index_zh.html`，无框架、零外部请求），经 GitHub Actions 部署到 GitHub Pages |
| 本仓库 | 双语 README（`README_zh.md` 原文 / `README.md` 译文）+ MIT `LICENSE`；`assets/` 与 `harmonypc-electron-versions/` 为本地普通目录，未入库 |
| 待办 ⬜ | `assets/screenshots/hos/` 尚无截图；`macOS` / `Linux` 的打包态验证；托盘与高 DPI / 深浅色呈现的专项取证 |

能力与验证状态的具体口径以**各产品仓内的文档**为准（本文件只做总览，可能与它们存在时差）。

## 许可

本仓库：[MIT](./LICENSE) © 2026 fellow99。

各子工程各自持有许可，本仓库不改变它们：`pi` 为 MIT（`earendil-works/pi`，© 2025 Mario Zechner）、`pi-web` 为 MIT
（`agegr/pi-web`，© 2026 agegr），两个壳与官网、运行时也各自带有仓内 `LICENSE`（MIT）。
使用某个子目录的内容时，以该目录内的 `LICENSE` 为准。

## 非官方声明

本 workspace 汇总的两个桌面封装工程均为**第三方作品**：它们不是 Pi / Pi Web 团队的产物，与上游作者
**无隶属关系，也未获其背书**。`Pi`（MIT，earendil-works/pi）与 `Pi Web`（MIT，agegr/pi-web）是各自作者的上游开源项目；
两个壳只**装配**它们 —— 不 fork、不改上游源码。`harmonypc-electron` 是其自身作者的项目，本仓库只做同级引用。
