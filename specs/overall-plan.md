# overall-plan.md — Pi Desktop Workspace 整体技术方案

> **层级：** 父工程 · 技术方案
> **对应规格：** [overall-spec.md](./overall-spec.md)（`FR-000-*` / `NFR-000-*`）
> **版本：** v0.12.1 基线 ｜ **最后更新：** 2026-10-31
> **说明：** 本文写“四版共同怎么做”；单版实现细节见各子工程 `overall-plan.md` 与模块 `plan.md`。

---

## 1. 技术上下文

### 1.1 运行时环境

| 形态 | 运行时 | Node | 渲染 | 产物 |
|---|---|---|---|---|
| Desktop | Electron 43.7.9 | 24.21.0 | Chromium 150 | Win/mac/Linux 安装包 |
| HOS | Electron-on-HarmonyOS 37.2.3 | 22.17.0 | Chromium 138 | HAP |
| Ark | ArkTS + ArkWeb + node-bridge 0.1.0 | 24.2.0 | 系统 ArkWeb | HAP |
| Android | Kotlin + WebView + node-bridge 0.2.0 | 24.22.1 | 系统 WebView | APK |

### 1.2 关键依赖

- 上游：`pi` v1.2.0、`pi-web` v0.12.1（Next.js 16.3.8）。
- 运行时：ohpm / Maven 的 node-bridge + node-runtime。
- 构建：npm / hvigor / Gradle / GitHub Actions。
- 物料：父工程 `assets/`。

### 1.3 仓库布局约束

四个子工程构建脚本按“父目录的兄弟目录”解析上游与物料：

```
pi-workspace/
├── pi/ pi-web/ assets/ harmonypc-electron/
├── pi-desktop/ pi-desktop-hos/ pi-desktop-ark/ pi-desktop-android/
```

因此**不能单独 clone 某个壳**，也不能改目录名；CI 必须把上游 clone 到同级。

---

## 2. 宪法合规检查

| 宪法条款 | 合规 | 落点 |
|---|---|---|
| 第一条 编号规则 | ✅ | 父/子 `specs/` 统一 `011`/`012`/`013`；1XX/2XX 对齐 |
| 第二条 文档分工 | ✅ | 父工程整体、子工程实现；`docs/` 存论证 |
| 第三条 文档集合 | ✅ | 父工程 7 份 + 3 模块，无 API/data-model/test-cases |
| 第四条 源码即真理 | ✅ | 各文档带路径/证据；迁移后回写 |
| 第五条 上游零改动 | ✅ | 补丁按 tag、幂等可逆；不 fork |
| 第六条 架构原则 | ✅ | 进程内宿主、同源 HTTP、薄壳、随机端口、失败可见 |
| 第七条 安全隐私 | ✅ | 最小权限、凭据不入库、导航围栏 |
| 第八条 版本构建 | ✅ | 上游钉住、运行时锁、版本分离、门禁 |
| 第九条 提交纪律 | ✅ | Conventional Commits |
| 第十条 评审纪律 | ✅ | 请求/接收评审流程 |
| 第十一条 specs-as-built | ✅ | spec + plan 成对、FR 回引 |
| 第十二条 变更治理 | ✅ | 迁移对照表 + 0 断链验证 |

---

## 3. 实现策略总览

### 3.1 共同模式

```
上游 tag（pi / pi-web）
  → 构建 pi-web 生产产物
  → 组装载荷（+ 补丁 + 品牌化 + 裁剪）
  → 壳启动进程内 Node + pi-web
  → 同源加载 UI
  → 平台能力桥
  → 门禁 + 真机/打包态验证
```

### 3.2 分版策略

#### Desktop

1. Electron 主进程 `startServer` 启动 Next；随机端口 + 随机密码。
2. `collect-pi` 组装 `pi-dist.tar.gz`；首启解压到 `userData`，SHA-256 幂等。
3. `BrowserWindow` 同源加载；导航围栏 + 权限白名单 + `window.open` 外链。
4. 托盘、通知、生命周期、更新检查。
5. Forge 打包 + Fuses；CI 三平台；Release 半自动更新。

