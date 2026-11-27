# 101 — 版本更新：0.19.0（version）测试用例

> **对应规格:** [`spec.md`](./spec.md)
> **本轮测试边界（用户裁定）：** Windows 本机受壳**单实例锁**限制，无法与已装的 0.18.0 并行测试 → 打 tag 走 CI，
> 安装包出来后由用户手动验证；**鸿蒙本轮无测试环境**；**本机只做安卓真机**，但装机被设备 MIUI「USB 安装」限制拦下。
> **列说明：** `预期` = 通过标准；`结果` = 本轮实测结论（`✅` 已测通过 · `❌` 已测失败 · `⬜` 未执行 · `🅿` 待用户手动执行）。

## A. 构建与版本口径（本机已测）

| ID | 级别 | 用例 | 预期 | 结果 |
|---|---|---|---|---|
| TC-A-01 | P0 | `pi` `git describe --tags` = `v1.6.1`，分支 `Branch_v1.6.1` | 一致 | ✅ `v1.6.1` / `Branch_v1.6.1` / `ef4a1cfc` |
| TC-A-02 | P0 | `pi-web` `git describe --tags` = `v0.19.0`，分支 `Branch_v0.19.0` | 一致 | ✅ `v0.19.0` / `Branch_v0.19.0` / `7b77aa1a` |
| TC-A-03 | P0 | 四端 `patches/v0.19.0/*.patch` 在 pristine v0.19.0 上按名排序 `git apply --check` 全绿 | 全绿 | ✅ 16 条（1/3/6/6）全绿 |
| TC-A-04 | P0 | 每端补丁往返：apply → 反序 `git apply --reverse` → `git status --porcelain` 为空 | 空 | ✅ 四端 `dirty=0` |
| TC-A-05 | P0 | 四次构建后 `../pi`、`../pi-web` `git status --porcelain` 为空 | 空 | ✅ desktop / hos / ark / android 四次后均为 0 |
| TC-A-06 | P0 | 四端应用版本 = `0.19.0`，HOS/Ark/Android `versionCode` = `1900`，**且产物内嵌 manifest 也必须是 0.19.0** | 一致 | ✅ HOS/Ark `pack.info` = `1900/0.19.0`；APK `aapt` = `1900/0.19.0`；desktop 产物名 `0.19.0`。**HOS 内嵌 `resources/resfile/resources/app/package.json` 首轮仍是 0.18.0（Review Critical，已修并重建为 0.19.0，并加 `collect-app` 版本一致性硬门禁）** |
| TC-A-07 | P0 | **载荷依赖版本 == `pi-web/package-lock.json`**（`next` / `@earendil-works/pi-coding-agent` / `pi-web` 自身） | 一致 | ✅ Desktop `0.19.0/16.4.6/1.6.1`；HOS 归档 SHA 与 `pi-dist.version` 一致；Ark 归档 85.7 MiB；Android 归档 **直读 tar 断言** `0.19.0 / 16.4.6 / 1.6.1` |
| TC-A-08 | P0 | **Android 载荷新鲜度**：重建后归档内版本 = 0.19.0（不是沿用 0.18.0 暂存） | 0.19.0 | ✅ 清理 `build/pi-dist` 后重编；**首跑曾误判「标记全部在位（跳过重编）」→ 已修正并断言** |
| TC-A-09 | P0 | HOS 载荷完整性：`main.js` 的每个相对 `require('./x.js')` 都在投放目录 | 全部在位 | ✅ `collect-app` 报「主进程相对 require 完整性：1 个引用全部在位」 |
| TC-A-10 | P1 | CI workflow 的 `PI_REF`/`PI_SHA`/`PI_WEB_REF`/`PI_WEB_SHA`/`PI_WEB_TAG` 与固化 tag 一致 | 一致 | ✅ 已更新并核对 SHA |
| TC-A-11 | P1 | 仓库内无凭证落盘（Key / 口令 / 私钥 / keystore） | 无 | ✅ 未落盘；`build-profile.json5` 的本地签名改动**未暂存** |
| TC-A-12 | P0 | 官网两页与各级 README 无 `0.18.0` 当轮口径残留 | 无残留 | ✅ `grep` 后仅保留明确的历史轮次叙述（已逐条回改 2 处误改：`0.18.0 轮 run #26`、`构建顺序红线 0.18.0 轮`） |

