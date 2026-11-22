# 101 — 版本更新：0.18.0（version）跨工程功能规格

> **Module:** version ｜ **Status:** In-progress ｜ **Last Updated:** 2026-11-22
> **上游基线:** `pi` v1.6.0（`faac585884ae2c013a6b1400f3e9e77f10b34bac`，Branch_v1.6.0）· `pi-web` v0.18.0（`2daa0d131eae6c2fd8c2a80872eee01ea3c7a0a1`，Branch_v0.18.0）
> **发布 tag:** `v0.18.0`
> **适用范围:** `pi-desktop` · `pi-desktop-hos` · `pi-desktop-ark` · `pi-desktop-android` · `pi-desktop-website`
> **父/子分工:** 本文写**跨工程版本口径与矩阵**；各子工程的升级步骤、补丁、构建证据写在各自提交与 `patches/v0.18.0/`。

---

## 1. 模块概述

### 1.1 目的

把五个工程统一收敛到 `v0.18.0` 基线，并把上游基线升到 `pi` v1.6.0 + `pi-web` v0.18.0。

### 1.2 解决的问题

- **上游 `pi-web` v0.17.0 → v0.18.0**：127 文件，+4874/−946，34 提交。功能要点（详见 plan §2 冲突评估）：
  - `pi` 升 **1.6.0**（GPT-6.2 Astra/Sol；`find` 工具路径模式改为相对搜索目录；undici 8.12.4 安全更新；扩展组件 `render()` 破坏性变更）；
  - Next **16.4.4 → 16.4.6**（#1738 修 `/api/sessions` 双 gzip、文件响应不再被 Next 压缩）；
  - 后台子 Agent 报告改为 **steer 进父 Agent 正在进行的运行**（#1742）；
  - 新增 **PlantUML「在 plantuml.com 渲染」**（#1741，点击后才外发）；
  - **`webpackIgnore` 修 HTML 导出**（#1695）——与本公司 HOS/Ark/Android 补丁**同一处修复重合**；
  - `/api/terminal` 新增 **cwd 缺失 400 分支**（#1750）；
  - 安全：项目文件夹 git 子进程 / 插件更新检查**擦除宿主环境**（#1732/#1693）；登录并发限流（#1694）；
  - 性能：长会话选取加速、会话搜索缓存、侧边栏只读 4 KB 会话头（#1692/#1755）；
  - 「在文件管理器中打开」WSL 下改用 Windows 资源管理器（#1755）。
- **补丁冲突（dry-run 实测，v0.18.0 detached worktree）**：26 条 hunk 中 22 条干净、**4 条（3 处文件）冲突**：
  HOS `0002`、Ark `0001`/`0004`、Android `0001`/`0004`。根因只有 2 个：
  ① 上游 #1695 与我们 HOS/Ark/Android 补丁**写了同一个 `webpackIgnore: true`** → 我方 hunk 变冗余，rebase 时删除；
  ② 上游 #1750 改动 `/api/terminal` 顶部（`import cwdMissingBody` + `stat` try/catch）→ 我方能力门 hunk 需重定位。
- **补丁行尾历史隐患**：v0.17.0 的 19 条补丁中 18 条为 CRLF（仅 `0007-sync-daemon` 与 `pi-desktop/0001` 为 LF）。
  在 LF 工作树（WSL/Linux）下 `git apply` 对 CRLF 补丁**必然失败**；只有 Windows Git-for-Windows（`core.autocrlf=true`）能容错。
  本轮 `patches/v0.18.0/` **统一生成 UTF-8/LF**，与 `patches/README.md` 纪律一致，并在 LF 环境下逐条验证可应用。

### 1.3 范围

**包含**：五版上游基线一致性（pi v1.6.0、pi-web v0.18.0）；应用版本/versionCode/产物/tag 矩阵；
`patches/v0.18.0/` rebase 与幂等验证；四端清理中间/临时文件；四端构建产物；官网/README 更新；CI 上游钉住更新；
父工程与各子工程 tag `v0.18.0`；测试交接文档。
**不包含**：修改上游源码（只允许幂等补丁）；签名 HAP；鸿蒙/安卓真机复验（本机无测试条件，转交另一台机）。

---

## 2. 需求条目

### FR-101-180-001 上游基线一致性
`pi` 工作树必须位于 `Branch_v1.6.0` / tag `v1.6.0`；`pi-web` 必须位于 `Branch_v0.18.0` / tag `v0.18.0`。
四端补丁目录名为 `patches/v0.18.0/`，`patches/README.md`、`scripts/*.mjs` 中的上游 tag 常量同步。

### FR-101-180-002 补丁集 rebase 与幂等
每条补丁必须满足既有的 8 条补丁纪律（幂等、冲突即终止、不带镜像噪声、按上游 tag 组织、不落在既有连续上下文、
可往返 SHA-256 还原、只在 detached worktree 生成、登记三要素）。新增/修改补丁必须在 `patches/README.md` 登记。
**本轮额外要求**：所有补丁文件为 UTF-8/LF；在 LF 工作树上 `git apply --check` 必须全绿。

### FR-101-180-003 版本号与产物矩阵
四端应用内版本 `0.18.0`、versionCode `1800`；产物名/tag 与矩阵一致（见 §3.2）。

### FR-101-180-004 清理中间/临时文件
构建前清理各工程构建产物与临时目录（各工程 `.gitignore` 已列明的派生目录），不删除源码、不删除已入库文件、
不删除签名材料与本地配置。

