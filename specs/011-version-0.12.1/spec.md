# 011 — 版本更新：0.12.1（version）跨工程功能规格

> **Module:** version ｜ **Status:** Implemented ｜ **Last Updated:** 2026-10-31
> **上游基线:** `pi` v1.2.0（`06a466935536f8c9bcacb2d5efc0fc021be1bdff`）· `pi-web` v0.12.1（`f83b1a502f8e434555a7153fe08e1c135dd65ca9`）
> **发布 tag:** `v0.12.1`
> **适用范围:** `pi-desktop` · `pi-desktop-hos` · `pi-desktop-ark` · `pi-desktop-android`
> **父/子分工:** 本文写**跨工程版本口径与矩阵**；各子工程的升级步骤、补丁、真机证据写在子工程
> `specs/011-version-0.12.1/`。

---

## 1. 模块概述

### 1.1 目的

把四个版别统一收敛到 `v0.12.1` 基线，并回答三个容易混淆的问题：

1. **上游版本**是什么？（`pi` / `pi-web` 的 tag 与 commit）
2. **应用版本**是什么？（四个壳各自的 `versionName` / `versionCode`）
3. **发布 tag** 是什么？（`v0.12.1`，只作发布标识）

### 1.2 解决的问题

- 四版应用版本不一致（0.12.1 vs 0.1.0），容易误判“版本回退”。
- 运行时版本（Electron / Node / bridge / runtime）分散在 package、ohpm、Gradle、lock 文件。
- 补丁数不同（0 / 2 / 6 / 6），升级时容易漏评估。
- 发布产物与 tag、上游 commit 的追溯链不完整。
- 历史版本文档（HOS 0.11.1）需要保留，不能因统一编号而丢失。

### 1.3 范围

**包含**

- 四版上游基线一致性：`pi` v1.2.0 + `pi-web` v0.12.1。
- 四版应用版本 / `versionCode` / 产物 / tag 的对照矩阵。
- 运行时版本矩阵：Electron / Node / node-bridge / node-runtime。
- 补丁集与版本目录（`patches/v0.12.1/`）。
- 发布与证据要求：Release / run id / 真机记录 / 哈希。
- 历史版本目录保留与编号迁移（`101-version-*` → `011-version-*`）。

**不包含**

- 修改上游源码；只允许幂等补丁。
- 具体构建脚本实现（见 `012` 与子工程）。
- 品牌化（见 `013`）。
- 应用商店上架材料（子工程 107）。

---

## 2. 版本矩阵

### 2.1 上游

| 项 | 版本 | Commit | 说明 |
|---|---|---|---|
| `pi` | **v1.2.0** | `06a466935536f8c9bcacb2d5efc0fc021be1bdff` | agent harness |
| `pi-web` | **v0.12.1** | `f83b1a502f8e434555a7153fe08e1c135dd65ca9` | UI + Next 服务 |

### 2.2 四版应用与运行时

| 版别 | 应用版本 | versionCode | 运行时 | Node | 补丁 | 产物 |
|---|---|---|---|---|---|---|
| `pi-desktop` | **0.12.1** | —（package.json） | Electron **43.7.9** | **24.21.0** | **0** | Squirrel / zip / dmg / deb / rpm |
| `pi-desktop-hos` | **0.12.1** | **1201** | Electron-on-HarmonyOS **37.2.3** | **22.17.0** | **2** | 签名 HAP |
| `pi-desktop-ark` | **0.1.0** | **100** | node-bridge **0.1.0** + node-runtime **24.2.0** | **24.2.0** | **6** | 签名 HAP |
| `pi-desktop-android` | **0.1.0** | **1** | node-bridge **0.2.0** + node-runtime **24.22.1** | **24.22.1** | **6** | 签名 APK |

> **口径：** 应用版本是“产品版本”，上游版本是“载荷版本”。Ark/Android 的 `0.1.0` 是首发预发布，
> 不随上游 `0.12.1` 改名；发布 tag 统一为 `v0.12.1`。

### 2.3 发布 tag 与产物

| 版别 | tag | 发布方式 | 校验和 | 证据 |
|---|---|---|---|---|
| Desktop | `v0.12.1` | GitHub Release（三 OS） | 产物哈希 | workflow run id + Release URL |
| HOS | `v0.12.1` | 本地签名 HAP + 真机 | HAP 哈希 | 真机记录 |
| Ark | `v0.12.1` | 本地签名 HAP + 真机 | HAP 哈希 | 真机记录 |
| Android | `v0.12.1` | GitHub Release（签名 APK） | `.sha256` | workflow run id + Release URL |

---

## 3. 功能需求

### 3.1 上游基线

- **FR-011-001**（MUST）：四版载荷必须来自 `pi-web` **v0.12.1**，agent 必须来自 `pi` **v1.2.0**。
- **FR-011-002**（MUST）：CI / 构建脚本必须断言 tag 对应的 commit SHA；上游重打 tag 必须失败。
- **FR-011-003**（MUST NOT）：不得修改上游 `pi` / `pi-web` 源码；只允许 `patches/v0.12.1/` 的幂等补丁。

### 3.2 应用版本