#### HOS（Electron on HarmonyOS）

1. `collect-runtime` 从 `../harmony*` 取 v37 运行时、SO、Chromium 资源；prune + overlay。
2. `prepare-next` 用 Node 22.17 构建；`smoke-node22` 门禁。
3. `collect-pi` 组装 tar + marker；`collect-app` 投放到 `resfile`。
4. Electron 主进程内 `require` Next；`0.0.0.0` 随机端口 + 密码 + Cookie 注入。
5. 纯 JS `bash`/`find`/`grep` 顶替；选择器补丁；`013` 品牌化。
6. hvigor 构建 + 签名 + 真机；`011` 版本收敛。

#### Ark（HarmonyOS）

1. `assemble-payload` 组装 `pi-web.tar.gz`（含 6 补丁、扩展、jitless、config）。
2. `@fellow99/node-bridge` 在应用进程内启动 Node；随机端口 + loopback 免密。
3. ArkWeb 加载；三态 UI + 错误页 + 返回键；`HostCapabilities` 9 项。
4. `compat`（`--jitless`）形态；纯 JS 工具与 Git 扩展。
5. hvigor 构建 + 签名 + 真机；`011`/`013` 文档。

#### Android

1. `assemble-payload` + `crop-rules` 组装载荷（6 补丁、8 扩展、jitless、config）。
2. `verify-runtime` 对 AAR 做 SHA-256 + ELF 六项门禁。
3. `PiDesktopRuntime` `dlopen` + `node::Start`；随机端口 + loopback 免密。
4. WebView 加载；`HostCapabilities` 10 项（含 keepAlive/SAF）；前台服务。
5. `generate-icons` + `brand-assets` 品牌化；`compliance-check`。
6. Gradle 构建 + 签名 + GitHub Release；`011`/`012`/`013` 文档。

---

## 4. 跨切面关注点

### 4.1 版本与上游钉住

- 上游 tag + commit SHA 写入 CI env 与 `upstream.lock.json`；重打 tag 即失败。
- 运行时版本写入 ohpm / Gradle / `runtime.lock.json`；SHA-256 + ELF 门禁。
- 应用版本与上游版本分离；tag 不注入 `versionName`/`versionCode`（Android 明确）。
- 升级流程：`011` spec → 改锁文件 → 重跑门禁 → 真机 → 回写文档 → tag。

### 4.2 补丁纪律

- 目录：`patches/<pi-web-tag>/`。
- 应用：反向检查 → 正向检查 → 应用 → 反向 → SHA-256 还原。
- 冲突：终止，不 `--3way`。
- 证据：补丁数、语义断言、真机结果写回 `specs` 与 `patches/README.md`。

### 4.3 鉴权

| 版别 | 方案 | 关键点 |
|---|---|---|
| Desktop | 随机密码 + Basic + Cookie | 覆盖 `/api/*` 与 SSE；导航 Cookie |
| HOS | 随机密码 + HMAC Cookie | 局域网绑定；Cookie 覆盖所有请求 |
| Ark/Android | loopback 免密 | 只绑 127.0.0.1；同进程 |

### 4.4 失败可见

- 启动阶段文本 / 进度 / 错误页 / 日志 / 诊断入口。
- 移动端崩溃为终态（Node 每进程一次）：明确提示重启，不假装可恢复。
- 构建失败：非零退出 + 明确修法；不产出半成品。

### 4.5 安全

- 最小权限；声明面有调用点；运行时申请。
- 导航围栏 + 外链分流；生产无调试入口。
- 凭据不入库；日志脱敏；提交前扫描。
- 签名材料仅本地/CI secrets；keystore 不入库。

### 4.6 质量门禁

