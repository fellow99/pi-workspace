# 101 — 版本更新：0.19.0（version）跨工程技术方案与执行结果

> **Module:** 101-version-0.19.0 ｜ **Status:** In-progress ｜ **Last Updated:** 2026-11-27
> **对应规格:** [`spec.md`](./spec.md) ｜ **任务清单:** [`tasks.md`](./tasks.md) ｜ **用例:** [`test-cases.md`](./test-cases.md)
> **日志目录:** 父工程 `logs/20261127-1/`（不入库）

---

## 1. 执行顺序

1. **环境评估**：`pi` @ `Branch_v1.6.1` / `v1.6.1`（`ef4a1cfc`）、`pi-web` @ `Branch_v0.19.0` / `v0.19.0`（`7b77aa1a`）已由用户固化；五子工程均在 v0.18.0 线、分支 `main`（本轮**不改分支**）。
2. **冲突评估**：读 GitHub Release v0.19.0 中文正文 + `git diff v0.18.0 v0.19.0`（284 文件 / +8976−2383 / 44 提交）+ pi v1.6.1 CHANGELOG；对 16 条补丁做 detached worktree 累加 dry-run。
3. **向用户汇报冲突 → 获批准**（结论：零阻断性冲突，冲突均为补丁上下文漂移）。
4. **补丁 rebase** → 四个 `patches/v0.19.0/`（UTF-8 / LF），逐端 `apply --check` + apply→reverse 还原验证。
5. **逐工程**：清理 → 版本号 0.19.0 / 1900 → 上游 tag 常量 → 构建 → 冒烟 → 产物断言。
6. **官网 + README + AGENTS + CI** 更新。
7. **Code Review**（`requesting-code-review` → `receiving-code-review`）→ 修复。
8. **测试**：Windows 本机（受单实例锁限制）、Android 真机（被 MIUI 拦）、鸿蒙（无环境）——边界与结论见 §5。
9. **打 tag `v0.19.0`**（五子工程 + 父工程），推送触发 CI。

## 2. 冲突评估（dry-run 实测）

方法：`git -C ../pi-web worktree add --detach ../.tmp/wt-019 v0.19.0`，在 worktree 内按文件名排序逐条
`git apply --check` → `git apply` 累加；四端各自重置后独立测。

### 2.1 实测结果

| 工程 | 补丁 | 结果 |
|---|---|---|
| pi-desktop | `0001-sync-daemon-settings-tab` | **冲突**（`components/SettingsPanel.tsx` 导入上下文） |
| pi-desktop-hos | `0001-workspace-picker-roots` | **冲突**（`components/DirectoryPicker.tsx`） |
| | `0002-session-export-in-process-fallback` | 干净 |
| | `0003-system-back` | 干净 |
| pi-desktop-ark | `0001-terminal-capability-unsupported` | **冲突**（`app/api/terminal/route.ts` 导入位置） |
| | `0002` / `0003` / `0004` | 干净 |
| | `0005-directory-picker` | **冲突**（`components/DirectoryPicker.tsx`） |
| | `0008-system-back` | 干净 |
| pi-desktop-android | `0001` / `0005` | **冲突**（同 Ark） |
| | `0002` / `0003` / `0004` / `0008` | 干净 |

**合计 16 条中 6 条冲突**（10 条干净）：desktop `0001`、hos `0001`、ark `0001` + `0005`、android `0001` + `0005`。

### 2.2 根因与处置

根因只有 2 个，全部来自上游 v0.19.0 的**内部重构**（#1757–#1767「合并重复的辅助函数、类型和请求封装」）：

