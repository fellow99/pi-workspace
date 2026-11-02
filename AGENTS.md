# AGENTS.md — pi-workspace（父工程 / submodule 容器）

> 本文件是 **`pi-workspace`（父工程）** 的 AI Agent 工作规范与导航入口。
> 本仓库**不含产品代码** —— 它是把四版 Pi Desktop 所需工程汇聚到同一父目录的 **submodule 容器**，
> 使各封装构建脚本期望的兄弟目录布局（`../pi`、`../pi-web`、`../assets`、`../harmonypc-electron`）在 clone 后天然成立。
> 语言默认中文；代码 / 变量 / 技术名词用英文。

---

## 一、仓库布局与子工程角色

| 目录 | 类型 | 角色 | 是否需要 AGENTS.md |
|---|---|---|---|
| `pi-desktop` | [submodule] | Pi Desktop · Electron 桌面壳（Win/macOS/Linux） | ✅（本目录 `AGENTS.md`） |
| `pi-desktop-hos` | [submodule] | Electron on HarmonyOS 版（HAP） | ✅（本目录 `AGENTS.md`） |
| `pi-desktop-ark` | [submodule] | HarmonyOS 版（ArkWeb + 进程内 Node，HAP） | ✅（本目录 `AGENTS.md`） |
| `pi-desktop-android` | [submodule] | Android 版（系统 WebView + 进程内 Node，APK） | ✅（本目录 `AGENTS.md`） |
| `pi-desktop-website` | [submodule] | 四版产品官网（纯静态） | ✅（本目录 `AGENTS.md`） |
| `pi` | [submodule] | **上游** agent harness（只读引用，不 fork） | ❌ 上游，勿改 |
| `pi-web` | [submodule] | **上游**浏览器 UI + Next 服务（只读引用） | ❌ 上游，勿改 |
| `harmonypc-electron` | [submodule] | **外部** Electron-on-鸿蒙运行时（symlink 到外部工作区，多数为二进制分发） | ❌ 上游/外部，勿改 |
| `harmonypc-electron-versions` | 普通目录 | 运行时发布归档 + 原生模块构建指南（**不入库**） | ❌ 非源码 |
| `assets/` | 普通目录 | 品牌物料唯一来源（图标 / Logo / 截图） | — |
| `specs/` | 普通目录 | 父工程跨工程规范文档 | — |
| `docs/` | 普通目录 | 父工程全局文档（能力对照清单等） | — |

> **上游工程不写 AGENTS.md、不改其内容**：`pi`、`pi-web`、`harmonypc-electron`、`harmonypc-electron-versions`
> 均为上游/外部只读参照，各自 README / AGENTS.md 已足以导航。**本工程编辑范围**限本地子工程与自身根目录。

## 二、本仓库的职责

- **只做装配**：四个版别 UI 100% 复用上游 `pi-web`，业务在进程内 Node 中运行，壳只做装配；不 fork 上游源码。
- **数据面共享**：四版同形 —— 进程内 Next 服务器来自 `pi-web`，视图同源加载；无 CORS、无自定义协议、无登录页。
- **兄弟目录强依赖**：构建脚本从「本工程根的父目录」推导上层路径（`../pi-web`、`../pi`、`../assets`、
  `../harmonypc-electron`、`../harmonypc-electron-versions`），**无回退**；脱离本容器单独 clone 任一子工程会构建失败。

## 三、子模块操作

```bash
git clone --recursive https://github.com/fellow99/pi-workspace.git
cd pi-workspace
git submodule update --init --recursive   # 拉取/刷新所有钉住的子模块
git submodule update --remote             # 推进钉住的提交（然后提交新指针）
```

- 只提交本会话改动的文件，**严禁 `git add -A` / `git add .`**（父子工程可能有并发 pi 会话）。
- 子模块内提交在其各自 gitdir（`./.git/modules/<名>/`）进行；本仓库自身的提交在 `./.git/`。

## 四、规范文档体系（specs）

- **父工程 `specs/`**：整体规划 / 整体设计 / 跨工程公共需求 / 版本矩阵 / 构建与品牌总规范
  （`011-version`、`012-building-workflow`、`013-branding`）。
- **子工程 `specs/`**：本工程具体技术实现、独立需求、模块 `spec.md`＋`plan.md`、真机证据。
- **`docs/`**：操作手册、工程规划、能力清单、真机验证记录。
- 编号规则：`0XX` 公共模块 · `1XX` 各工程基础功能 · `2XX` 补丁/插件模块。
- 原则：**源码即真理**；表述不明的必须标 `⬜ 待验证`，不得臆测。

