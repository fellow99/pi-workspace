[中文](./README_zh.md) | English

---

# Pi Desktop Workspace

> A single-repo workspace aggregating every project required to ship [Pi](https://github.com/earendil-works/pi) (an
> extensible agent harness) and its browser UI [Pi Web](https://github.com/agegr/pi-web) as desktop apps across
> Windows, macOS, Linux, HarmonyOS and Android, in **four** product forms:
> **Pi Desktop** (Electron shell · Windows / macOS / Linux) ·
> **Pi Desktop · Electron on HarmonyOS Edition** (`pi-desktop-hos`) ·
> **Pi Desktop · HarmonyOS Edition** (`pi-desktop-ark`, ArkWeb + a real in-process Node runtime from the `@fellow99/node-bridge` + `@fellow99/node-runtime` ohpm packages) ·
> **Pi Desktop · Android Edition** (`pi-desktop-android`, system WebView + an in-process Node runtime from the `aosp-node-bridge` + Maven Central `node-runtime`).
> The upstream repositories, the runtimes and the brand material sit side by side as **git submodules plus plain
> sibling directories**, so the workspace layout reproduces the exact sibling-directory structure each wrapper's build
> scripts expect (`../pi`, `../pi-web`, `../assets`, `../harmonypc-electron`).
> The same container also hosts **`pi-agent-hub`**, a companion project — a multi-agent session hub that aggregates,
> reviews and reaches back into agent sessions across machines. It is **not** a fifth desktop edition and is not part
> of the wrappers' data plane.

---

## What is this

`Pi` is an upstream open-source, extensible agent harness (MIT, [earendil-works/pi](https://github.com/earendil-works/pi))
with four integration entry points — CLI / RPC / JSON / SDK. `Pi Web` is its community browser UI (MIT,
[agegr/pi-web](https://github.com/agegr/pi-web)) where **a single Node process serves both the SPA and `/api/*`**, and
the agent is created inside that process. Four wrapper projects move this whole stack onto the desktop
(the HarmonyOS Edition's Node runtime ships as two ohpm packages, `@fellow99/node-bridge` + `@fellow99/node-runtime`; the Android Edition uses `aosp-node-bridge` + the Maven Central `node-runtime` AAR):

- **`pi-desktop`** — an Electron desktop shell (Windows / macOS / Linux), Electron **43.7.9**: the main process hosts
  `pi-web`'s Next server (agent and `/api/*` included) **in-process**, and the renderer loads the official Pi Web UI
  from that same origin.
- **`pi-desktop-hos`** — **Pi Desktop · Electron on HarmonyOS Edition**: a HarmonyOS PC / tablet port built on the
  Electron-on-HarmonyOS runtime from `harmonypc-electron` (Electron **37.2.3** / Node **22.17.0**), packaged as a
  signable, distributable **HAP**.
- **`pi-desktop-ark`** — **Pi Desktop · HarmonyOS Edition**: the same product **without Chromium or Electron** — an
  ArkTS host whose **ArkWeb** view loads the local `pi-web` server, which runs on a **real Node.js runtime embedded in
  the app process** (`@fellow99/node-bridge` + `@fellow99/node-runtime`: `dlopen` + `node::Start`). Smaller payload,
  system WebView, no Electron runtime; functional target is parity with the Electron on HarmonyOS Edition.
- **`pi-desktop-android`** — **Pi Desktop · Android Edition**: like the HarmonyOS Edition but for Android — no Chromium
  or Electron, a Kotlin host whose **system WebView** loads the local pi-web server, which runs on a **real Node.js
  runtime inside the app process** (`aosp-node-bridge` **0.2.0** + the Maven Central `io.github.fellow99:node-runtime`
  AAR, Node **26.13.0** small-ICU, `dlopen` of `libnode.so`). Ships as a **signed APK** published from a tag-triggered
  GitHub Actions workflow; engine provenance is gated by `scripts/verify-runtime.mjs` against `runtime.lock.json`.
- **`@fellow99/node-bridge` / `@fellow99/node-runtime`** — **the runtime packages behind the HarmonyOS Edition**
  (an ArkTS HAR + NAPI loader, plus the Node engine as a separate HAR; both published on the OpenHarmony ohpm registry,
  source: [github.com/fellow99/ohos-node-bridge](https://github.com/fellow99/ohos-node-bridge)). They are **not** an app
  and **not** an edition — they are the pieces that let any HarmonyOS application host Node in-process. Their only
  registered consumer is `pi-desktop-ark` (regular ohpm dependency, no side-by-side checkout needed).

The wrappers only **assemble**: the UI is reused 100% from upstream and no upstream source is forked (see
`pi-desktop/docs/工程规划.md` §1.1–§1.2, `pi-desktop-hos/docs/工程规划.md` §1.1–§1.2,
`pi-desktop-ark/docs/000-工程规划.md`). Patch policy is counted **per edition**: the desktop shell ships **1** patch,
the Electron on HarmonyOS Edition **3** registered patches, and the HarmonyOS Edition applies its `pi-web` patches as
idempotent, reversible files (`pi-desktop-ark/patches/v0.18.0/`, **6** at v0.18.0), and the Android Edition applies the
parallel **6** idempotent patches in `pi-desktop-android/patches/v0.18.0/`.

This repository itself contains no product code — it is the **container** that holds these projects, so every sibling
directory reference resolves as soon as the workspace is cloned.

## Repository layout

```
pi-workspace/                       # this repo — submodule container (the parent project)
├── pi/                             # [submodule] upstream agent harness (host + SDK, MIT)
├── pi-web/                         # [submodule] upstream browser UI + Next server (MIT)
├── pi-desktop/                     # [submodule] Electron desktop shell (Windows / macOS / Linux)
├── pi-desktop-hos/                 # [submodule] Pi Desktop · Electron on HarmonyOS Edition (HAP)
├── pi-desktop-ark/                 # [submodule] Pi Desktop · HarmonyOS Edition (ArkWeb + in-process Node, HAP)
├── pi-desktop-android/             # [submodule] Pi Desktop · Android Edition (system WebView + in-process Node, APK)
├── pi-desktop-website/             # [submodule] product website of the four editions (static, two pages)
├── pi-agent-hub/                   # [submodule] pi-agent-hub — multi-agent session hub (aggregate / view / deliver)
├── harmonypc-electron/             # [submodule] Electron-on-HarmonyOS runtime (Electron 37.2.3 / Node 22.17.0)
├── assets/                         # (plain dir) single source of brand material: logo.* + screenshots/
├── harmonypc-electron-versions/    # (plain dir, not in git) runtime release archives + native-module guides
└── LICENSE · README.md · README_zh.md
```

### Roles at a glance

| Directory | Role | Consumed by |
|---|---|---|
| `pi` | Upstream agent harness (host + SDK) | all wrappers (source reference and upstream pin baseline) |
| `pi-web` | Upstream browser UI + Next server | all wrappers (its production build is collected at build time) |
| `pi-desktop` | Electron desktop shell (Windows / macOS / Linux) | — |
| `pi-desktop-hos` | **Pi Desktop · Electron on HarmonyOS Edition** (HAP) | — |
| `pi-desktop-ark` | **Pi Desktop · HarmonyOS Edition** (HAP) | — |
| `pi-desktop-android` | **Pi Desktop · Android Edition** (signed APK; system WebView + Maven Central `node-runtime`) | — |
| `@fellow99/node-bridge` + `@fellow99/node-runtime` (ohpm) | In-process Node runtime packages (source: `ohos-node-bridge`) | `pi-desktop-ark` |
| `aosp-node-bridge` + Maven `node-runtime` | In-process Node runtime for Android (`dlopen libnode.so`) | `pi-desktop-android` |
| `pi-desktop-website` | Product website of the four editions | — |
| `pi-agent-hub` | **Multi-agent session hub**: aggregation + read-only viewer + governed prompt delivery | — (independent service; not part of the editions' data plane) |
| `harmonypc-electron` | Electron-on-HarmonyOS runtime (native SOs + ArkTS shell) | `pi-desktop-hos` only |
| `assets` | **The single source of brand material**: app icons / window icons / status-bar glyphs / page logo for the wrappers, plus the website's logo and screenshots | all wrappers + the website |
| `harmonypc-electron-versions` | Runtime release archives (v34 / v37) + material for rebuilding native modules | harmony tooling |

### The two plain directories (not submodules)

- **`assets/`** — brand material: `logo.svg`, `logo.png`, `logo-1024-with-bg.png`, `logo-200-with-bg.png`, `logo.ico`,
  `logo.icns`, plus `screenshots/` (`pc/` and `hos/`; `pc/` currently holds `1.png`–`6.png`, `hos/` is still empty ⬜).
  It is not shipped with any wrapper; instead the wrappers' brand-resource scripts and the website read it as their
  **only source of material**, which is why it has to stay a sibling of all of them.
- **`harmonypc-electron-versions/`** — the HarmonyOS runtime release archives and native-module build guides:
  `v34.8.5-*.zip`, `v37.2.3-*.zip`, `electron144_ohos_hap/`, `libelectron/`, `better-sqlite3编译指导（Electron37）/`,
  and two archives of Node-header (Headers) usage guides for Electron 34 / 37. Use them when you need a runtime version
  other than the pinned submodule, or when rebuilding a native module (such as `better-sqlite3`) for the HarmonyOS side.
  The directory is **not in version control** (`.gitignore` carries a `/harmonypc-electron-versions` entry); `assets/`
  is likewise currently untracked by git.

## How the pieces fit

Every build script of the wrappers derives upstream and material paths from "the parent of this project's root",
and there is **no fallback** to an installed location:

| Sibling path needed | File-level evidence |
|---|---|
| `../pi-web` | `pi-desktop/scripts/prepare-next.mjs`, `collect-pi.mjs`, `smoke-desktop.mjs` (`REPO_PARENT = resolve(ROOT, '..')`, then `join(REPO_PARENT, 'pi-web')`); `pi-desktop-hos/scripts/prepare-next.mjs`, `collect-pi.mjs`, `smoke-node22.mjs` |
| `../pi` | `pi-desktop/scripts/pin-upstream.mjs` (writes the commit / describe / branch of `../pi` and `../pi-web` into `upstream.lock.json`); `pi-desktop-hos/scripts/prepare-next.mjs` (hard existence check: `pi-web`'s `@earendil-works/*` resolves through the sibling `../pi`); `pi-desktop/.github/workflows/ci.yml` |
| `../assets` | `pi-desktop/scripts/brand-assets.mjs` and `pi-desktop-hos/scripts/brand-assets.mjs` (the material directory defaults to the parent project's `../assets`, falling back to the in-repo mirror `resources/brand/` only when it is absent) |
| `../harmonypc-electron` | `pi-desktop-hos/scripts/collect-runtime.mjs` (copies the runtime shell from the sibling `../harmonypc-electron/ohos_hap`) |
| `../harmonypc-electron-versions` | `pi-desktop-hos/scripts/collect-runtime.mjs` (that stage unpacks the v37 archive from the same-named directory for native SOs and Chromium resources) |

These projects therefore **must** live under the same parent directory, with unchanged directory names: cloning a single
wrapper elsewhere, or moving upstream somewhere else, breaks the build outright (`pi-desktop-hos/scripts/prepare-next.mjs`
`fail`s when `../pi-web` / `../pi` is missing; `pi-desktop/scripts/brand-assets.mjs` fails just as hard when neither the
parent material nor the in-repo mirror exists). CI works on the same assumption: the file header of
`pi-desktop/.github/workflows/ci.yml` states that "CI only checks out this repository, while the build scripts resolve
upstream sources at `../pi` and `../pi-web`", so it `git clone`s both into siblings of `pi-desktop`.

**The data plane is shared**: all four wrappers take the same shape — an in-process Next server from `pi-web`, with
the view loading it same-origin. On the Electron shells (`pi-desktop`, and `pi-desktop-hos` on the Electron-on-HarmonyOS
runtime) the agent and `/api/*` live inside the Electron main process; on the HarmonyOS Edition they live inside the
HarmonyOS app process itself (ArkWeb + embedded Node); on the Android Edition they live inside the Android app process
(system WebView + embedded Node via `dlopen`). Either way there is no CORS, no custom protocol, no login page, and no
upstream source fork. Differences are confined to the runtime and platform integration (runtime version,
packaging form, network binding, platform capability gaps) and are documented by each wrapper's own README and
`docs/工程规划.md`; the HarmonyOS Edition's runtime contract lives in [`ohos-node-bridge` `docs/000-工程规划.md`](https://github.com/fellow99/ohos-node-bridge/blob/main/docs/000-工程规划.md) §6.

## pi-agent-hub (companion project)

`pi-agent-hub` is an **independent project** living in this container — not another desktop edition, and not part of the
shared data plane above. Every agent (`pi`, `opencode`, `deepseek-harness`) keeps its own session archive on the
machine that runs it; `pi-agent-hub` brings all of them into one place so you can **read them in one viewer** and
**send a prompt back into a chosen session**:

- **Per-machine `sync-daemon` + plugins** — one Node child process per agent adapter reads that agent's own archive
  (direct storage → agent API → CLI), normalizes it to **pi session format v3** entries, and pushes
  `manifest → timestamp delta → REST sync` to the hub. A deterministic `hubSessionId` per
  `(host, agent, native session)` keeps repeated collection idempotent; large payloads travel through a gzip upload
  stream instead of the JSON body.
- **`hub-service`** — Express 5 + `node:sqlite`: merges into a pi v3 JSONL mirror (source of truth) with a rebuildable
  SQLite index, serves the read-only session API, keeps the SSE downlink open and hosts the console build.
- **`hub-console`** — a self-built Vue 3 SPA (project → `host|agent` → session tree, transcript with tool calls /
  images / sub-agents, project directory browsing, chat composer). **`pi-web` is not used**: the viewer is implemented
  in this project.
- **Governed downlink** — SSE carries `deliver` (text + images + model + thinking level), read-only host directory
  browsing (`fs_request`) and model-catalog requests to the machine that owns the session; the agent's **own**
  interface performs the write (pi RPC / pi-web HTTP / opencode HTTP or CLI / dsh RPC) — never by rewriting the archive.
- **`sync-daemon-tray`** — a Tauri tray (Windows / macOS) that supervises the daemon and edits `sync-daemon.json`;
  Linux uses `pm2` or a user-level systemd unit.

Deployment is one compose file: `docker compose -f pi-agent-hub/deploy/docker-compose.yml up -d`
(`hub-service` + `hub-console` + an nginx reverse proxy, SSE buffering disabled).

Current state (**v0.1.2**, MIT, Node ≥ 22.19, 12-package pnpm workspace): session aggregation, the read-only viewer,
prompt delivery, project-directory browsing and model info are implemented end to end; one-shot session import still
answers `501`, search is a title/cwd substring match only, and retention GC / audit / SSE replay are open.
Details: [`pi-agent-hub/README.md`](./pi-agent-hub/README.md) (English) ·
[`pi-agent-hub/README_zh.md`](./pi-agent-hub/README_zh.md) (Chinese) ·
[`pi-agent-hub/AGENTS.md`](./pi-agent-hub/AGENTS.md) (agent working rules).

## Submodules

All entries are recorded in [`.gitmodules`](.gitmodules):

| Path | Remote |
|---|---|
| `pi` | https://github.com/earendil-works/pi.git |
| `pi-web` | https://github.com/agegr/pi-web.git |
| `pi-desktop` | https://github.com/fellow99/pi-desktop.git |
| `pi-desktop-hos` | https://github.com/fellow99/pi-desktop-hos.git |
| `pi-desktop-ark` | https://github.com/fellow99/pi-desktop-ark.git |
| `pi-desktop-android` | https://github.com/fellow99/pi-desktop-android.git |
| `pi-desktop-website` | https://github.com/fellow99/pi-desktop-website.git |
| `pi-agent-hub` | https://github.com/fellow99/pi-agent-hub.git |
| `harmonypc-electron` | https://github.com/fellow99/harmonypc-electron.git |

```bash
git clone --recursive https://github.com/fellow99/pi-workspace.git
cd pi-workspace
git submodule update --init --recursive   # fetch/refresh all pinned submodules
```

> `assets/` and `harmonypc-electron-versions/` are not in version control and will not show up after cloning; prepare
> them yourself per "The two plain directories" above when you need to build brand resources or a HarmonyOS runtime
> archive (without the former, the desktop wrapper falls back to the in-repo mirror `resources/brand/`; the harmony
> wrapper's `collect-runtime` needs the v37 archive from `../harmonypc-electron-versions`).

To advance the pinned commits, run `git submodule update --remote` and commit the new pointers.

## Status

| Item | Status |
|---|---|
| Upstream baseline | `pi` **v1.6.0** + `pi-web` **v0.18.0** (tags `v1.6.0`/`v0.18.0`) consumed by all four editions; pinned commits recorded per wrapper (`upstream.lock.json` / `runtime.lock.json`) |
| `pi-desktop` | Version **0.18.0**; packaged build on Windows this round with `smoke-desktop` **10 PASS**; `probe-renderer`, packaged e2e, install-launch-uninstall and macOS / Linux **re-verification pending on the test machine** (0.17.0 round measured host / first screen / artifact bootstrap / branding / terminal PTY / native notifications end to end, see `docs/构建验证记录.md`); tag `v0.18.0` |
| `pi-desktop-hos` · **Electron on HarmonyOS Edition** | Version **0.18.0** / `versionCode` **1800**; **built this round as an unsigned HAP — signing and device re-verification pending on the test machine**. The 0.17.0 round was verified on a real HarmonyOS device (new bundle name, first on-device round): boot / render / version badge / DeepSeek key / sandbox workspace / text chat / image-attachment pipeline (bytes verified in session) / **git extension local lifecycle** (init/add/commit/log) / in-process session export / cookie web-auth; see `docs/真机验证记录.md` round 9; tag `v0.18.0` |
| `pi-desktop-ark` · **HarmonyOS Edition** | Version **0.18.0** / `versionCode` **1800**; **built this round as an unsigned HAP — device re-verification pending on the test machine**. The 0.17.0 round was verified on a real HarmonyOS device (new bundle name, first on-device round): boot / render / version badge / DeepSeek key / sandbox workspace / text chat / image-attachment pipeline / **git extension local lifecycle** / in-process session export / device-side suites `model 2/2`, `204-unit 26/0`, `204-e2e 17/0`, `108-fetch 14/0`, `108-photon 22/0`; see `docs/004-真机验证记录-0.17.0.md`; tag `v0.18.0` |
| `pi-desktop-android` · **Android Edition** | Version **0.18.0** / `versionCode` **1800**; in-process Node **26.13.0** (small-ICU) via Maven Central `node-runtime`, system WebView, 6 idempotent patches (`patches/v0.18.0/`); tag `v0.18.0`; debug and release APK built this round, device re-verification pending on the test machine (0.15.0 verified on device) |
| Runtime packages | HarmonyOS: `@fellow99/node-bridge` **0.1.0** + `@fellow99/node-runtime` **24.2.0** (ohpm); Android: `aosp-node-bridge` **0.2.0** + `io.github.fellow99:node-runtime` **26.13.0** (Maven Central) |
| Upstream changes | **1** patch in the desktop shell · **3** registered patches in the Electron on HarmonyOS Edition · **6** idempotent patches in the HarmonyOS Edition · **6** in the Android Edition — no upstream source is forked |
| `pi-desktop-website` | Static two-page site (`index.html` / `index_zh.html`, no framework, zero external requests), deployed to GitHub Pages via GitHub Actions |
| `pi-agent-hub` · **companion** | Version **0.1.2**; multi-agent session hub (hub-service + hub-console + sync-daemon + 5 agent plugins + Tauri tray); aggregation / read-only viewer / prompt delivery / project browsing / model info implemented end to end; import (`501`), full-text search, retention GC, audit and SSE replay still open |
| This repo | Bilingual README (`README.md` English / `README_zh.md` Chinese) + MIT `LICENSE`; `assets/` and `harmonypc-electron-versions/` are local plain directories, untracked |

The authoritative word on capabilities and verification state is each product repo's own documentation — this file is
only an overview and may lag behind it.

## Capability matrix

A simplified, status-icon view across the four editions. Legend: ✅ implemented **and** real-device verified ·
🔧 implemented / shipped but **no real-device record** yet · ⚠️ limited or partial · ❌ unavailable / by design not done ·
⬜ not implemented / unverified · ➖ not applicable. The full, source-verified matrix with per-item evidence lives in
[`docs/能力对照清单.md`](./docs/能力对照清单.md).

| Capability | Desktop | Android | HOS | Ark |
|---|---|---|---|---|
| Streaming chat / SSE | ✅ | ✅ | ✅ | ✅ |
| Models / provider / auth UI | ✅ | ✅ | ✅ | ✅ |
| Session persistence & restore | ✅ | ✅ | ✅ | ✅ |
| Session export | ✅ | 🔧 | ⚠️ | 🔧 |
| Image attachments / vision | ✅ | ✅ | ✅ | ✅ |
| `read` / `write` / `edit` / `ls` | ✅ | ✅ | ✅ | ✅ |
| `bash` (replacement shell) | ✅ | ✅ | ✅ | ✅ |
| `find` / `grep` (pure-JS replacement) | ✅ | ✅ | ✅ | ✅ |
| Git (isomorphic-git extension) | ✅ | ✅ | 🔧 | 🔧 |
| pnpm (in-process vendored) | ✅ | ✅ | ❌ | ❌ |
| Terminal / PTY panel | ✅ | ✅ | ❌ | ❌ |
| codemode (JS sandbox) | ✅ | ⬜ | ⚠️ | ❌ |
| MCP stdio (real server) | ✅ | ⚠️ | ⚠️ | ⚠️ |
| Directory picker | ✅ | ✅ | ✅ | 🔧 |
| User-directory access | ✅ | ⚠️ | ⚠️ | 🔧 |
| Native notifications | ✅ | 🔧 | ❌ | 🔧 |
| Background keep-alive / foreground service | ❌ | ✅ | ❌ | ❌ |
| Update check (semi-automatic) | ✅ | ❌ | ❌ | ❌ |
| System tray | ✅ | ➖ | ❌ | ❌ |
| External links → system browser | ✅ | ✅ | ✅ | ✅ |
| Project trust gate | ✅ | ✅ | ✅ | ✅ |
| Patch count vs. pi-web v0.18.0 | 1 | 6 | 3 | 6 |

## License

This repository: [MIT](./LICENSE) © 2026 fellow99.

Each subproject keeps its own license and this repository does not change them: `pi` is MIT (`earendil-works/pi`,
© 2025 Mario Zechner), `pi-web` is MIT (`agegr/pi-web`, © 2026 agegr), `pi-agent-hub` is MIT
(`fellow99/pi-agent-hub`, © 2026 fellow99), and the wrappers, the website and the runtimes
each carry their own in-repo `LICENSE` (MIT). When reusing content from a subdirectory, that directory's `LICENSE`
governs.

## Unofficial notice

The desktop wrappers aggregated by this workspace are **third-party projects**: they are not products of the Pi or
Pi Web teams and have **no affiliation with, and no endorsement from, the upstream authors**. `Pi` (MIT,
earendil-works/pi) and `Pi Web` (MIT, agegr/pi-web) are upstream open-source projects owned by their respective authors;
the wrappers merely **assemble** them — no fork, no upstream source changes. `harmonypc-electron` belongs to its own
authors; this repository only references it as a sibling.