## B. 桌面版（pi-desktop，Windows x64）

| ID | 级别 | 用例 | 预期 | 结果 |
|---|---|---|---|---|
| TC-B-01 | P0 | `smoke-desktop` 10 PASS / 0 FAIL | 全绿 | ✅ 10 PASS（`[next] ▲ Next.js 16.4.6`，BUILD_ID `vxIdm0QC9upmemdyHwCDn`） |
| TC-B-02 | P1 | `probe-renderer` 7 PASS | 全绿 | ✅ **7 PASS / 0 FAIL**。关键：打包态要用独立 `--user-data-dir=<tmp>` 启动（单实例锁按 userData 区分；`APPDATA` 环境变量无效）。R1–R7 全过，R7 实跑 `echo PI_DESKTOP_PTY_OK`。**注意 attach 必须等首屏渲染完成**（160 节点时 attach 会 R4 FAIL） |
| TC-B-03 | P0 | 打包产物启动，主窗口加载 Pi Web UI，无白屏 | 通过 | ✅ `out/Pi Desktop-win32-x64/` 打包态启动到 `ready-to-show`，渲染层 loopback 且首屏 DOM 701 节点。**安装-启动-卸载（`Setup.exe`）仍 🅿 待用户** |
| TC-B-04 | P0 | 版本徽标显示 `0.19.0` | 通过 | ✅ UI 实测 **`web v0.19.0 · pi v1.6.1`** |
| TC-B-05 | P0 | 设置 DeepSeek API KEY → 成功 | 通过 | ✅ `POST /api/auth/api-key/deepseek` → `{success:true}`（密钥仅运行期注入，未落盘） |
| TC-B-06 | P0 | 创建并采纳测试目录作为工作区 | 通过 | ✅ 建 `.tmp/win-test/ws2`，`POST /api/cwd/validate {cwd}` → 200 `success:true` |
| TC-B-07 | P0 | 文本对话「介绍一下你自己」正常回复 | 通过 | ✅ 真实 DeepSeek，`stopReason=stop`，**825 字符**，回复正确引用工作目录与 `AGENTS.md`（会话文件直读核对） |
| TC-B-08 | P0 | 图片上传 + 图片分析对话正常 | 通过 | 🅿 待用户（本机未跑） |
| TC-B-09 | P1 | 内置终端可执行命令（PTY） | 通过 | ✅ `probe-renderer` R7：PTY 创建 → `echo PI_DESKTOP_PTY_OK` 回显命中 → 删除 |
| TC-B-10 | P1 | **0.19.0 新增**：输入框输入 `#` 弹出会话选择器，引用后发送展开为有界 transcript | 通过 | 🅿 待用户 |
| TC-B-11 | P1 | **0.19.0 新增**：侧边栏时钟按钮切换「按时间排序」（今天/昨天/过去 7 天/更早） | 通过 | 🅿 待用户 |
| TC-B-12 | P1 | **0.19.0 新增**：聊天中 `src/app.ts:42` 链接跳到源码第 42 行并高亮；面板 ≥520px 时 Git diff 双栏 | 通过 | 🅿 待用户 |
| TC-B-13 | P1 | **0.19.0 新增**：设置›通用 出现「引导消息 / 后续消息 / 网络代理 / 提示（四开关）」 | 通过 | 🅿 待用户 |
| TC-B-14 | P1 | **回归**：「中枢服务」设置分区仍在（`0001` 补丁），可开可配 | 通过 | 🅿 待用户 |
| TC-B-15 | P1 | **0.19.0 新增**：折叠的工具步骤标题汇总费用 | 通过 | 🅿 待用户 |
| TC-B-16 | P2 | 会话导出 HTML 成功 | 通过 | 🅿 待用户 |

## C. Electron on HarmonyOS（pi-desktop-hos）