| 冲突 | 上游变更 | 处置 |
|---|---|---|
| HOS `0001` / Ark `0005` / Android `0005`（`DirectoryPicker.tsx`） | 上游把 `createDirectory` 改用 `lib/fetch-json.ts` 的 `readJsonOrThrow`，并把两处 `cause instanceof Error ? … : String(cause)` 改为 `lib/errors.ts` 的 `errorMessage(cause)`，同时新增两条 import | `git apply --exclude=components/DirectoryPicker.tsx` 应用其余文件后，在 v0.19.0 上**重放该文件并采纳上游 5 处改动**（2 条 import + `createDirectory` 改用 `readJsonOrThrow` + 2 处 `errorMessage(cause)`）。但 `loadDirectories` **不采纳** `readJsonOrThrow`：本补丁要从错误响应体读 `code` / `permType` / `permission` / `canRequest` / `roots`，而 `readJsonOrThrow` 抛出后拿不到响应体 | 
| Ark / Android `0001`（`app/api/terminal/route.ts`） | 上游新增 `import { errorMessage } from "@/lib/errors"`，正好插在本补丁插入 `terminalCapability` 导入的位置 | 仅把 `terminalCapability` 导入移到 `errorMessage` 之后；`GET()` 能力查询与 `POST` 能力门位置不变 |
| pi-desktop `0001`（`SettingsPanel.tsx`） | 上游新增 `HttpProxyForm` / `readJsonOrThrow` / `errorMessage` 三条 import，破坏了首个 hunk 的「`McpConfig` → `ConfigButton`」相邻上下文 | `--exclude=components/SettingsPanel.tsx` 应用其余 5 个文件后，手工重放该文件 6 处插入（导入 / `sync` 图标 / `useState` 惰性初值 / 分区枚举 / `setLastSettingsSection` 守卫 / `sectionHost("sync")`），语义逐字不变 |

### 2.3 功能层面评估结论

**无功能性冲突**（逐条依据见 `spec.md` FR-101-190-005）。本轮三个需要在文档中明确的新事实：

1. **`#1827` iframe 防嵌套**：四端都是顶层加载，不收 `X-Frame-Options: SAMEORIGIN` 影响；**不设 `PI_WEB_FRAME_ANCESTORS`**（保持默认 `'self'`，安全面最小）。
2. **`pi` 1.6.1 的 `node:zlib.crc32` 是新的 Node 下限（≥ 22.2）**：本机唯一的 Node 22 是 **22.1.0**，
   `next build` 在「Collecting page data」阶段报 `SyntaxError: The requested module 'node:zlib' does not provide an export named 'crc32'`
   （来自 `pi/packages/coding-agent/src/utils/zip.ts`）→ HOS 构建改用 **Node 24.7.0** 后通过。
   设备运行时不受影响（Electron 37 = 22.17.0 / Ark 24.2.0 / Android 26.13.0）。
3. **`#1828` `PI_OFFLINE` 统一为 `!== undefined`**：ark 设 `PI_OFFLINE=1`（不变）；其余三端**不设**该变量，
   只设 `PI_WEB_SKIP_VERSION_CHECK=1`（仍是 app-update 的短路条件）。

## 3. 各工程执行结果

### 3.1 补丁 rebase

| 工程 | 目录 | 条数 | 冲突处置 | 验证 |
|---|---|---|---|---|
| pi-desktop | `patches/v0.19.0/` | 1 | 无冲突，按 v0.19.0 上下文重出 | apply→reverse 后 `git status` 空 |
| pi-desktop-hos | `patches/v0.19.0/` | 3 | `0001` 重放并采纳上游 5 处改动；`0002`/`0003` 逐字节沿用 | 同上；`prepare-next` 往返校验 7/1/1 文件 |
| pi-desktop-ark | `patches/v0.19.0/` | 6 | `0001` 导入后移；`0005` 同 HOS；`0003`/`0004`/`0008` 无冲突但**重生成**（hunk 内容逐字不变，`index` 与行号随上游重排）；`0002` 逐字节相同 | 同上；`assemble-payload` 往返校验 2/2/4/1/7/1 文件 |
| pi-desktop-android | `patches/v0.19.0/` | 6 | 同 Ark（`0002` 逐字节相同；`0003`/`0004`/`0008` 重生成） | 同上；`assemble-payload` 往返校验 2/2/4/1/7/1 文件 |

全部 16 条为 **UTF-8 / LF**；在 pristine v0.19.0 上按序 apply → 反序 reverse 后 `git status --porcelain` 为空；
四次构建（desktop / hos / ark / android）结束后 `../pi`、`../pi-web` 均 pristine。

