# 012 — 构建工作流（building-workflow）跨工程技术方案

> **Module:** 012-building-workflow ｜ **Status:** Implemented（Desktop / Android） ｜ **Last Updated:** 2026-10-31
> **对应规格:** [`spec.md`](./spec.md)（FR-012-001 … FR-012-028）
> **适用范围:** `pi-desktop`、`pi-desktop-android`；**鸿蒙两版不适用**。
> **父/子分工:** 本文写两条流水线的共同结构与差异；具体脚本/workflow 见子工程 `012-building-workflow/plan.md`。

---

## 1. 技术上下文

### 1.1 两条流水线

| 项 | Desktop | Android |
|---|---|---|
| 包管理 | npm（`package-lock.json`） | Gradle + pnpm（载荷构建） |
| 构建 | Electron Forge 7 + Vite 7 | AGP 8.10.1 + Gradle 8.12 |
| 运行时 | Electron 43.7.9（Node 24.21.0） | Node 24.22.1（AAR） |
| CI | GitHub Actions 三平台 | GitHub Actions `ubuntu-latest` |
| 产物 | Squirrel / zip / dmg / deb / rpm | 签名 APK / debug APK |
| 发布 | `release.yml`（tag） | `build-release.yml`（tag） |
| 门禁 | typecheck + smoke + 原生依赖 | verify-runtime + compliance |

### 1.2 共同外部依赖

- 上游：`pi` v1.2.0 / `pi-web` v0.12.1。
- 物料：父工程 `assets/`。
- 签名：本地 secrets / 环境变量。
- 运行时：Electron / Maven AAR。

---

## 2. 宪法合规检查

| 条款 | 合规 | 说明 |
|---|---|---|
| 第一条 编号 | ✅ | 统一 `012-building-workflow`；鸿蒙不建 |
| 第二条 分工 | ✅ | 父写要求，子写脚本 |
| 第三条 文档集合 | ✅ | 父只写 spec/plan |
| 第四条 源码即真理 | ✅ | 以 workflow/脚本为准 |
| 第五条 上游零改动 | ✅ | 补丁幂等可逆 |
| 第八条 版本构建 | ✅ | 钉住、门禁、可复现 |
| 第九条 提交 | ✅ | `build:` / `ci:` / `docs:` |

---

## 3. 共同流水线结构

```
清理
  → 安装依赖（锁文件）
  → 检出/校验上游（tag + SHA）
  → 组装载荷（+ 补丁 + 品牌化 + 裁剪）
  → 门禁（类型 / smoke / 引擎 / 合规）
  → 打包（Forge / Gradle）
  → 签名 / 校验和
  → 发布（tag 触发）
  → 证据归档
```

### 3.1 清理清单

- Desktop：`.vite/`、`dist/`、`out/`、`pi-dist*`、`.collect-pi-tmp/`、`node_modules/.cache`。
- Android：`app/build/`、`.gradle/`、`payload/pi-web/`、`dist/`、临时 AAR 缓存。
- 不删：源码、配置、`resources/`、`assets/`、`patches/`、`specs/`、`docs/`、脚本、锁文件。

### 3.2 上游校验

```bash
git clone --depth 1 --branch <tag> <repo> ../pi
test "$(git -C ../pi rev-parse HEAD)" = "<sha>"
```

- tag 与 SHA 双校验；失败即 `exit 1`。
- 上游目录不写入；补丁只在副本/工作树应用。

### 3.3 门禁顺序

1. 静态：类型检查 / lint。
2. 载荷：tar 内容、裁剪、补丁可逆、品牌 0 残留。
3. 运行时：引擎 SHA/ELF（Android）。
4. 冒烟：启动服务、HTTP、SSE、agent。
5. 合规：权限/隐私/敏感信息（Android）。
6. 打包：Forge / Gradle。

失败即停止；不跳过、不降级（除已登记的 debug 兜底）。

---

## 4. Desktop 流水线

### 4.1 本地

```bash
npm ci
npm run prepare:next
npm run collect:pi
npm run smoke
npm run package   # 或 npm run make
```

- `prepackage` / `premake` 自动串联 `prepare-next` + `collect-pi`。
- `brand` 脚本可由 `collect-pi` 调用或在构建前执行。
- 产物在 `out/`；`pi-dist.tar.gz` + `pi-dist.version` 进 `extraResource`。

