# 101 — 版本更新：0.18.0（version）任务清单

> **对应规格:** [`spec.md`](./spec.md) ｜ **方案:** [`plan.md`](./plan.md)
> 状态：`[ ]` 待做 ｜ `[x]` 完成 ｜ `[!]` 阻塞

## 阶段 0 — 前置

- [x] T0-1 核实 `pi` @ `Branch_v1.6.0` / tag `v1.6.0`（`faac5858`）
- [x] T0-2 核实 `pi-web` @ `Branch_v0.18.0` / tag `v0.18.0`（`2daa0d13`）
- [x] T0-3 建立 v0.18.0 detached worktree 用于补丁分析与生成（`../.tmp/wt-018`）
- [x] T0-4 26 条补丁 hunk 累加 dry-run，定位 4 条冲突
- [x] T0-5 向用户汇报冲突并获批准（rebase 策略 / PlantUML 选项 A）

## 阶段 1 — 补丁 rebase（`patches/v0.18.0/`，UTF-8/LF）

- [x] T1-1 pi-desktop：`0001-sync-daemon-settings-tab` 重新生成
- [x] T1-2 pi-desktop-hos：`0001`/`0002`/`0003`/`0004` 重新生成（`0002` 删冗余 hunk）
- [x] T1-3 pi-desktop-ark：`0001`/`0002`/`0003`/`0004`/`0005`/`0007`/`0008` 重新生成（`0001` 重定位、`0004` 删冗余 hunk）
- [x] T1-4 pi-desktop-android：同 Ark（android 命名）
- [x] T1-5 往返校验：应用 → 反向 → 受影响文件 SHA-256 逐条还原
- [x] T1-6 各工程 `patches/README.md` 更新到 v0.18.0 口径（上游 tag、commit、受影响文件）
- [x] T1-7 各工程脚本中的上游 tag 常量（`PI_WEB_PATCH_TAG` 等）更新为 `v0.18.0`

## 阶段 2 — 各工程（清理 → 版本 → 构建 → 提交）

### pi-desktop
- [x] T2-1 清理 `out/`、`.vite/`、`pi-dist/`、`logs/`
- [x] T2-2 `package.json` version → `0.18.0`；`pin-upstream.mjs` baseline → pi v1.6.0 / pi-web v0.18.0
- [x] T2-3 `prepare-next` + `collect-pi` + `make`，产物含 0.18.0
- [x] T2-4 `smoke-desktop` 10 PASS / 0 FAIL（`probe-renderer` **未执行**：宿主进程即 pi-desktop，带调试端口重启会中断会话 → 转交测试机，见 TC-B-17）
- [x] T2-5 提交

### pi-desktop-hos
- [x] T3-1 清理 `build/`、`.hvigor/`、`oh_modules/`、`pi-dist/`、`pi-dist.tar.gz`、`runtime/`、`logs/`
- [x] T3-2 `AppScope/app.json5` → versionName `0.18.0` / versionCode `1800`；`prepare-next.mjs` 上游 tag 常量
- [x] T3-3 `prepare-next` + `collect-pi` + `collect-runtime` + 构建 HAP
- [x] T3-4 冒烟（`smoke-node22 --allow-any-node`）：8 PASS / 2 SKIP / 0 FAIL（对**最终**载荷执行；SSE 探针因无真实会话返回 404 而 SKIP）
- [x] T3-5 提交

### pi-desktop-ark
- [x] T4-1 清理 `build/`、`.hvigor/`、`oh_modules/`、`entry/src/main/resources/resfile/pi-web/`、`logs/`
- [x] T4-2 `AppScope/app.json5` → `0.18.0` / `1800`；`assemble-payload.mjs` 上游 tag 常量
- [x] T4-3 `assemble-payload --apply` + 构建 HAP
- [x] T4-4 载荷断言（marker 命中）通过
- [x] T4-5 提交

### pi-desktop-android
- [x] T5-1 清理 `build/`、`app/build/`、`.gradle/`、`app/src/main/assets/pi-web/`、`app/src/main/assets/cli-tools/`、`logs/`
- [x] T5-2 `build.gradle.kts` → versionName `0.18.0` / versionCode `1800`；`assemble-payload.mjs` 上游 tag 常量
- [x] T5-3 `assemble-payload --apply` + Gradle 构建 APK
- [x] T5-4 载荷断言 + aapt 版本核对
- [x] T5-5 提交

### pi-desktop-website
- [x] T6-1 `index.html` / `index_zh.html` 版本号、版本描述、能力矩阵、补丁数更新到 0.18.0
- [x] T6-2 `README.md` 更新
- [x] T6-3 提交

## 阶段 3 — 全局收尾

- [x] T7-1 父工程 `README.md` / `README_zh.md`、`specs/README.md`、`specs/overall-*` 版本口径更新
- [x] T7-2 四子工程 `README.md` / `README_zh.md` 更新
- [x] T7-3 `pi-desktop/.github/workflows/{ci,release}.yml`：`PI_REF`/`PI_SHA`/`PI_WEB_REF`/`PI_WEB_SHA`
- [x] T7-4 `pi-desktop-android/.github/workflows/build-release.yml`：`PI_WEB_TAG`（及 pi 钉住项）
- [x] T7-5 父工程 submodule 指针更新 + 提交

## 阶段 4 — Code Review

- [x] T8-1 `requesting-code-review`：分派 reviewer（Critical/Important/Minor 分级）
- [x] T8-2 产出 `logs/20261122-1/REVIEW_REPORT.md`
- [x] T8-3 `receiving-code-review`：逐条核验 → 修 Critical/Important → 复评
- [x] T8-4 提交修复

## 阶段 5 — Tag 与交接

- [x] T9-1 五个子工程打 tag `v0.18.0`
- [x] T9-2 父工程打 tag `v0.18.0`
- [x] T9-3 产出测试交接文档 `D:\knowledge-base\pi-workspace\011-version-0.18.0-测试交接.md`