### 3.2 各工程构建

- **pi-desktop**：version `0.19.0`；`npm ci`（`../pi-web`）必须执行（升级前安装树仍是 `@earendil-works/*@1.6.0`）；
  `prepare-next` → `next build` **11.0 min**（TypeScript 22.2 s）→ **BUILD_ID `vxIdm0QC9upmemdyHwCDn`**；
  `collect-pi` → `pi-dist.tar.gz` **207.1 MB** / SHA-256 `2e2381fa…`（包内 `pi-web 0.19.0` / `next 16.4.6` / `pi-coding-agent 1.6.1`）；
  `smoke-desktop` **10 PASS / 0 FAIL**；`npx electron-forge make` →
  `Pi Desktop-0.19.0 Setup.exe` 363,840,000 B / `pi_desktop-0.19.0-full.nupkg` 362,882,768 B / `Pi Desktop-win32-x64-0.19.0.zip` 368,292,670 B。
  打包态自测（`--user-data-dir=<tmp>` + `--remote-debugging-port=9222`）：`probe-renderer` **7 PASS / 0 FAIL**；
  版本徽标 **`web v0.19.0 · pi v1.6.1`**；DeepSeek Key `{success:true}`；`/api/cwd/validate` 200；
  真实 DeepSeek 文本对话 `stopReason=stop` / 825 字符（密钥仅运行期注入）；图片分析对话未在本机跑。
- **pi-desktop-hos**：`0.19.0` / `1900`；`collect-runtime` 全项 PASS；`prepare-next`（Node 24.7.0 + `--max-old-space-size=12288`）
  BUILD_ID `q-l7KIxPhITdBEhOomW4T`，3 补丁往返校验通过后回退；`smoke-node22 --allow-any-node` **8 PASS / 2 SKIP / 0 FAIL**；
  `collect-pi` 归档 **160.9 MB** / SHA-256 `ff8e8a3d…`；`collect-app` 投放 9 文件（主进程相对 `require` 1/1 在位，
  0.18.0 起 301 退场后不再需要 `sync-runtime.js`）；HAP **debug 签名通过 382,924,600 B**，
  `pack.info` = `1900/0.19.0`，内嵌 `pi-dist.version` 与 `collect-pi` SHA-256 一致。
- **pi-desktop-ark**：`0.19.0` / `1900`；6 条补丁往返校验通过 + 依赖门禁（精确钉住 19 项）通过；
  载荷刷新判定为「缺补丁标记 ⇒ 重编」→ `next build` 678 s；归档 **89,830,460 B（85.7 MiB）** / SHA-256 `07c79770…`；
  HAP **debug 签名通过 136,077,761 B**，`pack.info` = `1900/0.19.0`。
- **pi-desktop-android**：`0.19.0` / `1900`；6 条补丁往返校验通过 + 依赖门禁通过；
  载荷归档 **76.8 MiB** / SHA-256 `a5ac8c46…`（包内 `pi-web 0.19.0` / `next 16.4.6` / `pi-coding-agent 1.6.1`）；
  `assembleDebug` **482,106,993 B** + `assembleRelease` **473,074,747 B**，`aapt` = `1900 / 0.19.0`。

### 3.3 本轮踩到并在过程中修掉的两个构建缺陷

| 级别 | 缺陷 | 处置 |
|---|---|---|
| **Critical** | **Android 载荷可能是 0.18.0 的旧编译产物**：`assemble-payload` 的新鲜度判据是「补丁标记是否在 `.next` 里命中」，而 `build/pi-dist` 里躺着上一轮 0.18.0 的暂存（同名标记跨版本必然命中）→ 首跑输出「载荷断言：补丁标记全部在位（跳过重编）✔」，会把 0.18.0 的 `.next` 打进 0.19.0 的包 | 清理 `build/`、`app/build/`、`.gradle/`、`app/src/main/assets/pi-web/` 后重跑；重建后**逐项断言归档内的 `pi-web` / `next` / `pi-coding-agent` 版本**（0.19.0 / 16.4.6 / 1.6.1） |
| Important | **HOS `prepare-next` 用 Node 22.1.0 会在 `Collecting page data` 阶段崩溃**（两种形态：先 `JavaScript heap out of memory`（4 GB 堆），加大堆后暴露真因 `node:zlib` 无 `crc32` 导出） | 改用 Node 24.7.0（= `pi` 1.6.1 要求的 Node ≥ 22.2 满足），并保留 `--max-old-space-size=12288`；文档记明该 Node 下限 |

