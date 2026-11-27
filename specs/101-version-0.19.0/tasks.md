# 101 — 版本更新：0.19.0（version）任务清单

> **对应规格:** [`spec.md`](./spec.md) ｜ **方案:** [`plan.md`](./plan.md)
> 状态：`[ ]` 待做 ｜ `[x]` 完成 ｜ `[!]` 阻塞

## 阶段 0 — 前置

- [x] T0-1 核实 `pi` @ `Branch_v1.6.1` / tag `v1.6.1`（`ef4a1cfc`）
- [x] T0-2 核实 `pi-web` @ `Branch_v0.19.0` / tag `v0.19.0`（`7b77aa1a`）
- [x] T0-3 建立 v0.19.0 detached worktree（`../.tmp/wt-019`）用于补丁分析与生成
- [x] T0-4 16 条补丁 hunk 累加 dry-run，定位 **6 条冲突 / 10 条干净**（desktop 1 / hos 1 / ark 2 / android 2）
- [x] T0-5 向用户汇报冲突并获批准（结论：零阻断性冲突，仅补丁上下文漂移）

## 阶段 1 — 补丁 rebase（`patches/v0.19.0/`，UTF-8/LF）

- [x] T1-1 pi-desktop：`0001-sync-daemon-settings-tab` 重放（`SettingsPanel.tsx` 6 处插入）
- [x] T1-2 pi-desktop-hos：`0001` 重放（`DirectoryPicker.tsx`，采纳上游 `readJsonOrThrow`/`errorMessage`）；`0002`/`0003` 逐字节沿用
- [x] T1-3 pi-desktop-ark：`0001` 导入后移、`0005` 重放；`0002/0003/0004/0008` 逐字节沿用
- [x] T1-4 pi-desktop-android：同 Ark（`0001` 的 `lib/terminal-capability.ts` Android PTY 分支不变）
- [x] T1-5 往返校验：四端 apply → 反序 reverse → `git status --porcelain` 为空
- [x] T1-6 各工程 `patches/README.md` 新增「0.0 v0.19.0 rebase 记录」，旧记录顺延 `0.0b`
- [x] T1-7 各工程脚本上游 tag 常量：`PI_WEB_PATCH_TAG` / `EXPECTED_PI_WEB_VERSION` / `pin-upstream.mjs` baseline → v0.19.0 / v1.6.1

## 阶段 2 — 各工程（清理 → 版本 → 构建 → 提交）

### pi-desktop
- [x] T2-1 清理派生目录（`out/make` 保留，删掉残留的 `0.18.0` zip）
- [x] T2-2 `package.json` version → `0.19.0`
- [x] T2-3 `prepare-next`（BUILD_ID `vxIdm0QC9upmemdyHwCDn`）+ `collect-pi`（207.1 MB）+ `npx electron-forge make`
- [x] T2-4 `smoke-desktop` 10 PASS / 0 FAIL
- [!] T2-5 `probe-renderer` **阻塞**：宿主即 0.18.0 安装版，壳 `requestSingleInstanceLock()` 劝退新实例 → 转 CI 产物人工验证
- [x] T2-6 提交（`build:`）+ tag `v0.19.0` + 推送（main + tag）

### pi-desktop-hos
- [x] T3-1 清理 `build/`、`.hvigor/`、`pi-dist/`、`pi-dist.tar.gz`
- [x] T3-2 `AppScope/app.json5` → versionName `0.19.0` / versionCode `1900`
- [x] T3-3 `collect-runtime` + `prepare-next`（BUILD_ID `q-l7KIxPhITdBEhOomW4T`）+ `collect-pi`（160.9 MB）+ `collect-app`（9 文件）
- [x] T3-4 `smoke-node22 --allow-any-node` 8 PASS / 2 SKIP / 0 FAIL
- [x] T3-5 HAP debug 签名 382,924,600 B，`pack.info` = 1900/0.19.0
- [ ] T3-6 提交 + tag `v0.19.0`

### pi-desktop-ark
- [x] T4-1 清理 `build/`、`entry/src/main/resources/resfile/pi-web/`
- [x] T4-2 `AppScope/app.json5` → `0.19.0` / `1900`
- [x] T4-3 `assemble-payload --apply`（6 补丁往返 + 依赖门禁 + 重编 678 s）→ 归档 85.7 MiB
- [x] T4-4 载荷标记断言通过；HAP debug 签名 136,077,761 B，`pack.info` = 1900/0.19.0
- [ ] T4-5 提交 + tag `v0.19.0`

### pi-desktop-android
- [x] T5-1 清理 `build/`、`app/build/`、`.gradle/`、`app/src/main/assets/pi-web/`
- [x] T5-2 `app/build.gradle.kts` → versionName `0.19.0` / versionCode `1900`
- [x] T5-3 `assemble-payload --apply`（6 补丁往返 + 依赖门禁 + 重编）→ 归档 76.8 MiB，包内 0.19.0 / 16.4.6 / 1.6.1
- [x] T5-4 `assembleDebug` 482,106,993 B + `assembleRelease` 473,074,747 B，`aapt` = 1900/0.19.0
- [!] T5-5 装机 **阻塞**：`adb install -r` / `pm install` / `adb_install_need_confirm=0` 全部返回
      `INSTALL_FAILED_USER_RESTRICTED: Install canceled by user`（MIUI「USB 安装」限制，设备端无弹窗）；
      旧包 `org.fellow99.pi.PiDesktopAndroid` 0.15.0 **未卸载**（按用户要求）
- [ ] T5-6 提交 + tag `v0.19.0`

### pi-desktop-website
- [x] T6-1 `index.html` / `index_zh.html`：版本号 / versionCode / 补丁数 / 各版状态
- [x] T6-2 `README.md`：版本矩阵、分发状态、事实表
- [ ] T6-3 提交 + tag `v0.19.0`

## 阶段 3 — 全局收尾

- [x] T7-1 父工程 `README.md` / `README_zh.md` 版本矩阵与状态
- [x] T7-2 四子工程 `README*.md` / `AGENTS.md`
- [x] T7-3 `pi-desktop/.github/workflows/{ci,release}.yml`：`PI_REF`/`PI_SHA`/`PI_WEB_REF`/`PI_WEB_SHA`
- [x] T7-4 `pi-desktop-android/.github/workflows/build-release.yml`：`PI_WEB_TAG`
- [x] T7-5 父工程 `docs/能力对照清单.md` 新增 §19（本轮测试边界与构建证据）
- [ ] T7-6 父工程 submodule 指针更新 + 提交 + tag `v0.19.0`

## 阶段 4 — Code Review

- [ ] T8-1 `requesting-code-review`：分派 reviewer（Critical/Important/Minor 分级）
- [ ] T8-2 产出 `logs/20261127-1/REVIEW_REPORT.md`
- [ ] T8-3 `receiving-code-review`：逐条核验 → 修 Critical/Important → 复评
- [ ] T8-4 提交修复

## 阶段 5 — CI 与交接

- [ ] T9-1 观察 `pi-desktop` 的 CI / Release 工作流（tag `v0.19.0` 触发）
- [ ] T9-2 观察 `pi-desktop-android` 的 `build-release.yml`（tag `v0.19.0` 触发）
- [ ] T9-3 向用户交付：产物路径、CI 运行链接、安卓装机需用户开启的开关