| 工程 | 门禁 |
|---|---|
| Desktop | `typecheck` + `smoke-desktop` 四项 + 原生依赖完整性 |
| HOS | `smoke-node22` 0 FAIL + collect-runtime 守卫 + 补丁可逆 |
| Ark | 载荷 tar 校验 + 补丁可逆 + HAP 编译 + 真机 |
| Android | `verify-runtime` + `compliance-check` + Gradle + 真机/CI |

### 4.7 文档

- 父工程：整体/公共模块；子工程：实现/独立需求。
- `spec.md` 技术无关；`plan.md` 实现细节并回引 FR。
- 编号迁移后更新所有引用；断链即缺陷。

---

## 5. 构建与发布

### 5.1 桌面

```
npm ci
→ prepare:next（pi-web 构建 + 原生件裁剪 + 预览密钥轮换）
→ collect:pi（pi-dist.tar.gz + marker）
→ smoke（4 断言）
→ forge package/make（premake 自动串联）
→ GitHub Release（tag v*）
```

### 5.2 HOS

```
collect-runtime（v37 → electron/web_engine/SO/Chromium + prune + overlay）
→ prepare-next（Node 22.17 构建）
→ smoke-node22
→ collect-pi（tar + marker）
→ collect-app（resfile）
→ hvigor + 签名
→ 真机安装/验证
```

### 5.3 Ark

```
assemble-payload（pi-web 产物 + 6 补丁 + extensions + jitless + config + 品牌化）
→ hvigor 构建
→ 签名 HAP
→ 真机安装/验证
```

### 5.4 Android

```
assemble-payload（pi-web 产物 + 6 补丁 + 8 扩展 + jitless + config + 品牌化）
→ verify-runtime（AAR SHA-256 + ELF 六项）
→ gradle assembleRelease/Debug（+ compliance-check）
→ GitHub Release（tag v*）
```

### 5.5 产物与证据

- 产物：安装包 / HAP / APK + `.sha256`。
- 证据：workflow run id、Release URL、HAP/APK 哈希、真机设备与时间。
- 历史版本目录保留，不回改。

---

## 6. 测试与验证

### 6.1 分层

| 层 | 内容 | 落点 |
|---|---|---|
| L0 静态 | 类型检查、lint、配置一致性 | CI / 本地 |
| L1 单元/脚本 | 载荷组装、裁剪、补丁可逆、品牌生成 | 子工程 `scripts` + `test-cases.md` |
| L2 冒烟 | 服务启动、HTTP、SSE、agent 创建 | `smoke-desktop` / `smoke-node22` / devtest |
| L3 打包态 | 安装包启动、首屏、会话、对话 | 桌面 / 移动真机 |
| L4 真机矩阵 | 设备形态、权限、选择器、外链、通知 | 各子工程真机记录 |

### 6.2 判据

- HTTP 200 / SSE 建连 / 会话 JSONL / 截图 / marker / SHA-256。
- 不以“日志有一行”作为成功判据；以可读回证据为准。
- 模拟器只做轻量探针；移动端关键路径以真机为准。

### 6.3 回归

- 版本升级后重跑：载荷、补丁、品牌、引擎、启动、对话、工具、Git、导出。
- 品牌化后检查 0 残留与技术标识符计数。
- 权限变更后重跑权限矩阵。

---

## 7. 风险与处置

| # | 风险 | 处置 |
|---|---|---|
| R1 | 上游内部入口变化（Next `start-server`） | 门禁 + 回退 sidecar 预案（仅登记） |
| R2 | 原生模块 ABI/预构建不完整 | 构建期完整性门禁 + `@electron/rebuild` 预案 |
| R3 | 鸿蒙运行时/系统版本差异 | 固定运行时 + 真机矩阵 + 能力清单 |
| R4 | JIT/ACL 限制 | Ark 固定 `compat`；Android 默认 JIT |
| R5 | WebView 碎片 | polyfill + minSdk 约束 + 真机 |
| R6 | 无系统 git/fd/rg | 纯 JS 同名顶替扩展 |
| R7 | 签名/密钥泄露 | 不入库 + secrets + 扫描 |
| R8 | 品牌物料缺失 | 构建硬失败 |
| R9 | 应用名/Logo 未统一（Ark） | 013 登记 ⬜，产品定名 + SVG 方案 |
| R10 | 文档漂移 | 源码即真理 + 不一致项清单 + 回写 |
| R11 | 父/子工程指针不同步 | 子工程先提交，父工程再提指针 |
| R12 | 构建机环境差异 | Node 版本显式指定 + 锁文件 + CI |