### 3.4 Code Review 阶段

`requesting-code-review` 分派 reviewer（附 spec / `plan.md` §2 冲突评估 / 四端 diff 范围 / 构建日志摘要），
`receiving-code-review` 逐条核验后形成的结论见 [`tasks.md`](./tasks.md) 阶段 4 与 `logs/20261127-1/REVIEW_REPORT.md`。

## 4. 文档与 CI

- `pi-desktop/.github/workflows/{ci,release}.yml`：`PI_REF`/`PI_SHA` → `v1.6.1`/`ef4a1cfcca62b77ad3558337ff3e1c7f6beeadfe`；
  `PI_WEB_REF`/`PI_WEB_SHA` → `v0.19.0`/`7b77aa1a01a74597280dd5bbc9221195411bddf4`。
- `pi-desktop-android/.github/workflows/build-release.yml`：`PI_WEB_TAG` → `v0.19.0`（该流水线不钉 `pi`，`pi` 随 pi-web 的 `node_modules`）。
- 官网两页（`index.html` / `index_zh.html`）与 `README.md`：版本 / versionCode / 补丁数 / 上游基线 / 各版状态一并通过。
- 父工程 `README.md` / `README_zh.md`、`docs/能力对照清单.md`（新增 §19）、四子工程 `README*.md` + `AGENTS.md`。
- 四端 `patches/README.md` 各新增「0.0 v0.19.0 rebase 记录（2026-11-27）」，旧的 v0.18.0 记录顺延为 `0.0b` 保留可追溯。

## 5. 测试

用户裁定：Windows 不等待本机测试，打 tag 走 CI，安装包出来后由用户手动验证；鸿蒙本轮无测试环境；**本机只做安卓真机**。
实际执行结果：

- **Windows 打包态**：发现了绕过单实例锁的办法（`--user-data-dir`），因此补做了 `probe-renderer` **7 PASS / 0 FAIL**
  与基本功能 API 级验证（Key / 工作区 / 真实文本对话）；**图片分析对话与安装-启动-卸载仍转用户手动验证**。
- **Android**：APK 已出，但装机被设备 MIUI「USB 安装」限制拦下（`INSTALL_FAILED_USER_RESTRICTED`），未上机。
- **鸿蒙两版**：只构建 + debug 签名，未上机。

逐条结论见 [`test-cases.md`](./test-cases.md)。

## 6. 待办 / 已知限制

1. **Android 真机复验未完成**：需在设备上开启「开发者选项 → USB 安装」（需登录小米账号），或改用设备端文件管理器安装 APK。
   另：设备上既有应用是 **0.15.0 / `org.fellow99.pi.PiDesktopAndroid`**，而 0.17.0 起包名改为 `top.fellow99.pi.PiDesktopAndroid`
   ⇒ 只能**并存安装**，无法覆盖升级；本轮**未卸载任何既有应用**（按用户要求）。
2. **Windows 打包态自测**：需要一台没有运行 Pi Desktop 的 Windows 机器（或先退出宿主进程）才能跑 `probe-renderer` 与打包态 e2e。
3. **鸿蒙两版**：HAP 已签名（debug Profile），但本轮无设备可装。
4. `runtime.lock.json` 的引擎校验（`provision-engine --check`）在本机不匹配：Gradle 实际用的是 Maven Central 的
   `node-runtime:26.13.0` AAR，`jniLibs` 里是历史遗留的旧引擎文件；不影响构建，但该 `--check` 已失去判据意义（下轮清理或改为校验 AAR 坐标）。