### FR-101-180-005 功能零冲突约束
四端**不得**为适配 0.18.0 而新增除补丁 rebase 之外的源码改动。0.18.0 的功能变更对四端均为继承性受益或与本机无关：
- `find` 路径模式语义变化：四端文档/skills 无依赖旧语义的写法（已核查），仅在交接文档提示；
- 扩展组件 `render()` 破坏性变更：四端扩展未使用 pi 扩展 UI 组件 `render()`（已核查）；
- Next 16.4.6：`next.config.ts` 未变，四端壳不接管压缩；四端 Node 均满足要求（HOS/Desktop 随 Electron，Ark 24.2.0，Android 26.13.0）；
- 宿主环境擦除：四端依赖的 `PI_CODING_AGENT_DIR` / `PI_WEB_APP_NAME` / `PI_WEB_SKIP_VERSION_CHECK` 契约在 v0.18.0 仍被读取；
- 登录限流：桌面壳走镜像 token / Basic 头，不经 `/api/web-auth`；
- 「在文件管理器中打开」WSL 分支：四端均不设 `WSL_DISTRO_NAME`。

### FR-101-180-006 PlantUML 外部出网（决策：选项 A）
`pi-web` v0.18.0 新增的「在 plantuml.com 渲染」按钮**保留上游原样**，四端**不写补丁**。
仅在官网/README/测试交接文档中标注：该按钮会把图源码编码进 URL 发往第三方 `www.plantuml.com`，涉密内容勿点。

### FR-101-180-007 官网与 README 更新
`pi-desktop-website` 两页（`index.html` / `index_zh.html`，含能力矩阵）与父/子工程 README 的版本号、版本描述、
补丁数、上游基线更新到 0.18.0 口径。

### FR-101-180-008 CI 上游钉住更新
`pi-desktop/.github/workflows/{ci,release}.yml` 的 `PI_REF`/`PI_WEB_REF`/`PI_SHA`/`PI_WEB_SHA`；
`pi-desktop-android/.github/workflows/build-release.yml` 的 `PI_WEB_TAG`（及 pi 钉住项，如存在）更新到 v1.6.0 / v0.18.0。

### FR-101-180-009 提交与 tag 纪律
提交信息使用 Conventional Commits 前缀（`build`/`ci`/`docs`/`fix`/`chore` 等），提交用户 `fellow99 <fellow99@163.com>`；
只暂存本会话改动文件（严禁 `git add -A`）；各子工程与父工程打 tag `v0.18.0`。

### FR-101-180-010 测试交接
本机无测试条件。测试阶段产出交接文档 `D:\knowledge-base\pi-workspace\011-version-0.18.0-测试交接.md`，
含产物路径与 SHA256、安装/启动步骤、测试用例（含 0.18.0 新增行为点）、回滚方式。

---

## 3. 版本矩阵

### 3.1 上游

| 上游 | 版本 | SHA / Branch |
|---|---|---|
| `pi` | v1.6.0 | `faac585884ae2c013a6b1400f3e9e77f10b34bac` / Branch_v1.6.0 |
| `pi-web` | v0.18.0 | `2daa0d131eae6c2fd8c2a80872eee01ea3c7a0a1` / Branch_v0.18.0（annotated tag `38bb6805`） |

### 3.2 应用

| 工程 | 版本 | versionCode | 构建产物 |
|---|---|---|---|
| pi-desktop | 0.18.0 | — | Setup.exe / nupkg / zip |
| pi-desktop-hos | 0.18.0 | 1800 | HAP（签名依本机材料） |
| pi-desktop-ark | 0.18.0 | 1800 | HAP（签名依本机材料） |
| pi-desktop-android | 0.18.0 | 1800 | debug / release APK |
| pi-desktop-website | 0.18.0 | — | 静态站 |

### 3.3 补丁矩阵（目标）

| 工程 | 补丁数 | 目录 | 说明 |
|---|---|---|---|
| pi-desktop | 1 | `patches/v0.18.0/` | `0001-sync-daemon-settings-tab`（干净） |
| pi-desktop-hos | 4 | `patches/v0.18.0/` | `0002` 删除冗余 `webpackIgnore` hunk |
| pi-desktop-ark | 7 | `patches/v0.18.0/` | `0001` 重定位；`0004` 删除冗余 hunk |
| pi-desktop-android | 7 | `patches/v0.18.0/` | 同 Ark |

> 补丁**编号只增不改、不复用**：本轮沿用 v0.17.0 的编号（同一补丁的升级版），不新占全局编号；
> `0006` 为空号（v0.17.0 起已合并进 `0005`），保留空号不复用。

---

## 4. 非功能要求

| 项 | 要求 |
|---|---|
| 上游零改动 | `../pi`、`../pi-web` 在构建窗口期之外必须 pristine（`git status` clean） |
| 补丁可往返 | 应用 → 反向 → 受影响文件 SHA-256 逐条还原 |
| 敏感信息 | 不落盘任何 Key / 口令 / 私钥 / keystore |
| 构建可复现 | 补丁按文件名排序应用，结果确定性；`BUILD_ID` 写入日志 |
| 文档一致 | 官网/README/CI 三处版本口径一致，无残留 0.17.0 |

## 5. 边界与假设

- 假设 1：本机 `pi`/`pi-web` 分支已由用户固化（已核实：`Branch_v1.6.0` @ v1.6.0、`Branch_v0.18.0` @ v0.18.0）。
- 假设 2：本机具备 Electron / DevEco(hvigor) / Gradle 工具链；签名材料按各自 `signing.*.local.json` / `keystore.properties` 现状处理。
- 假设 3：真机与签名复验不在本机执行（转交另一台机，见交接文档）。
