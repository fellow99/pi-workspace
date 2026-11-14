# 013 — 品牌化（branding）跨工程技术方案

> **Module:** 013-branding ｜ **Status:** Implemented（Desktop / HOS / Android）；Partial（Ark） ｜ **Last Updated:** 2026-10-31
> **对应规格:** [`spec.md`](./spec.md)（FR-013-001 … FR-013-023）
> **父/子分工:** 本文写四版共同的物料/替换/校验策略与各版落点；具体脚本与资源清单见子工程 `013-branding/plan.md`。

---

## 1. 技术上下文

### 1.1 物料

| 文件 | 规格 | 用途 |
|---|---|---|
| `logo.svg` | 含 `<svg>` / `viewBox` | 页面 Logo（矢量） |
| `logo.png` | 200×200，含透明像素 | 托盘模板、透明位图、adaptive 前景 |
| `logo-1024-with-bg.png` | 1024×1024，alpha 全 255 | 应用图标、PWA 512、legacy 图标 |
| `logo.ico` / `logo.icns` | Windows / macOS 图标 | 桌面安装器/应用图标 |

### 1.2 工具

| 工程 | 工具 | 说明 |
|---|---|---|
| Desktop | Node + `brand-assets.mjs`（ffmpeg/Pillow 可选） | 文案替换、web 图标、`resources/*` |
| HOS | Node + Pillow / ffmpeg | `AppScope` 图标、overlay 字形、pi-dist 文案 |
| Ark | Node + ffmpeg 7.1 | 图标生成；无 SVG 光栅化器 |
| Android | Node + Python/Pillow | 五密度/adaptive 图标、载荷文案/图标 |

---

## 2. 宪法合规检查

| 条款 | 合规 | 说明 |
|---|---|---|
| 第一条 编号 | ✅ | 统一 `013-branding`；历史迁移 |
| 第二条 分工 | ✅ | 父写口径，子写实现 |
| 第三条 文档集合 | ✅ | 父只写 spec/plan |
| 第四条 源码即真理 | ✅ | 以脚本/资源为准 |
| 第五条 上游零改动 | ✅ | 只在副本上替换 |
| 第六条 架构 | ✅ | 品牌化不改数据面 |
| 第七条 安全 | ✅ | 不改签名/凭据 |
| 第八条 版本 | ✅ | 不改变版本号；包名按 R2 决策变更（见 spec §R2） |

---

## 3. 共同策略

### 3.1 物料校验

1. `logo.svg`：文本包含 `<svg` 与 `viewBox`。
2. `logo.png`：200×200，且存在 alpha < 255 的像素。
3. `logo-1024-with-bg.png`：1024×1024，alpha 全 255（minAlpha=255）。
4. 缺失/不可解析/尺寸不符 ⇒ 硬失败。

### 3.2 文案替换

- 只替换**带空格、首字母大写**的 `Pi Web` → `Pi Desktop`；大小写敏感天然避开 `pi-web`。
- 按内容判定文本（前 8 KB 无 NUL，跳过 `.map`），覆盖 `.rsc` 等无扩展名文件。
- 排除 `node_modules/`（第三方技术面）。
- 替换后终检：0 处 `Pi Web`，技术标识符计数不变。

### 3.3 图标映射

| 目标 | 物料 | 规则 |
|---|---|---|
| 应用图标 | `logo-1024-with-bg.png`（或 `logo.png` + 白底） | 1024²、rgb24、不透明；Ark 用后者 |
| 启动图标 | 同源或 `logo.png` | 按平台尺寸；允许透明 |
| 窗口/托盘 | `logo.png` / `logo-200-with-bg.png` | 200²；macOS 模板需透明 |
| PWA 512 | `logo-1024-with-bg.png` | 512²、不透明 |
| PWA 192 / apple-touch | `logo.png` | 192/180、rgba |
| favicon | `logo.png` | PNG 字节，不得 JPEG |
| 页面 Logo | `logo.svg` | 逐字节复制，`<img src>` 指向 `/icons/logo.svg` |

### 3.4 运行时开关

- `PI_WEB_APP_NAME=Pi Desktop`。
- `PI_WEB_SKIP_VERSION_CHECK=1`（保留 `currentVersion` 显示）。
- 桌面另有自己的半自动更新检查（`113-release-update`），与上游检查分离。

### 3.5 终检

