# 011 — 版本更新：0.12.1（version）跨工程技术方案

> **Module:** 011-version-0.12.1 ｜ **Status:** Implemented ｜ **Last Updated:** 2026-10-31
> **对应规格:** [`spec.md`](./spec.md)（FR-011-001 … FR-011-023）
> **父/子分工:** 本文写跨工程升级步骤与验证矩阵；单版命令、补丁与真机证据见子工程 `011-version-0.12.1/plan.md`。

---

## 1. 技术上下文

### 1.1 版本来源

| 版本 | 权威来源 | 镜像/锁定 |
|---|---|---|
| `pi` v1.2.0 | 上游 git tag | `pi-desktop/upstream.lock.json`（生成物）、CI env |
| `pi-web` v0.12.1 | 上游 git tag | 同上；各版构建脚本 `PI_WEB_TAG` |
| Electron 43.7.9 | `pi-desktop/package.json` | package-lock |
| Electron-on-HarmonyOS 37.2.3 | `harmony*` 运行时归档 | `pi-desktop-hos` 文档 |
| Node 24.2.0 / bridge 0.1.0 | `pi-desktop-ark/oh-package.json5` | `oh-package-lock.json5` |
| Node 24.22.1 / bridge 0.2.0 | `pi-desktop-android/runtime.lock.json` + `libs.versions.toml` | Gradle 锁 |
| 应用版本 | `package.json` / `app.json5` / `build.gradle.kts` | 各版文档 |

### 1.2 升级影响面

```
上游 tag
  ├── 载荷（pi-web 产物）→ 四版
  ├── agent 依赖（pi 包）→ 四版
  ├── 补丁集（按 tag 重建）→ HOS / Ark / Android
  ├── 运行时（Electron/Node/bridge）→ 分版
  └── 文档 / 能力清单 / 真机记录
```

---

## 2. 宪法合规检查

| 条款 | 合规 | 说明 |
|---|---|---|
| 第一条 编号 | ✅ | 统一 `011-version-<VERSION>`；历史迁移 |
| 第二条 分工 | ✅ | 父工程矩阵，子工程实现 |
| 第三条 文档集合 | ✅ | 父工程只写 spec/plan |
| 第四条 源码即真理 | ✅ | 以锁文件/版本表为准 |
| 第五条 上游零改动 | ✅ | 补丁幂等可逆 |
| 第六条 架构 | ✅ | 版本升级不改变架构 |
| 第八条 版本 | ✅ | 版本分离、tag、门禁 |
| 第九条 提交 | ✅ | `docs:` / `chore:` 前缀 |

---

## 3. 升级步骤（跨工程）

### 3.1 准备

1. 确认上游 tag / commit：
   ```bash
   git -C pi rev-parse v1.2.0^{commit}
   git -C pi-web rev-parse v0.12.1^{commit}
   ```
2. 确认四个子工程工作树干净；记录当前版本与指针。
3. 建立升级清单（见 §4 矩阵）。

### 3.2 逐版执行

#### Desktop

1. `package.json` 版本改为 `0.12.1`（如未改）。
2. `ci.yml` / `release.yml` 的 `PI_REF` / `PI_SHA` / `PI_WEB_REF` / `PI_WEB_SHA` 更新。
3. 清理中间产物 → `npm ci → prepare:next → collect:pi → smoke → package`。
4. 打包态启动 + 基本功能（Key / 工作区 / 对话）。
5. 打 tag `v0.12.1`，发布 Release。
6. 回写 `specs/011-version-0.12.1/`、README、`docs/`。

#### HOS

1. `AppScope/app.json5`：`versionName=0.12.1`、`versionCode=1201`。
2. `patches/v0.12.1/` 两条补丁重建/验证。
3. `collect-runtime → prepare-next → smoke-node22 → collect-pi → collect-app`。
4. hvigor 构建 + 签名 + 真机安装。
5. 真机三条基本路径 + 能力复测。
6. 打 tag `v0.12.1`；回写 `specs/011-version-0.12.1/` 与能力清单。

#### Ark

1. 确认 `payload/` 来自 pi-web v0.12.1；`patches/v0.12.1/` 六条。
2. 确认 `oh-package.json5`：bridge 0.1.0 / runtime 24.2.0。
3. `assemble-payload → hvigor`；签名 HAP；真机安装。
4. 验证启动、对话、工具、Git、权限。
5. 打 tag `v0.12.1`；回写 `specs/011-version-0.12.1/`。
6. 应用版本保持 `0.1.0` / `100`。

#### Android

1. 确认 `PI_WEB_TAG=v0.12.1`、`PDA_NODE_RUNTIME_VER=24.22.1`。
2. 确认 `runtime.lock.json` 与 `libs.versions.toml` 一致；`patches/v0.12.1/` 六条。
3. 预取 AAR → `verify-runtime` → `assemble-payload` → `gradle`。
4. 签名 APK / debug 兜底；真机安装；`/facts` + 对话。
5. 打 tag `v0.12.1`，workflow 发布 Release；回写文档。
6. 应用版本保持 `0.1.0` / `1`。