## 五、CodeGraph 代码地图规范（强制）

> 本方法是本工作区的**强制规范**：所有子工程与父工程均已建立 CodeGraph 语义索引，并配置了
> git `post-commit` 钩子在每次提交后自动增量更新。AI 探索代码**必须优先走 CodeGraph**，
> grep + Read 仅用于 CodeGraph 未覆盖的细节确认。

### 5.1 索引布局

| 工程 | `.codegraph/` 位置 | post-commit 钩子 |
|---|---|---|
| 父工程 `pi-workspace` | `./.codegraph/`（索引全部子模块源码） | `./.git/hooks/post-commit` |
| `pi-desktop` | `./pi-desktop/.codegraph/` | `./.git/modules/pi-desktop/hooks/post-commit` |
| `pi-desktop-hos` | `./pi-desktop-hos/.codegraph/` | `./.git/modules/pi-desktop-hos/hooks/post-commit` |
| `pi-desktop-ark` | `./pi-desktop-ark/.codegraph/` | `./.git/modules/pi-desktop-ark/hooks/post-commit` |
| `pi-desktop-android` | `./pi-desktop-android/.codegraph/` | `./.git/modules/pi-desktop-android/hooks/post-commit` |
| `pi-desktop-website` | `./pi-desktop-website/.codegraph/` | `./.git/modules/pi-desktop-website/hooks/post-commit` |

> `.codegraph/` 一律不入库（各工程 `.gitignore` 已忽略）。

### 5.2 自动更新钩子（已配置，勿移除非阻塞约定）

- 每个子工程与父工程的 `post-commit` 钩子在每次提交后自动运行 `codegraph sync`（增量 / 廉价）。
- 钩子**严格非阻塞**：`codegraph` 缺失或出错都不影响提交（退出码恒为 0）。
- 在相应工程的**仓库根目录**触发 `git commit` 时即生效；无需手动干预。

### 5.3 常用命令速查

| 场景 | 命令 |
|---|---|
| 索引健康检查 | `codegraph status`（Journal 应为 `wal`） |
| 探索一个功能区域 | `codegraph context "<任务描述>"` |
| 查找符号定义 | `codegraph query <符号名>` |
| 谁调用了 X / X 调用了谁 | `codegraph callers <符号>` / `codegraph callees <符号>` |
| 变更影响范围 | `codegraph impact <符号>` |
| 变更影响的测试 | `codegraph affected <文件列表>` |
| 手动增量更新 | `codegraph sync` |
| 首次（重）索引 | `codegraph init -i` |

> **父工程索引包含子模块源码**：`pi-workspace` 的 `.codegraph/` 已把全部子模块源码索引在内，
> 在父目录即可跨工程查询（如 `codegraph query artifact` 命中 `pi-desktop/src/main/artifact.ts`）。
> 查询不到时先 `codegraph sync`，仍查不到再用 `codegraph files --pattern` 定位或回退 grep。

## 六、交互与代码质量规范

- 与用户交互、文档、提交注释用**中文**；代码 / 变量 / 技术术语用**英文**。
- 严禁提交密钥 / Token / 密码 / 私钥（`.env`、`*.local.json`、`signing.*.json`、`*.keystore`、`keystore.properties` 等）。
- 严禁 `as any` / `@ts-ignore` / `@ts-expect-error` 压制类型错误；严禁空的 `catch(e) {}`。
- 修复 Bug 只做最小改动，禁止顺手重构；禁止臆测未读取的代码。
- 修改源码后如有 `npm run check` / 类型检查命令，先跑确保无新增错误再提交。
- 调试遵循**系统化调试**（先根因调查，严禁在未查明根因前提出修复方案）。

## 七、提交纪律（Conventional Commits）

- 提交信息 `<type>[可选 scope]: <描述>`；允许 type：`feat`/`fix`/`docs`/`style`/`refactor`/`perf`/`test`/`build`/`ci`/`chore`/`revert`。
- description 用现在时、祈使句，≤72 字符；破坏性变更加 `!` 或 footer `BREAKING CHANGE:`。
- **严禁** `--no-verify` 跳过 Git Hooks；严禁使用表外 type。
- 只暂存本会话改动的文件，提交前 `git status` 核对。

## 八、维护者与更新

> 维护者：fellow99 · 规范随仓库演变持续维护；优先回引各子工程 `AGENTS.md` 与其 `docs/` / `specs/` 权威文档。