- 独立命令（`--check`）可只读运行。
- 输出：残留文件列表、技术标识符计数、图标尺寸/透明度。
- 失败退出非 0，CI 阻断。

---

## 4. 各版落点

### 4.1 Desktop

- `scripts/brand-assets.mjs`：`pi-dist` 文案/Logo/web 图标/favicon；`resources/icon.{ico,icns,png}`、`tray*.png`。
- `resources/brand/`：仓内镜像（CI 无父工程时使用，WARN）。
- 运行时：`PI_WEB_APP_NAME` / `PI_WEB_SKIP_VERSION_CHECK`。
- 终检：`pi-dist` 0 残留；`index.html` / `login.html` 含 `Pi Desktop`；技术标识符计数不变。

### 4.2 HOS

- `scripts/brand-assets.mjs`：`AppScope` 的 `app_icon` / `icon` / `startIcon`，`runtime-overlays` 的 `electron_white.png`。
- `collect-pi` / `prepare-next` 内对 pi-dist 文案/Logo/web 图标做替换。
- 运行时开关同 Desktop。
- 真机：图标槽位、启动页、首屏、PWA 名。

### 4.3 Ark

- `scripts/brand-assets.mjs`：`app_icon`（1024² rgb24）、`icon`（1024² rgb24）、`startIcon`（512² rgba）；ffmpeg，`--check` 幂等。
- `assemble-payload.mjs#applyWebBranding`：pi-web 副本文案替换（内容判定，31 文件/1011 处）。
- `payload/env.js`：`PI_WEB_APP_NAME=Pi Desktop` / `PI_WEB_SKIP_VERSION_CHECK=1`。
- **未闭合**：页面 Logo（无 SVG 光栅化器）、应用名 `Pi Desktop Ark`、首屏标记。
- 建议方案：在副本中把 `<img src>` 指向 `/icons/logo.svg` 并复制 SVG；产品定名后统一 `string.json`。

### 4.4 Android

- `scripts/generate-icons.mjs`：五密度 legacy/round/adaptive 前景 + 背景色 + `mipmap-anydpi-v26`。
- `scripts/brand-assets.mjs`：载荷文案/Logo/favicon/PWA。
- `res/values/strings.xml`：`app_name=Pi Desktop`。
- `payload/env.js`：品牌环境变量。
- 终检：`--check`；0 残留；技术标识符计数不变。

---

## 5. 错误处理

| 场景 | 处理 |
|---|---|
| 物料缺失 | 打印路径与获取方式；exit 1 |
| 尺寸不符 | 打印期望/实际；exit 1 |
| 透明度不符 | 打印 minAlpha；exit 1 |
| 图片工具缺失 | 提示安装 Pillow / ffmpeg；exit 1 |
| 残留 `Pi Web` | 列出文件；exit 1 |
| 技术标识符计数变化 | 打印差异；exit 1 |

---

## 6. 测试考量

- 正向：四版生成/替换后检查图标、文案、Logo。
- 负向：物料缺失、尺寸/透明度不符、残留注入。
- 幂等：连续两次生成字节一致。
- 回归：启动、会话、对话、工具、文件上传、Git。
- 真机：四版首屏与应用列表图标。

---

## 7. 文件清单

| 文件 | 工程 | 作用 |
|---|---|---|
| `scripts/brand-assets.mjs` | Desktop / HOS / Ark | 图标/文案品牌化 |
| `scripts/collect-pi.mjs` / `prepare-next.mjs` | Desktop / HOS | 调用品牌化 |
| `scripts/generate-icons.mjs` | Android | 五密度/adaptive 图标 |
| `scripts/assemble-payload.mjs` | Ark / Android | 载荷品牌化 + 组装 |
| `resources/` / `AppScope` / `res/mipmap-*` | 各版 | 图标落点 |
| `payload/env.js` | Ark / Android | 运行时品牌开关 |
| `assets/` | 父工程 | 唯一物料来源 |

---

## 8. 与规格交叉引用

| FR | 落点 |
|---|---|
| FR-013-001–005 | 文案/运行时开关/技术标识符 |
| FR-013-006–011 | 物料校验 + 图标映射 |
| FR-013-012–015 | 四版落点 |
| FR-013-016–018 | Ark 未闭合项 |
| FR-013-019–023 | 终检 / 回归 / 不变版本 |
