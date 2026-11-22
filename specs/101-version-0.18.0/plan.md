# 101 — 版本更新：0.18.0（version）跨工程技术方案与执行结果

> **Module:** 101-version-0.18.0 ｜ **Status:** In-progress ｜ **Last Updated:** 2026-11-22
> **对应规格:** [`spec.md`](./spec.md)

---

## 1. 执行顺序

1. **环境评估**：`pi` @v1.6.0（Branch_v1.6.0）、`pi-web` @v0.18.0（Branch_v0.18.0）已由用户固化；四子工程均在 v0.17.1 线。
2. **冲突评估**：pi-web v0.17.0→v0.18.0（127 文件）与 26 条补丁 hunk 的累加 dry-run → 4 条 hunk / 3 处文件冲突。
3. **向用户汇报冲突 → 批准 rebase 策略与 PlantUML 处置（选项 A：零补丁、仅文档标注）**。
4. **补丁 rebase** → `patches/v0.18.0/`（UTF-8/LF）。
5. **逐工程**：清理 → 版本号 0.18.0/1800 → 上游钉住 → 构建 → 冒烟 → 提交。
6. **官网 + README + CI** 更新。
7. **Code Review**（requesting-code-review → receiving-code-review）→ 修复。
8. **打 tag v0.18.0**（五子工程 + 父工程收尾）+ 测试交接文档。

## 2. 冲突评估（dry-run 实测）

方法：`git -C ../pi-web worktree add --detach ../.tmp/wt-018 v0.18.0`，在 worktree 内按文件名排序逐条
`git apply --check` → `git apply` 累加。为排除行尾干扰，补丁先 `tr -d '\r'` 归一为 LF 再测。

### 2.1 实测结果

| 工程 | 补丁 | 结果 |
|---|---|---|
| pi-desktop | `0001-sync-daemon-settings-tab` | 干净 |
| pi-desktop-hos | `0001-workspace-picker-roots` | 干净 |
| | `0002-session-export-in-process-fallback` | **冲突** |
| | `0003-system-back`、`0004-sync-daemon-settings-tab` | 干净 |
| pi-desktop-ark | `0001-terminal-capability-unsupported` | **冲突** |
| | `0002`、`0003`、`0005-directory-picker` | 干净 |
| | `0004-session-export-runtime-identity-guard` | **冲突** |
| | `0007-sync-daemon-settings-tab`、`0008-system-back` | 干净 |
| pi-desktop-android | `0001`、`0004` | **冲突** |
| | `0002`、`0003`、`0005`、`0007`、`0008` | 干净 |

（对照实验：同套补丁在 v0.17.0 worktree 上以 LF 归一后 **全部干净**，证明冲突确由 0.18.0 引入，非测试方法误判。）

### 2.2 根因与处置

| 冲突 | 上游变更 | 处置 |
|---|---|---|
| HOS `0002` / Ark `0004` / Android `0004`（导出路由） | #1695 给动态 import 加 `/* webpackIgnore: true */`，与我方补丁**同一处同一写法** | **删除我方该 hunk**（含 4 行中文注释）。HOS `0002` 保留首 hunk（`existsSync(process.execPath)` 守卫）；Ark/Android `0004` 保留首 hunk（`runtimeIsSpawnable` 守卫）+ 保留 `console.warn` 诊断 |
| Ark/Android `0001`（`/api/terminal`） | #1750 顶部新增 `import { cwdMissingBody }`，并把 `stat(cwd)` 包进 try/catch 返回 400 | 仅**重定位**：能力门仍插在 `export const runtime` 之后、`POST` 之前；`GET()` 能力查询保留。语义与上游不重叠（上游判 cwd 存在性，我方判 PTY 能力） |

### 2.3 功能层面评估结论

**无功能性冲突。** 逐条见 spec §FR-101-180-005；两条需要文档提示的新增行为：

1. `pi` 1.6.0 的 `find` 工具路径模式改为**相对搜索目录**（`src/*.ts` 不再匹配 `pkg/src/a.ts`）—— 影响 Agent 检索行为，非四端代码。
2. PlantUML 按钮会把图源码编码进 URL 发往 `www.plantuml.com`（仅点击后）—— 决策：保留上游原样，零补丁，仅文档标注。

## 3. 各工程执行结果

### 3.1 补丁 rebase

| 工程 | 目录 | 条数 | 冲突处置 | 验证 |
|---|---|---|---|---|
| pi-desktop | `patches/v0.18.0/` | 1 | 无冲突，按 v0.18.0 上下文重出 | apply→reverse 后 `git status` 空 |
| pi-desktop-hos | `patches/v0.18.0/` | 4 | `0002` 删除冗余 `webpackIgnore` hunk（1.8 KB → 965 B） | 同上；prepare-next 往返校验 7 文件 |
| pi-desktop-ark | `patches/v0.18.0/` | 7 | `0001` 重定位（上游 #1750）；`0004` 删除冗余 hunk 片段 | 同上；assemble-payload 往返校验 2/2/4/1/7/7/1 文件 |
| pi-desktop-android | `patches/v0.18.0/` | 7 | 同 ark | 同上 |

全部 19 条为 **UTF-8/LF**（`grep -c $'\r'` = 0），v0.17.0 的 18 条 CRLF 隐患就此关闭。

### 3.2 各工程构建

- **pi-desktop**：version `0.18.0`；`npm run make` 通过，`BUILD_ID = jzNmTY0R2vE5oNxj97v6w`；
  `Pi Desktop-0.18.0 Setup.exe` / `pi_desktop-0.18.0-full.nupkg` / `Pi Desktop-win32-x64-0.18.0.zip`；
  `smoke-desktop` **10 PASS / 0 FAIL**（服务端横幅 `[next] ▲ Next.js 16.4.6`）；
  `probe-renderer` **未执行** —— 本会话宿主进程即运行中的 pi-desktop，带 `--remote-debugging-port` 重启会中断会话，
  转交测试机（`logs/20261122-1/probe.log` 里那次失败的尝试即由此产生，不代表探针本身损坏）。