| ID | 级别 | 用例 | 预期 | 结果 |
|---|---|---|---|---|
| TC-C-01 | P0 | `smoke-node22 --allow-any-node` 8 PASS / 2 SKIP / 0 FAIL | 全绿 | ✅ 8/2/0（SSE 探针无真实会话 → SKIP；Node 版本探针 SKIP） |
| TC-C-02 | P0 | HAP 构建并 debug 签名通过，`pack.info` = `1900/0.19.0` | 通过 | ✅ 382,924,600 B |
| TC-C-03 | P0 | 安装 HAP 后启动，首屏渲染正常 | 通过 | ⬜ **本轮无鸿蒙测试环境** |
| TC-C-04 | P0 | 版本徽标 `0.19.0` | 通过 | ⬜ |
| TC-C-05 | P0 | 沙箱工作区授权后可读写（`0001-workspace-picker-roots`） | 通过 | ⬜ |
| TC-C-06 | P0 | 文本对话 + 图片分析对话 | 通过 | ⬜ |
| TC-C-07 | P1 | 会话导出走进程内回退（`0002`） | 通过 | ⬜ |
| TC-C-08 | P1 | 系统返回键（`0003`） | 通过 | ⬜ |

## D. HarmonyOS 版（pi-desktop-ark）

| ID | 级别 | 用例 | 预期 | 结果 |
|---|---|---|---|---|
| TC-D-01 | P0 | 载荷 6 条标记全命中 + 依赖门禁（19 项精确钉住）通过 | 通过 | ✅（判定为「缺标记 ⇒ 重编」，重编 678 s） |
| TC-D-02 | P0 | HAP 构建并 debug 签名通过，`pack.info` = `1900/0.19.0` | 通过 | ✅ 136,077,761 B；归档 85.7 MiB / `07c79770f41d…` |
| TC-D-03 | P0 | 安装 HAP 后启动，首屏渲染正常 | 通过 | ⬜ **本轮无鸿蒙测试环境** |
| TC-D-04 | P0 | 终端面板显示「平台不支持 PTY」类明确提示（`0001` 能力门） | 通过 | ⬜ |
| TC-D-05 | P0 | 沙箱根来自鸿蒙 roots（`0002`）、hdc 路径别名归一（`0003`） | 通过 | ⬜ |
| TC-D-06 | P0 | 目录选择器：主目录 Tab / 面包屑 / 权限流（`0005`） | 通过 | ⬜ |
| TC-D-07 | P0 | 文本对话 + 图片分析对话 | 通过 | ⬜ |
| TC-D-08 | P1 | 会话导出进程内回退（`0004`）、系统返回键（`0008`） | 通过 | ⬜ |

## E. Android 版（pi-desktop-android）

| ID | 级别 | 用例 | 预期 | 结果 |
|---|---|---|---|---|
| TC-E-01 | P0 | `assembleDebug` / `assembleRelease` 成功，`aapt` = `1900 / 0.19.0` | 通过 | ✅ 482,106,993 B / 473,074,747 B |
| TC-E-02 | P0 | 载荷归档内 `pi-web 0.19.0` / `next 16.4.6` / `pi-coding-agent 1.6.1` | 一致 | ✅ tar 直读断言 |
| TC-E-03 | P0 | `adb install -r` 覆盖安装（不卸载） | 成功 | ❌ `INSTALL_FAILED_USER_RESTRICTED: Install canceled by user`（MIUI「USB 安装」限制；`pm install`、`adb_install_need_confirm=0` 同样失败，设备端无任何弹窗） |
| TC-E-04 | P0 | 安装后启动，首屏渲染正常 | 通过 | ⬜ 被 TC-E-03 阻塞 |
| TC-E-05 | P0 | 版本徽标 `0.19.0` | 通过 | ⬜ 被 TC-E-03 阻塞 |
| TC-E-06 | P0 | 设置 DeepSeek API KEY | 通过 | ⬜ 被 TC-E-03 阻塞 |
| TC-E-07 | P0 | 创建沙箱工作区（`0002`/`0003`） | 通过 | ⬜ 被 TC-E-03 阻塞 |
| TC-E-08 | P0 | 文本对话「介绍一下你自己」 | 通过 | ⬜ 被 TC-E-03 阻塞 |
| TC-E-09 | P0 | 图片分析对话 | 通过 | ⬜ 被 TC-E-03 阻塞 |
| TC-E-10 | P0 | 终端：Android 有 PTY 支持（`globalThis.__piWebPty.ok` 为 true 时应可用） | 通过 | ⬜ 被 TC-E-03 阻塞 |
| TC-E-11 | P1 | 目录选择器全流程（`0005`）、系统返回键（`0008`） | 通过 | ⬜ 被 TC-E-03 阻塞 |
| TC-E-12 | P1 | **既有应用不受影响**：`org.fellow99.pi.PiDesktopAndroid`（0.15.0）仍在且数据完好 | 未删除 | ✅ 未执行任何 `uninstall`；设备上仍为 0.15.0 / `versionCode 1500`、`firstInstallTime 2026-11-04` |