---

## 8. 模块映射（父 → 子）

| 父工程 | Desktop | HOS | Ark | Android |
|---|---|---|---|---|
| `011` | `011-version-0.12.1` | `011-version-0.11.1` / `0.12.1` | `011-version-0.12.1` | `011-version-0.12.1` |
| `012` | `012-building-workflow` | —（鸿蒙暂无） | —（鸿蒙暂无） | `012-building-workflow` |
| `013` | `013-branding` | `013-branding` | `013-branding` | `013-branding` |
| 101 | `101-host` | `101-host` | `101-runtime-integration-*` | `101-runtime-integration-*` |
| 102 | `102-artifact-bootstrap` | 待生成 | `102-payload-pipeline-*` | `102-payload-pipeline-*` |
| 103 | `103-window` | 待生成 | `103-arkweb-shell-*` | `103-webview-shell-*` |
| 104 | `104-preload` | 待生成 | `104-host-capabilities-*` | `104-host-capabilities-*` |
| 105 | —（并入 101） | —（并入 101） | `105-auth-loopback-*` | `105-auth-loopback-*` |
| 106 | —（系统工具） | `106-harmony-js-tools` | `106-tools-extensions-*` | `106-tools-extensions-*` |
| 107 | —（并入构建/安全） | —（合规文档在 docs） | `107-compliance-*` | `107-compliance-*` |
| 108+ | `108-tray`…`113-release-update` | `108-signing-identity` | `108-jitless` | — |
| 2XX | `201-plugin-runtime` | `203` / `204` | `204` | —（并入 106） |

> `—` 表示该工程将职责并入其它模块或以文档/脚本承担；不强制为对齐而虚构模块。

---

## 9. 交付顺序（建议）

1. **父工程基线**：README / STRUCTURE / TECH / ARCHITECTURE / constitution / overall-spec / overall-plan。
2. **公共模块**：011 / 012 / 013 的 `spec.md` + `plan.md`。
3. **子工程编号迁移**：`git mv` + 引用更新 + 索引回写 + 0 断链。
4. **子工程公共模块补齐**：011 / 012（适用）/ 013（全部）。
5. **门禁与验证**：链接、计数、FR 一致性、构建脚本引用。
6. **评审**：按 `requesting-code-review` → `receiving-code-review` 闭环。
7. **提交**：子工程 `docs:` 提交；父工程更新 submodule 指针并提交。

---

## 10. 开放问题

| # | 问题 | 影响 | 下一步 |
|---|---|---|---|
| Q1 | Ark 应用名是否统一为 `Pi Desktop` | 品牌一致性 | 产品定名；更新 `string.json` + 013 |
| Q2 | Ark 页面 Logo 的 SVG 接入方案 | 品牌完整性 | 副本引用 `/icons/logo.svg` 或引入光栅化器 |
| Q3 | HOS 缺失的项目级 specs 文档 | 文档完整性 | ✅ 2026-10-31 已补齐（`pi-desktop-hos/specs/` 的 constitution/ARCHITECTURE/TECH/STRUCTURE/overall-*） |
| Q4 | Android 是否独立 204/205/206 插件模块 | 编号对齐 | 视后续拆分需要；当前并入 106 |
| Q5 | Desktop `201-plugin-runtime` 是否要做 | 可选功能 | 产品决策；未决策前保持 ⬜ |
| Q6 | 鸿蒙两版是否引入 GitHub Actions 构建 | 可复现/发布 | 当前本地构建；如引入再评估 012 适用性 |
