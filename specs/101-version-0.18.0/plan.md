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

（执行后回填）

## 4. 测试

（执行后回填）

## 5. 待办 / 已知限制

- 本机无测试条件：真机安装、签名 HAP 复验、功能与回归测试转交另一台机（见交接文档）。
