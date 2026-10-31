# ARCHITECTURE.md — Pi Desktop Workspace 整体架构

> 描述四个版别共同的数据面、进程模型、分层、鉴权差异、载荷/补丁策略与构建发布拓扑。
> 各版实现细节见子工程 `specs/ARCHITECTURE.md`；本文件只写**跨版共同结论**与**版间差异**。

---

## 1. 一句话架构

**四个版别都是「进程内 Node 托管 pi-web（SPA + `/api/*` + agent）+ 同源 Web 渲染 + 薄壳」**：
壳只负责启动运行时、装配环境、创建窗口/WebView、注入平台能力与生命周期；业务 100% 在上游 `pi` / `pi-web` 中。

```
┌──────────────────────────── 应用进程（每版不同） ────────────────────────────┐
│                                                                              │
│  ┌────────────────────┐      same-origin HTTP + SSE      ┌────────────────┐  │
│  │ 渲染层              │ ───────────────────────────────► │ pi-web (Next)  │  │
│  │ Chromium / ArkWeb   │ ◄─────────────────────────────── │ SPA + /api/*   │  │
│  │ 系统 WebView        │                                  │                │  │
│  └────────────────────┘                                  │  ┌──────────┐  │  │
│           ▲                                              │  │ pi agent │  │  │
│           │ 平台桥 / 外链 / 选择器 / 通知 / 日志            │  │ + tools  │  │  │
│  ┌────────┴───────────┐                                  │  └──────────┘  │  │
│  │ 壳（Electron/ArkTS/ │                                  └────────────────┘  │
│  │ Kotlin）            │                                                      │
│  └────────────────────┘                                                      │
└──────────────────────────────────────────────────────────────────────────────┘
        │ 文件系统 / 网络 / 进程 / 通知 / 系统浏览器 / 权限
        ▼
   操作系统 / 设备
```

---

## 2. 四种进程模型

| 版别 | 应用进程 | Node 宿主位置 | 渲染 | 启动方式 |
|---|---|---|---|---|
| Desktop | Electron 主进程 | Electron 主进程内 `next start-server` | Chromium `BrowserWindow` | `app.whenReady` → 宿主 → 窗口 |
| HOS | ArkTS 应用进程 + Electron 运行时 | Electron 主进程内 `require` Next | Chromium（包内） | EntryAbility → Electron runtime → 宿主 |
| Ark | ArkTS 应用进程 | App 进程内 `dlopen` + `node::Start` | 系统 ArkWeb | EntryAbility → 投放/启动 Node → ArkWeb |
| Android | Kotlin 应用进程 | App 进程内 `dlopen` + `node::Start` | 系统 WebView | MainActivity → 解包/启动 Node → WebView |

共同点：

- **无子进程 Node**（除构建期脚本）；无孤儿进程、无进程树清理。
- **就绪判定靠 HTTP 轮询**（`/api/web-auth` 等），不依赖启动回调。
- **随机端口 + 单实例**；端口回报给壳。
- **宿主崩溃 = 应用崩溃**（进程内宿主），因此必须有 fail-visible 错误页 / 日志 / 重启提示。

---

## 3. 分层

| 层 | 职责 | 落点 |
|---|---|---|
| 渲染层 | 展示 pi-web SPA；不直接访问 Node | Chromium / ArkWeb / WebView |
| 壳层 | 启动、窗口、导航围栏、平台桥、生命周期、错误页 | Electron main / ArkTS / Kotlin |
| 数据面 | SPA、`/api/*`、SSE、agent 会话 | 上游 `pi-web`（进程内） |
| 业务/工具 | agent、模型、工具、MCP、扩展 | 上游 `pi` + 载荷扩展 |
| 运行时 | Node 引擎、libnode、JNI/NAPI 桥 | Electron / node-bridge / node-runtime |
| 系统 | 文件、网络、通知、权限、浏览器 | OS / 设备 |

**依赖方向**：渲染层 →（HTTP/SSE）→ 数据面 → 业务/工具 → 运行时 → 系统。
壳层只通过 HTTP 与数据面交互；**数据面不过 IPC/JavaScriptInterface**。

---

## 4. 数据流

### 4.1 启动