- **FR-011-004**（MUST）：Desktop / HOS 的 `versionName` 必须为 `0.12.1`；HOS 的 `versionCode` 必须为 `1201`。
- **FR-011-005**（MUST）：Ark / Android 的 `versionName` 必须为 `0.1.0`；`versionCode` 分别为 `100` / `1`。
- **FR-011-006**（MUST）：`versionCode` 必须单调递增；不得回退或复用。
- **FR-011-007**（MUST）：发布 tag 必须为 `v0.12.1`；**MUST NOT** 把 tag 注入 `versionName` / `versionCode`（Android 明确不注入）。

### 3.3 运行时

- **FR-011-008**（MUST）：Desktop 必须使用 Electron 43.7.9 / Node 24.21.0。
- **FR-011-009**（MUST）：HOS 必须使用 Electron-on-HarmonyOS 37.2.3 / Node 22.17.0 / ABI 138。
- **FR-011-010**（MUST）：Ark 必须使用 `@fellow99/node-bridge` 0.1.0 + `@fellow99/node-runtime` 24.2.0。
- **FR-011-011**（MUST）：Android 必须使用 `io.github.fellow99:node-bridge` 0.2.0 + `node-runtime` 24.22.1（small-ICU）。
- **FR-011-012**（MUST）：运行时版本必须与锁文件/版本表一致；升级必须同时改锁文件与文档。

### 3.4 补丁

- **FR-011-013**（MUST）：HOS 必须保留 2 条补丁、Ark/Android 各 6 条补丁，目录为 `patches/v0.12.1/`。
- **FR-011-014**（MUST）：补丁必须幂等、可逆；应用后反向还原必须逐文件 SHA-256 一致。
- **FR-011-015**（MUST）：补丁冲突必须终止；禁止 `--3way`。
- **FR-011-016**（MUST）：补丁数与语义断言必须写回子工程 `specs/` 与 `patches/README.md`。

### 3.5 发布与证据

- **FR-011-017**（MUST）：每个版别必须有可追溯的发布证据：tag、产物、哈希、构建/run id、真机记录。
- **FR-011-018**（MUST）：Release 说明必须包含上游 tag、运行时版本、补丁数。
- **FR-011-019**（MUST）：签名材料、keystore、API Key 不得入库；日志不得回显。
- **FR-011-020**（MUST）：历史版本结论（HOS 0.11.1、旧真机轮次、旧能力数值）必须保留，不得回改。

### 3.6 文档与编号

- **FR-011-021**（MUST）：版本模块编号必须为 `011-version-<VERSION>`；历史 `101-version-*` 必须迁移。
- **FR-011-022**（MUST）：父工程 `specs/011-version-0.12.1/` 与子工程同名目录必须存在，内容分工见宪法。
- **FR-011-023**（MUST）：版本矩阵、运行时版本、补丁数必须与源码/锁文件一致；不一致时以源码为准并回写。

---

## 4. 关键实体

| 实体 | 说明 | 关键属性 |
|---|---|---|
| 上游基线 | 载荷与 agent 来源 | `pi` tag/SHA、`pi-web` tag/SHA |
| 应用版本 | 各壳产品版本 | `versionName`、`versionCode` |
| 运行时 | Electron / Node / bridge / runtime | 版本、ABI、SHA-256 |
| 补丁集 | 对 pi-web 的可逆修改 | 目录、条数、幂等/可逆 |
| 发布产物 | 安装包 / HAP / APK | 文件名、哈希、tag、run id |
| 历史版本 | 旧版本文档与证据 | 目录保留、不回改 |

---

## 5. 验收场景

### 场景：上游一致

- Given 四个子工程均已检出
- When 检查 `pi` / `pi-web` 的 tag 与 commit
- Then 全部为 v1.2.0 / `06a46693…` 与 v0.12.1 / `f83b1a50…`

### 场景：版本矩阵一致

- Given 读取四个子工程的版本配置
- When 对照本文 §2.2
- Then Desktop 0.12.1、HOS 0.12.1/1201、Ark 0.1.0/100、Android 0.1.0/1

### 场景：补丁可逆

- Given pristine pi-web v0.12.1
- When 应用并反向还原各版补丁
- Then 逐文件 SHA-256 一致

### 场景：发布可追溯

- Given 某版发布完成
- When 检查 Release / 真机记录
- Then 能追到 tag、上游 SHA、运行时版本、补丁数、产物哈希

---

## 6. 非功能需求

- **可复现**：同一 tag + 锁文件产出等价包（签名材料除外）。
- **可追溯**：发布链完整（tag → 上游 → 运行时 → 产物）。
- **一致性**：四版会话/配置/目录格式保持一致。
- **安全**：签名与密钥不入库。
- **可维护**：版本升级流程化，历史保留。

---

## 7. 假设与约束

- 上游 tag/分支在各仓库可用；网络可达（或本地镜像）。
- 运行时包在 ohpm / Maven Central 可用；离线时有本地兜底。
- 应用版本 `0.1.0` 是预发布，不代表功能缺失。
- 发布 tag 由发布者创建；CI 只响应 tag。

---

## 8. 依赖

- 父工程：`constitution.md`、`012-building-workflow`、`013-branding`。
- 子工程：各自 `011-version-0.12.1/`、构建文档、补丁目录、真机记录。
- 外部：GitHub、ohpm、Maven Central、模型 Provider。
