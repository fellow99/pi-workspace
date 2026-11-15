# 101 — 版本更新：0.17.0（version）跨工程技术方案与执行结果

> **Module:** 101-version-0.17.0 ｜ **Status:** In-progress（四端构建完成；签名与真机复验待后续） ｜ **Last Updated:** 2026-11-15
> **对应规格:** [`spec.md`](./spec.md)

---

## 1. 执行顺序

1. **环境评估**：各子模块本地 main 均在 v0.15.0 发布线；`pi` @v1.5.2、`pi-web` @v0.17.0 已由用户固化。
2. **冲突评估**：pi-web v0.15.0→v0.17.0 diff（297 文件）与 19 条补丁接触面比对 + 累加顺序 dry-run → 冲突集中于 DirectoryPicker。
3. **向用户汇报冲突 → 批准 rebase**。
4. **补丁 rebase**（HOS 0001 / Ark 0005+0006+0007 / Android 0005+0006+0007）→ `patches/v0.17.0/`。
5. **逐工程**：清理 → 版本号 0.17.0 → 上游钉住 → 构建 → 冒烟 → 提交。
6. **README/官网更新**。
7. **Code Review** → 修复。
8. **打 tag v0.17.0**（五子工程 + 父工程收尾）。

## 2. 各工程执行结果

### 2.1 补丁 rebase（关键）

| 工程 | 原冲突 | 处理 | 验证 |
|---|---|---|---|
| HOS 0001-workspace-picker-roots | DirectoryPicker.tsx 冲突 | 整体重写 + 移植 #1622 IME 守卫 | prepare-next apply→build→revert 全绿 |
| Ark 0005/0006/0007 | DirectoryPicker 冲突 | 合并为 `0005-directory-picker.patch` | assemble-payload apply→build→revert；载荷断言 6 marker 命中 |
| Android 0005/0006/0007 | 同 Ark | 同 Ark（android-sandbox 命名） | 载荷断言 6 marker 命中 |

### 2.2 各工程构建

- **pi-desktop**：0 补丁；version 0.17.0；prepare-next BUILD_ID `EhX9_aizd5OHP23x77cQb`；smoke 10 PASS；probe-renderer 7 PASS（含 PTY）；Setup.exe/nupkg/zip 0.17.0。
- **pi-desktop-hos**：AppScope 1700/0.17.0；prepare-next BUILD_ID `LZ9grZ3lM88e4xPRFQrRD`（3 补丁回退 pristine）；smoke-node22（Node 22.23.5 + --allow-any-node）8 PASS/2 SKIP；collect-pi 175.2 MB；HAP 未签名 396 MB，pack.info code 1700/0.17.0。
- **pi-desktop-ark**：AppScope 1700/0.17.0；assemble-payload 归档 83.8 MB（载荷断言 6 marker 命中）；HAP 未签名 134 MB，1700/0.17.0。
- **pi-desktop-android**：build.gradle 1700/0.17.0；assemble-payload 归档 75.8 MB（载荷断言 6 marker 命中）；APK debug 493 MB / release 482 MB，aapt 1700/0.17.0。
- **pi-desktop-website**：两页 + README 更新至 v0.17.0（Device 复验待后续口径）。

## 3. 待办 / 已知限制

- **签名 HAP 未产出**（本机签名材料缺失；用户指示签名信息有误、真机后再处理）。
- **鸿蒙/安卓真机复验未执行**（用户明晚安排）。
- smoke-node22 版本门禁 SKIP（本机无 Node 22.17.x，用 22.23.5 + --allow-any-node 功能验证）。