```
壳启动
  → 单实例锁 / 单实例声明
  → 环境装配（数据目录、PI_CODING_AGENT_DIR、PI_WEB_*、代理、证书）
  → 载荷就位（解压 / 只读 resfile / 直读）
  → 启动 Node + pi-web（随机端口）
  → HTTP 轮询就绪（/api/web-auth 或等价）
  → 创建窗口/WebView，加载 http://127.0.0.1:<port>/
  → 注入鉴权（桌面/HOS）或 loopback 免密（Ark/Android）
  → 首屏渲染完成
```

失败时：阶段文本 + 错误页 + 日志尾 + 诊断入口（各版实现不同）。

### 4.2 对话

```
用户输入
  → POST /api/agent/<id>（同源）
  → pi agent 创建/恢复 AgentSession
  → 模型流式返回（SSE）
  → UI 增量渲染
  → 会话 JSONL 落盘（<agentDir>/sessions/）
  → 工具调用（read/write/edit/ls/find/grep/bash/git/...）
```

### 4.3 工具与扩展

- 桌面 / HOS：系统 bash/git 可用；HOS 用纯 JS 同名顶替 `bash`/`find`/`grep`。
- Ark / Android：无系统 git/fd/rg，使用载荷内纯 JS 扩展（`harmony-git` / `android-git`、`*-search`、`*-exec`）。
- 扩展通过 `settings.json` 的 `defaultTools` 与 `extensions/` 目录装载；不 fork 上游。

---

## 5. 鉴权矩阵

| 版别 | 监听 | 密码 | 注入 | 原因 |
|---|---|---|---|---|
| Desktop | `127.0.0.1` 随机端口 | 随机强密码 | Basic（`/api/*` + SSE）+ 派生会话 Cookie（导航） | 同机其他进程可访问 loopback |
| HOS | `0.0.0.0` 随机端口 | 随机强密码 | 会话 Cookie（HMAC） | Electron 渲染进程 loopback 隔离 |
| Ark | `127.0.0.1` 随机端口 | 无（loopback 免密） | 无需注入 | 应用进程自身，ArkWeb 同进程 |
| Android | `127.0.0.1` 随机端口 | 无（loopback 免密） | 无需注入 | 应用进程自身，WebView 同进程 |

纪律：

- 密码/凭据不入日志；会话 Cookie 只在本机；不把 `PI_WEB_PASSWORD` 写入磁盘明文。
- 桌面/HOS 的凭据注入是**必需项**，不是可选加固；缺失会落到 `/login`。
- 移动端 loopback 免密的前提是「服务只绑 loopback + 同进程」；若改成局域网绑定必须恢复密码。

---

## 6. 载荷与补丁

### 6.1 载荷

| 版别 | 上游构建 | 组装 | 投放 | 首启 |
|---|---|---|---|---|
| Desktop | `prepare-next` 构建 pi-web | `collect-pi` → `pi-dist.tar.gz` + marker | `extraResource` | 解压到 `<userData>/pi-dist`，SHA-256 幂等 |
| HOS | `prepare-next`（Node 22.17） | `collect-pi` → tar + marker | `resfile` | 解压 + marker 幂等 |
| Ark | 预构建产物 | `assemble-payload` → `pi-web.tar.gz` | `resfile` | 运行时解包到 `<filesDir>` |
| Android | 预构建产物 | `assemble-payload` → `pi-web.tar.gz` | `resfile` | `PayloadArchive` 解包到 `<filesDir>` |

共同纪律：

- 从上游生产产物原样组装，不 fork、不改上游源码。
- 裁剪只删不用项，**先删后验**；体积账记录在案。
- 内容级幂等（SHA-256 / marker），不是「目录存在即跳过」。
- 品牌化只在**本工程副本**上做。

### 6.2 补丁

| 版别 | 补丁数 | 目录 | 说明 |
|---|---|---|---|
| Desktop | 0 | 无 | 上游机制足够 |
| HOS | 2 | `patches/v0.12.1/` | 选择器 roots、会话导出进程内回退 |
| Ark | 6 | `patches/v0.12.1/` | 终端 unsupported、roots、hdc 别名、导出守卫、选择器契约/权限流 |
| Android | 6 | `patches/v0.12.1/` | 与 ark 同构（adb 别名） |

补丁纪律：