## F. 官网（pi-desktop-website）

| ID | 级别 | 用例 | 预期 | 结果 |
|---|---|---|---|---|
| TC-F-01 | P0 | 中英文页面全部版本号显示 `0.19.0`、`versionCode 1900` | 通过 | ✅ `grep 0.18.0` = 0 |
| TC-F-02 | P0 | 能力矩阵补丁数（Desktop 1 / Android 6 / HOS 3 / Ark 6）与实际一致 | 一致 | ✅ 注：列序为 Desktop·Android·HOS·Ark |
| TC-F-03 | P1 | 页面无破版、链接可点 | 通过 | ⬜ 需浏览器目视 |

## G. 回归（跨端）

| ID | 级别 | 用例 | 预期 | 结果 |
|---|---|---|---|---|
| TC-G-01 | P0 | 四端均能加载同一套 Pi Web UI（无 CORS / 无自定义协议 / 无登录页） | 通过 | ✅ 桌面 / HOS 两次服务端冒烟均确认同源加载；真机部分见 B–E |
| TC-G-02 | P1 | `#1827` 页面响应带 `frame-ancestors 'self'`（未设 `PI_WEB_FRAME_ANCESTORS`），顶层导航不受影响 | 通过 | ✅ 代码核对 + 冒烟 `GET /` 200 且渲染标记命中 |
| TC-G-03 | P1 | `#1828` 设 `PI_WEB_SKIP_VERSION_CHECK=1` 时 `/api/app-update` 返回 `updateAvailable:false` | 通过 | ✅ 桌面冒烟覆盖；源码核对短路条件仍在 |
| TC-G-04 | P1 | **0.19.0 新增**：`/api/notices` / `/api/queue-modes` / `/api/http-proxy` / `/api/file-index` 路由存在且不 500 | 通过 | ✅ 构建产物路由表包含 4 条新路由；`/api/cache-miss-notices` 已消失 |
| TC-G-05 | P1 | 模型列表可加载、可发起真实对话 | 通过 | ✅ 桌面打包态已完成真实对话（TC-B-05/07）；安卓与鸿蒙未上机 |
| TC-G-06 | P2 | 会话列表、切换、删除正常 | 通过 | ⬜ 需真机 |

## H. 阻塞项与本轮结论

| # | 阻塞项 | 解除方式 |
|---|---|---|
| 1 | Android 装机：MIUI「USB 安装」限制 | 设备「设置 → 更多设置 → 开发者选项 → USB 安装」开启（需登录小米账号），或把 APK 拷到设备用文件管理器安装 |
| 2 | Windows 打包态自测：壳单实例锁 | 换一台未运行 Pi Desktop 的 Windows 机器，或先退出宿主进程后再跑 `probe-renderer` / e2e |
| 3 | 鸿蒙两版真机 | 接入鸿蒙测试设备（`hdc list targets` 可见且允许侧载） |

> **本轮结论**：构建与版本口径 **A 段 12/12 通过**；功能与端侧能力 **B–G 段未在任何真机上回归**（原因见 §H），
> 三项阻塞**均属环境/设备侧**，不涉及本轮代码改动。