### 3.3 收尾

1. 更新父工程 submodule 指针（先提交子工程）。
2. 更新父工程 `011` 矩阵（如有变化）。
3. 检查四版文档的上游基线、版本号、补丁数一致。
4. 检查历史版本目录未回改。
5. 提交：子工程 `docs:` / `chore:`；父工程 `chore:`（指针）。

---

## 4. 验证矩阵

| 检查 | Desktop | HOS | Ark | Android | 方法 |
|---|---|---|---|---|---|
| 上游 tag/SHA | ✅ | ✅ | ✅ | ✅ | `git rev-parse` |
| 应用版本 | 0.12.1 | 0.12.1/1201 | 0.1.0/100 | 0.1.0/1 | 读配置 |
| 运行时版本 | Electron 43.7.9 | Electron 37.2.3 | bridge 0.1.0 / rt 24.2.0 | bridge 0.2.0 / rt 24.22.1 | 读锁文件 |
| 补丁数 | 0 | 2 | 6 | 6 | `ls patches/v0.12.1` |
| 补丁可逆 | — | ✅ | ✅ | ✅ | 应用/反向/SHA-256 |
| 构建 | Forge | hvigor | hvigor | Gradle | 构建日志 |
| 产物 | 安装包 | HAP | HAP | APK | 文件 + 哈希 |
| 启动 | 打包态 | 真机 | 真机 | 真机 | 日志/截图 |
| 对话 | 打包态 | 真机 | 真机 | 真机 | 会话 JSONL |
| 发布证据 | Release | 真机记录 | 真机记录 | Release | run id/URL |

---

## 5. 数据与锁文件

| 文件 | 工程 | 升级时必须改 |
|---|---|---|
| `pi-desktop/package.json` / lock | Desktop | 应用版本 / Electron |
| `pi-desktop/.github/workflows/*.yml` | Desktop | 上游 tag + SHA |
| `pi-desktop-hos/AppScope/app.json5` | HOS | 版本 / versionCode |
| `pi-desktop-hos/patches/v0.12.1/` | HOS | 补丁重建 |
| `pi-desktop-ark/AppScope/app.json5` | Ark | 版本（本次不变） |
| `pi-desktop-ark/oh-package.json5` / lock | Ark | bridge / runtime |
| `pi-desktop-ark/patches/v0.12.1/` | Ark | 补丁 |
| `pi-desktop-android/app/build.gradle.kts` | Android | 版本（本次不变） |
| `pi-desktop-android/gradle/libs.versions.toml` | Android | AAR 版本 |
| `pi-desktop-android/runtime.lock.json` | Android | Node / bridge / SHA |
| `pi-desktop-android/.github/workflows/build-release.yml` | Android | `PI_WEB_TAG` / runtime 版本 |

---

## 6. 错误处理与回滚

| 场景 | 处理 |
|---|---|
| 上游 tag 不存在 / SHA 不符 | 停止；核对 tag；不得跳过断言 |
| 补丁冲突 | 停止；不得 `--3way`；人工评估或登记为不需要 |
| 构建门禁失败 | 修复后重跑；不得绕过 |
| 真机失败 | 记录阶段/错误/日志；不得改判通过 |
| 发布错误 | 删除错误 Release / tag（需明确授权）；重新发布 |
| 运行时包不可用 | 用本地归档兜底；仍失败则停止升级 |

**回滚：**

- 子工程：`git revert` 升级提交；恢复旧指针；重新构建。
- 父工程：恢复 submodule 指针。
- 历史版本目录不删除，便于对照。

---

## 7. 测试考量

- 版本矩阵脚本化核对（grep 锁文件）。
- 补丁可逆自动化（应用/反向/SHA-256）。
- 载荷 tar 内容与裁剪规则。
- 品牌化 0 残留。
- 真机基本路径 + 权限/选择器/外链。
- CI 失败路径（错误 tag、错误 SHA）。

---

## 8. 文件清单

| 文件 | 作用 |
|---|---|
| `specs/011-version-0.12.1/spec.md` / `plan.md` | 本文档 |
| 子工程 `specs/011-version-0.12.1/` | 单版实现与证据 |
| `patches/v0.12.1/` | 补丁集 |
| `runtime.lock.json` / `oh-package.json5` / `libs.versions.toml` | 运行时锁定 |
| `.github/workflows/*` | CI/Release |
| `docs/能力对照清单.md` | 四版能力汇总 |

---

## 9. 与规格交叉引用

| FR | 落点 |
|---|---|
| FR-011-001/002/003 | 上游 tag/SHA + 补丁纪律 |
| FR-011-004/005/006/007 | 各版版本配置 + tag |
| FR-011-008–012 | 运行时锁文件 |
| FR-011-013–016 | `patches/v0.12.1/` |
| FR-011-017–020 | Release / 真机 / 文档 |
| FR-011-021–023 | 父/子 `011` 目录 + 矩阵 |
