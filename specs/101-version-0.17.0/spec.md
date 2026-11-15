# 101 — 版本更新：0.17.0（version）跨工程功能规格

> **Module:** version ｜ **Status:** In-progress（四端构建完成；签名与真机复验待后续） ｜ **Last Updated:** 2026-11-15
> **上游基线:** `pi` v1.5.2（`b4afdddf44e059c67ccf5097e5317d913521b470`，Branch_v1.5.2）· `pi-web` v0.17.0（`174bff967807cdb59abb0204198d3d7cc19c862d`，Branch_v0.17.0）
> **发布 tag:** `v0.17.0`
> **适用范围:** `pi-desktop` · `pi-desktop-hos` · `pi-desktop-ark` · `pi-desktop-android` · `pi-desktop-website`
> **父/子分工:** 本文写**跨工程版本口径与矩阵**；各子工程的升级步骤、补丁、构建证据写在各自提交与 `patches/v0.17.0/`。

---

## 1. 模块概述

### 1.1 目的

把五个工程统一收敛到 `v0.17.0` 基线，并把上游基线升到 `pi` v1.5.2 + `pi-web` v0.17.0。

### 1.2 解决的问题

- **上游 `pi-web` v0.15.0 → v0.17.0（297 文件，+12278/−1517）**：含 v0.16.0（55 提交）与 v0.17.0 两批。
  功能要点（详见 plan §冲突评估）：v0.16.0 升级 **pi 至 1.5.2**（与固化的 pi v1.5.2 对齐）、session-export 私有临时目录（#1626）、
  DirectoryPicker 跟随输入路径（#1550）/IME Escape（#1622）/路径提示 i18n（#1651）、llama.cpp provider、搜索/工具开关等；
  v0.17.0 文件 100MB 上限、侧聊搜索、Agents 条、MCP 重连、npx 清除宿主环境、可信目录工程配置等安全加固。
  均为纯前端/纯 JS 层变更，不涉宿主桥/运行时/内核 → 对四端**无明显行为冲突**。
- **补丁冲突（dry-run 实测）**：`pi` 停 `v1.5.2`（升级）；`pi-web` 停 `v0.17.0`。
  原 19 条补丁（0/3/8/8）中，**HOS 0001、Ark/Android 0005/0006/0007** 因上游重构 `DirectoryPicker.tsx`（navigateTo 返回类型、canSelect/canCreate、handleSelect、IME 守卫、pathInputRef）**冲突**。
- **补丁 rebase（经用户批准）**：
  - HOS `0001-workspace-picker-roots`：整体重写 DirectoryPicker（主目录 Tab/面包屑/权限流/无路径输入），移植上游 #1622 IME 守卫 ×2；
  - Ark/Android `0005+0006+0007` → 合并为单个 `0005-directory-picker.patch`（保留全部 marker：`pi-hos-picker-contract` / `pi-hos-perm-manual`）；
  - 其余干净应用。
- **四端构建**：均编译通过（详见 plan、各工程提交与 `logs/20261115-1/`）。
  因本机缺签名材料（用户指示：签名信息有误，真机后再处理），HOS/Ark 本轮产出**未签名 HAP**；Android 产出 debug/release APK。

### 1.3 范围

**包含**：五版上游基线一致性（pi 升至 v1.5.2、pi-web 升至 v0.17.0）；应用版本/versionCode/产物/tag 矩阵；
补丁集版本目录（`patches/v0.17.0/`）rebase 与幂等验证；各工程构建产物；官网/README 更新；发布 tag；桌面打包态验证。
**不包含**：修改上游源码（只允许幂等补丁）；签名 HAP；鸿蒙/安卓真机复验（待用户明晚安排）。

---

## 2. 版本矩阵

### 2.1 上游

| 上游 | 版本 | SHA/Branch |
|---|---|---|
| `pi` | v1.5.2 | `b4afdddf44e059c67ccf5097e5317d913521b470` / Branch_v1.5.2 |
| `pi-web` | v0.17.0 | `174bff967807cdb59abb0204198d3d7cc19c862d` / Branch_v0.17.0 |

### 2.2 应用

| 工程 | 版本 | versionCode | 构建产物 |
|---|---|---|---|
| pi-desktop | 0.17.0 | — | Setup.exe / nupkg / zip |
| pi-desktop-hos | 0.17.0 | 1700 | 未签名 HAP（396 MB） |
| pi-desktop-ark | 0.17.0 | 1700 | 未签名 HAP（134 MB） |
| pi-desktop-android | 0.17.0 | 1700 | debug/release APK |
| pi-desktop-website | 0.17.0 | — | 静态站 |

### 2.3 补丁矩阵

| 工程 | 补丁数 | 目录 |
|---|---|---|
| pi-desktop | 0 | — |
| pi-desktop-hos | 3 | `patches/v0.17.0/`（0001 rebase+2 clean） |
| pi-desktop-ark | 6 | `patches/v0.17.0/`（0005 = 合并 0005+0006+0007） |
| pi-desktop-android | 6 | `patches/v0.17.0/`（0005 = 合并 0005+0006+0007） |

---

## 3. 验证汇总

执行完成后由 plan.md 回填。

## 4. 测试

- 桌面：`smoke-desktop` 10 PASS、`probe-renderer` 7 PASS（含 PTY）。
- HOS/Ark/Android：构建冒烟通过；真机复验待用户安排。
- 敏感信息零落盘（DeepSeek Key 仅运行期注入）。
