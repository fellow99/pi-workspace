[中文](./README_zh.md) | English

---

# Pi Desktop Workspace

> A single-repo workspace aggregating every project required to ship [Pi](https://github.com/earendil-works/pi) (an
> extensible agent harness) and its browser UI [Pi Web](https://github.com/agegr/pi-web) as desktop apps on four
> platforms, in **three** product forms:
> **Pi Desktop** (Electron shell · Windows / macOS / Linux) ·
> **Pi Desktop · Electron on HarmonyOS Edition** (`pi-desktop-hos`) ·
> **Pi Desktop · HarmonyOS Edition** (`pi-desktop-ark`, ArkWeb + a real in-process Node runtime from `ohos-node-bridge`).
> The upstream repositories, the two runtimes and the brand material sit side by side as **git submodules plus plain
> sibling directories**, so the workspace layout reproduces the exact sibling-directory structure each wrapper's build
> scripts expect (`../pi`, `../pi-web`, `../assets`, `../harmonypc-electron`).

---

## What is this

`Pi` is an upstream open-source, extensible agent harness (MIT, [earendil-works/pi](https://github.com/earendil-works/pi))
with four integration entry points — CLI / RPC / JSON / SDK. `Pi Web` is its community browser UI (MIT,
[agegr/pi-web](https://github.com/agegr/pi-web)) where **a single Node process serves both the SPA and `/api/*`**, and
the agent is created inside that process. Three wrapper projects plus one runtime library move this whole stack onto
the desktop:

- **`pi-desktop`** — an Electron desktop shell (Windows / macOS / Linux), Electron **43.7.9**: the main process hosts
  `pi-web`'s Next server (agent and `/api/*` included) **in-process**, and the renderer loads the official Pi Web UI
  from that same origin.
- **`pi-desktop-hos`** — **Pi Desktop · Electron on HarmonyOS Edition**: a HarmonyOS PC / tablet port built on the
  Electron-on-HarmonyOS runtime from `harmonypc-electron` (Electron **37.2.3** / Node **22.17.0**), packaged as a
  signable, distributable **HAP**.
- **`pi-desktop-ark`** — **Pi Desktop · HarmonyOS Edition**: the same product **without Chromium or Electron** — an
  ArkTS host whose **ArkWeb** view loads the local `pi-web` server, which runs on a **real Node.js runtime embedded in
  the app process** (`ohos-node-bridge`: `dlopen` + `node::Start`). Smaller payload, system WebView, no Electron
  runtime; functional target is parity with the Electron on HarmonyOS Edition.
- **`ohos-node-bridge`** — **the runtime library behind the HarmonyOS Edition** (`@fellow99/node-bridge`: an
  ohpm-publishable ArkTS HAR + NAPI loader). It is **not** an app and **not** an edition — it is the piece that lets any
  HarmonyOS application host Node in-process. Its only registered consumer is `pi-desktop-ark`.

The wrappers only **assemble**: the UI is reused 100% from upstream and no upstream source is forked (see
`pi-desktop/docs/工程规划.md` §1.1–§1.2, `pi-desktop-hos/docs/工程规划.md` §1.1–§1.2,
`pi-desktop-ark/docs/000-工程规划.md`). Patch policy is counted **per edition**: the desktop shell ships **0** patches,
the Electron on HarmonyOS Edition **2** registered patches, and the HarmonyOS Edition applies its `pi-web` patches as
idempotent, reversible files (`pi-desktop-ark/patches/v0.12.1/`, **5** at v0.12.1).

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
├── ohos-node-bridge/               # [submodule] Node runtime library embedded by the HarmonyOS Edition
├── pi-desktop-website/             # [submodule] product website of the three wrappers (static, two pages)
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
| `ohos-node-bridge` | In-process Node runtime library (`@fellow99/node-bridge`) | `pi-desktop-ark` |
| `pi-desktop-website` | Product website of the three wrappers | — |
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

**The data plane is shared**: all three wrappers take the same shape — an in-process Next server from `pi-web`, with
the view loading it same-origin. On the Electron shells (`pi-desktop`, and `pi-desktop-hos` on the Electron-on-HarmonyOS
runtime) the agent and `/api/*` live inside the Electron main process; on the HarmonyOS Edition they live inside the
HarmonyOS app process itself (ArkWeb + embedded Node). Either way there is no CORS, no custom protocol, no login page,
and zero upstream source changes. Differences are confined to the runtime and platform integration (runtime version,
packaging form, network binding, platform capability gaps) and are documented by each wrapper's own README and
`docs/工程规划.md`; the HarmonyOS Edition's runtime contract lives in `ohos-node-bridge/docs/000-工程规划.md` §6.

## Submodules

All entries are recorded in [`.gitmodules`](.gitmodules):

| Path | Remote |
|---|---|
| `pi` | https://github.com/earendil-works/pi.git |
| `pi-web` | https://github.com/agegr/pi-web.git |
| `pi-desktop` | https://github.com/fellow99/pi-desktop.git |
| `pi-desktop-hos` | https://github.com/fellow99/pi-desktop-hos.git |
| `pi-desktop-ark` | https://github.com/fellow99/pi-desktop-ark.git |
| `ohos-node-bridge` | https://github.com/fellow99/ohos-node-bridge.git |
| `pi-desktop-website` | https://github.com/fellow99/pi-desktop-website.git |
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
| Upstream baseline | `pi` **v1.1.0** (`Branch_v1.1.0`) + `pi-web` **v0.11.1** (`Branch_v0.11.1`); pinned commits recorded in `pi-desktop/upstream.lock.json` |
| `pi-desktop` | Version **0.11.1**; packaged-state end-to-end measured on Windows (in-process host / first screen / artifact bootstrap / branding / terminal PTY / native notifications / install-launch-uninstall ✅, see `docs/构建验证记录.md` rounds 1–2); **macOS / Linux still unverified**; visual presentation of the system tray ⬜ not yet separately verified (`docs/桌面能力清单.md`) |
| `pi-desktop-hos` · **Electron on HarmonyOS Edition** | Version **0.11.1** / `versionCode` **1101**; **built, signed and verified on device** (HarmonyOS 6.1.0.135 / API 24, see `docs/真机验证记录.md`); capability boundaries in `docs/鸿蒙环境能力清单-v0.11.1.md` |
| `pi-desktop-ark` · **HarmonyOS Edition** | Skeleton + Phase 1 capabilities on device: in-process Node `v24.2.0`, in-process Next server, ArkWeb UI, file-mailbox host bridge, permissions/pickers/notifications; real-device verified on HUAWEI MateBook Pro (`HAD-W24`, API 26). Capability record: `docs/001-能力复刻评估.md` |
| `ohos-node-bridge` | Runtime library `@fellow99/node-bridge` **0.1.0**: `dlopen` + `node::Start` bootstrap, host bridge, payload bootstrap/entry resolver, ELF+SHA-256 provisioning gates, self-check endpoints; consumed by `pi-desktop-ark` |
| Upstream changes | **0** required patches in the desktop shell, **2** registered patches in the Electron on HarmonyOS Edition, **5** idempotent patch files in the HarmonyOS Edition (`pi-desktop-ark/patches/v0.12.1/`) — no upstream source is forked |
| `pi-desktop-website` | Static two-page site (`index.html` / `index_zh.html`, no framework, zero external requests), deployed to GitHub Pages via GitHub Actions |
| This repo | Bilingual README (`README_zh.md` original / `README.md` translation) + MIT `LICENSE`; `assets/` and `harmonypc-electron-versions/` are local plain directories, untracked |
| Open ⬜ | no screenshots yet in `assets/screenshots/hos/`; packaged-state verification on macOS / Linux; separate verification of the tray and its high-DPI / light-dark presentation |

The authoritative word on capabilities and verification state is each product repo's own documentation — this file is
only an overview and may lag behind it.

## License

This repository: [MIT](./LICENSE) © 2026 fellow99.

Each subproject keeps its own license and this repository does not change them: `pi` is MIT (`earendil-works/pi`,
© 2025 Mario Zechner), `pi-web` is MIT (`agegr/pi-web`, © 2026 agegr), and the wrappers, the website and the runtimes
each carry their own in-repo `LICENSE` (MIT). When reusing content from a subdirectory, that directory's `LICENSE`
governs.

## Unofficial notice

The desktop wrappers aggregated by this workspace are **third-party projects**: they are not products of the Pi or
Pi Web teams and have **no affiliation with, and no endorsement from, the upstream authors**. `Pi` (MIT,
earendil-works/pi) and `Pi Web` (MIT, agegr/pi-web) are upstream open-source projects owned by their respective authors;
the wrappers merely **assemble** them — no fork, no upstream source changes. `harmonypc-electron` belongs to its own
authors; this repository only references it as a sibling.