- **pi-desktop-hos**：`0.18.0` / `1800`；`prepare-next` BUILD_ID `J85eP85kiaHSlf4B66vUq`（4 补丁应用后回退）；
  `collect-pi` 归档 154.8 MB；`collect-app` 投放 10 文件（**含 `sync-runtime.js`**，相对 require 完整性 2/2）；
  HAP 未签名 345 MB，`pack.info` = `1800/0.18.0`。`smoke-node22 --allow-any-node` **8 PASS / 2 SKIP / 0 FAIL**。
- **pi-desktop-ark**：`0.18.0` / `1800`；`assemble-payload --apply` 归档 80.3 MiB、SHA-256 `7c310b7b…`、
  7 条补丁标记全命中；归档内 `next=16.4.6` / `pi-coding-agent=1.6.0` / `pi-web=0.18.0`；
  HAP 未签名 135.7 MB，`pack.info` = `1800/0.18.0`。
- **pi-desktop-android**：`0.18.0` / `1800`；`assemble-payload --apply` 归档 76.8 MiB、7 条标记全命中；
  归档内 `next=16.4.6` / `pi-coding-agent=1.6.0` / `pi-web=0.18.0`；
  `assembleDebug` 482 MB + `assembleRelease` 473 MB，`aapt` = `1800/0.18.0`。
- **pi-desktop-website**：两页 + README 事实表更新至 `v0.18.0`（补丁数口径 1 / 7 / 4 / 7，列序 Desktop·Android·HOS·Ark）。

### 3.3 Code Review 阶段发现并修复的缺陷

`requesting-code-review` 分派出 reviewer，`receiving-code-review` 逐条核验后：

| 级别 | 缺陷 | 处置 |
|---|---|---|
| **Critical** | 首轮四端产物仍携带 **v0.17.0 的依赖树**（`next 16.4.4` / `pi 1.5.2`）：`prepare-next` 以「`node_modules` 存在」判定跳过 `npm ci`，而上游 tag 刚升版 → 整轮升级静默落空 | 在 `../pi-web` 执行 `npm ci`；给 `pi-desktop` 与 `pi-desktop-hos` 的 `prepare-next.mjs` 增加 `installedVersionsStale()` 门禁（与 `package-lock.json` 逐条比对，不一致即强制重装）；四端全部重建并逐个核对产物内版本 |
| **Critical** | HOS HAP **无法启动**：`main.js` 顶层 `require('./sync-runtime.js')`，但 `collect-app.mjs` 的 `mainFiles` 清单仍是 3 文件，投放目录缺该模块 → `MODULE_NOT_FOUND` | `mainFiles` 增加 `sync-runtime.js`；新增完整性门禁（扫描 `main.js` 全部相对 `require` 并断言在投放目录存在）；重建 HAP，`pack.info` 与 app 目录 10 文件已复核 |
| Important | `test-cases.md` B–G 的「通过」被读成实测结果（实际未测） | 重写为「预期 / 结果」两列，B–G 全部 `⬜ 待测试机执行`，A 段回填实测结论，并补 TC-A-10/11、TC-B-16/17 |
| Important | 官网/README 缺 FR-101-180-006 的 PlantUML 外发提示 | 官网两页 + `pi-desktop/README{,_zh}.md` 各加一条明确声明 |
| Important | 父工程 README 补丁数行仍为 0/2/6/6；4 处 `patches/README.md` 回退命令仍指向 `v0.17.0/`；HOS/Ark 注册表头部口径陈旧；Android README 补丁表列了已并入的 `0006` 却缺 `0007/0008` | 逐条改正 |
| Important | 四个 `AGENTS.md` 的版本/上游基线/补丁数陈旧（android 更是停在 0.13.0） | 全部更新（本项是**计划缺口**：T7-2 只列了 `README*.md`） |
| Minor | 补丁生成用 worktree 残留在 `.tmp/` 且未忽略；website README 引用不入库的 `upstream.lock.json` | 清理 worktree、`.tmp/` 加入 `.gitignore`、事实来源改为 `git describe` |

对 reviewer 的两点**技术性反驳**：
1. 「本轮提交把 301 的 `src-main/main.js` 源码改动一并夹带」**不成立** —— 该 `require` 来自既有提交 `9415ef3`（feat(301)），
   本轮只重新生成了**派生**文件 `web_engine/.../resfile/resources/app/main.js`，而该派生文件此前是陈旧的（不含 require），
   正因为它陈旧，v0.17.0 时代的 HAP 才没有暴露缺 `sync-runtime.js` 的问题。真正的缺陷是 `collect-app` 的清单（已按 C2 修复）。
2. 「`probe-renderer` 被当作 skipped 记录会让人以为探针坏了」—— 接受措辞修正（已在 §3.2 写明真实原因），但结论不变：
   本机无法执行该探针。

## 4. 测试

- **本机可测项（A 段）全绿**：见 [`test-cases.md`](./test-cases.md) A 段 11 条（含新增的产物依赖版本门禁、HOS 载荷完整性门禁）。
- **B–G 段（真机 / 交互 / 官网目视）全部 ⬜ 待另一台机执行**：交接文档
  `D:\knowledge-base\pi-workspace\011-version-0.18.0-测试交接.md`。
- 敏感信息：本轮未落盘任何凭证；模型 Key 仅运行期注入。

## 5. 待办 / 已知限制

- 本机无测试条件：真机安装、签名 HAP 复验、功能与回归测试转交另一台机（见交接文档）。