### 4.2 CI

`ci.yml`：

- 三平台矩阵；clone `../pi` / `../pi-web` 并断言 SHA。
- 安装 ffmpeg（品牌位图）。
- `npm ci` → 构建 → smoke → package。
- 失败上传日志。

`release.yml`：

- tag `v*` 触发；校验 tag 与 `package.json.version`。
- 三平台构建 → 汇总资产 → GitHub Release。
- 不签名/不公证（当前决策）；不上传 `RELEASES`/`.nupkg` 的自动更新资产（半自动更新）。

### 4.3 产物

- Windows：Squirrel `.exe` / `.nupkg` / `RELEASES`（如需要）。
- macOS：`.dmg` / `.zip`。
- Linux：`.deb` / `.rpm` / `.zip`。
- 统一 `.sha256`（如发布脚本生成）。

---

## 5. Android 流水线

### 5.1 本地/CI

```bash
node scripts/assemble-payload.mjs
node scripts/verify-runtime.mjs --aar <node-runtime.aar>
./gradlew assembleRelease   # 或 assembleDebug
node scripts/compliance-check.mjs --apk <apk>
```

- `assemble-payload`：pi-web 产物 + 6 补丁 + 8 扩展 + jitless + config + 品牌化。
- `verify-runtime`：AAR SHA-256 + ELF 六项。
- Gradle：`versionName 0.1.0` / `versionCode 1`；签名来自 `keystore.properties`。
- `build-release.yml`：tag 触发 → 预取 AAR → 载荷 → 门禁 → APK → Release。

### 5.2 签名

- secrets：`ANDROID_KEYSTORE` / `KEYSTORE_PASSWORD` / `KEY_ALIAS` / `KEY_PASSWORD`。
- 全齐 → release 签名；全缺 → debug 兜底并标注；部分 → `::error::` 失败。
- 本地 `keystore.properties` gitignore；模板 `keystore.properties.sample`。

### 5.3 产物

- `pi-desktop-android-<name>.apk` + `.sha256`。
- Release 说明含上游 tag、Node 版本、bridge 版本、补丁数、run id。

---

## 6. 错误处理

| 场景 | 处理 |
|---|---|
| 依赖安装失败 | 重试/镜像；仍失败停止 |
| 上游 SHA 不符 | 停止；核对 tag |
| 补丁冲突 | 停止；不 `--3way` |
| 门禁失败 | 停止；输出失败项与修法 |
| 签名缺失/部分 | 全缺 debug 兜底；部分失败 |
| 发布失败 | 保留产物；重跑 workflow；不重复上传 |
| 平台差异（bash 3.2 等） | 显式兼容（变量定界、避免 GNU 特性） |

---

## 7. 测试考量

- 干净构建复现（不同机器/不同时间）。
- 门禁负向：错误 SHA、篡改 AAR、品牌残留、空壳 prebuilds。
- 产物安装/启动/对话。
- Release 附件与哈希。
- 失败日志可读性。

---

## 8. 文件清单

| 文件 | 工程 | 作用 |
|---|---|---|
| `package.json` / `package-lock.json` | Desktop | 依赖与脚本 |
| `forge.config.ts` | Desktop | Forge / Fuses / makers |
| `.github/workflows/ci.yml` / `release.yml` | Desktop | CI / Release |
| `scripts/prepare-next.mjs` / `collect-pi.mjs` / `smoke-desktop.mjs` | Desktop | 构建 / 组装 / 门禁 |
| `.github/workflows/build-release.yml` | Android | CI / Release |
| `scripts/assemble-payload.mjs` / `verify-runtime.mjs` / `compliance-check.mjs` | Android | 载荷 / 引擎 / 合规 |
| `app/build.gradle.kts` / `gradle/libs.versions.toml` | Android | 构建配置 |
| `runtime.lock.json` | Android | 引擎锁定 |

---

## 9. 与规格交叉引用

| FR | 落点 |
|---|---|
| FR-012-001–005 | 清理 + 锁文件 + 脚本 |
| FR-012-006–009 | 上游校验 + 运行时锁 + 补丁 |
| FR-012-010–014 | 门禁脚本 |
| FR-012-015–019 | 打包 / 产物 / 哈希 |
| FR-012-020–025 | CI / Release / secrets |
| FR-012-026–028 | 证据 / 真机 / 历史 |