1. 按被 patch 的上游 tag 建目录，不跨 tag 复用。
2. 幂等：应用前 `--reverse --check`，已应用则跳过。
3. 可逆：应用 → 反向 → 逐文件 SHA-256 还原断言。
4. 冲突即终止，禁止 `--3way`。
5. 补丁数、语义断言、真机证据写回子工程 `specs/` 与 `patches/README.md`。

---

## 7. 构建与发布拓扑

```
上游 tag（pi v1.2.0 / pi-web v0.12.1）
        │
        ├── desktop: npm ci → prepare-next → collect-pi → smoke → Forge package/make → GitHub Release
        ├── hos:     collect-runtime → prepare-next → smoke-node22 → collect-pi → collect-app → hvigor → 签名 HAP
        ├── ark:     assemble-payload → hvigor → 签名 HAP
        └── android: assemble-payload → verify-runtime → gradle → 签名 APK → GitHub Release
```

- **桌面 / Android**：tag 触发 GitHub Actions，产物上传 Release。
- **鸿蒙两版**：当前本地构建 + 签名 + 真机安装（无 GitHub Actions 构建工作流，故父工程 `012-building-workflow` 不覆盖）。
- **品牌物料**：父工程 `assets/` 是唯一来源；CI 通过 sparse checkout 获取。
- **版本**：应用版本与上游版本分离；tag 只用于发布标识（Android 明确不注入 `versionName`）。

---

## 8. 关键架构决策

| # | 决策 | 理由 | 代价 / 风险 |
|---|---|---|---|
| A1 | 进程内 Node，无子进程 | 无孤儿、就绪确定、少一层 IPC | 宿主崩溃 = 应用崩溃；需崩溃兜底 |
| A2 | 同源 HTTP + SSE，无 CORS/自定义协议 | 零跨域、可用 curl 取证、复用上游 | 受网络栈/WebView 限制 |
| A3 | 上游零 fork，补丁按 tag 归档 | 升级成本低、可审计 | 补丁需幂等/可逆/真机验证 |
| A4 | 薄壳：只做装配与平台能力 | 业务不漂移、四版一致 | 平台能力需逐版补齐 |
| A5 | 失败可见（阶段文本/错误页/日志/诊断） | 可定位、可证伪 | 实现成本；不能静默降级 |
| A6 | 版本与运行时锁定（tag + SHA-256） | 可复现 | 升级流程重 |
| A7 | 四版共享 `pi-web` 载荷 | UI/能力一致 | 平台差异靠扩展/补丁/桥 |
| A8 | 品牌物料单一来源 | 一致、可复现 | 物料缺失需硬失败 |
| A9 | 移动端 loopback 免密 | 同进程、无跨进程暴露 | 若改绑定必须恢复密码 |
| A10 | 鸿蒙 Ark 固定 `compat` | 避免 JIT 误判导致整机不可用 | code-mode/WASM 不可用 |

---

## 9. 责任边界

| 责任 | 父工程 `specs/` | 子工程 `specs/` | 子工程 `docs/` |
|---|---|---|---|
| 整体规划 / 四版共同需求 | ✅ | 引用 | 引用 |
| 模块编号 / 文档分工 | ✅ 宪法 | 遵循 | — |
| 版本矩阵 / 构建品牌总规范 | ✅ `011`/`012`/`013` | 本工程实现 | 操作手册 |
| 具体技术实现 / 模块设计 | ❌ | ✅ `spec.md` + `plan.md` | 论证与证据 |
| 真机记录 / 能力清单 | 汇总口径 | 引用 | ✅ |
| API / 数据模型 / 测试用例 | 刻意不写 | ✅（按需） | 证据 |

---

## 10. 部署与运行拓扑

| 版别 | 运行位置 | 数据目录 | 网络 | 更新 |
|---|---|---|---|---|
| Desktop | 用户机器（Win/mac/Linux） | `userData` / 可选共享 `~/.pi/agent` | loopback + 出网 | 半自动检查 → 引导下载 |
| HOS | 鸿蒙 PC / 平板 | 应用沙箱 | 局域网绑定 + 出网 | 无（禁用上游检查） |
| Ark | 鸿蒙 2in1 / 平板 / 手机 | `<filesDir>/pi-agent` | loopback + 出网 | 无 |
| Android | Android 手机 / 平板 | `<filesDir>/pi-agent` | loopback + 出网 | 无（GitHub Release 手动） |

共同：无服务端部署、无账号体系、无遥测服务；所有数据在设备本地（模型 API 出网除外）。
