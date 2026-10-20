# PBRP build for Redmi Note 8 Pro (`begonia`)

GitHub Actions recipe that compiles **PBRP** (PitchBlack Recovery Project)
`recovery.img` for the Xiaomi Redmi Note 8 Pro, codename `begonia`.

Everything is derived from the device tree and PBRP sources published by
**SaiKrishna1504**. The CI recipe is adapted from
[`pacman-Sy/pbrp-begonia-build`](https://github.com/pacman-Sy/pbrp-begonia-build).

## What this builds

| | |
|---|---|
| PBRP manifest | `PitchBlackRecoveryProject/manifest_pb` @ `android-12.1` (branch tip, not pinned) |
| Recovery source | `PitchBlackRecoveryProject/android_bootable_recovery` @ `android-12.1` |
| PBRP vendor config | `PitchBlackRecoveryProject/vendor_pb` @ `pb-12.1` |
| Device tree | `methylphetamine/device_xiaomi_begonia-pbrp` @ `twrp-12.1` (fork of `SaiKrishna1504/device_xiaomi_begonia-pbrp`) |
| Product / lunch | `pb_begonia` / `pb_begonia-eng` |
| Make target | `recoveryimage` |
| Output | `out/target/product/begonia/recovery.img` |

## Device tree branch selection

`device_xiaomi_begonia-pbrp` has three branches, and only one is PBRP:

| Branch | Product file | Project |
|---|---|---|
| `twrp-12.1` | `pb_begonia.mk` | **PBRP — this is the one we use** |
| `dynamic` | `omni_begonia.mk` | TWRP / OmniROM |
| `fbev2` | `omni_begonia.mk` | TWRP / OmniROM |

## Usage

1. **Actions → PBRP build (begonia) → Run workflow**
2. Options:
   - `makefile` — default `pb_begonia-eng`
   - `build_target` — default `recoveryimage`
   - `manifest_branch` — default `android-12.1`
   - `jobs` — default `4` (the runner has 4 vCPUs)
   - `ccache` — default `true`
   - `publish_release` — attach the image to a GitHub release
3. When the job goes green, download `pbrp-begonia-recovery` from the run's
   **Artifacts** section. `recovery.img` is the file to flash to the
   `recovery` partition.

## Updating PBRP source

`android-12.1` is PBRP's newest maintained branch: `manifest_pb` was last
updated there in September 2025, while `android-14.0` has been untouched
since September 2024 and is not compatible with this 12.1 begonia tree.

Every workflow run does a fresh `repo init` / `repo sync` of the
`manifest_branch` tip, so re-running the workflow is enough to pick up new
PBRP commits. The **Record PBRP source revisions** step prints the exact
commits used by that run. Tips verified on 2026-10-20:

| Project | Commit | Date |
|---|---|---|
| `manifest_pb` | `59ff14c8ad` | 2025-09-11 |
| `android_bootable_recovery` | `c3ee88af57` | 2025-11-20 |
| `vendor_pb` (`pb-12.1`) | `31bf0d4cd6` | 2024-12-08 |
| `android_build` | `27af0ea018` | 2024-01-24 |
| `android_system_core` | `ac4f36c7eb` | 2025-02-22 |

## Publishing

After a successful compile the workflow:

1. uploads `recovery.img` as the `pbrp-begonia-recovery` Actions artifact;
2. uploads it anonymously to Gofile and prints the download page;
3. sends it to Telegram as a document when it is under Telegram's 50 MB bot
   limit, otherwise sends the Gofile link and SHA-256.

Telegram publishing uses two repository secrets:

| Secret | Value |
|---|---|
| `TELEGRAM_BOT_TOKEN` | bot token from @BotFather |
| `TELEGRAM_CHAT_ID` | target chat/channel ID (e.g. `-100...`) |

Gofile needs no credentials. Both uploads are best-effort: a Gofile/Telegram
outage does not fail the build, and the Actions artifact remains available.

## Upstream breakage worked around

`manifest_pb/android-12.1/pbrp-default.xml` declares:

```xml
<project path="vendor/utils" name="vendor_utils"
         remote="PitchBlackRecoveryProject" revision="pb" />
```

The `pb` branch of `PitchBlackRecoveryProject/Utils` **no longer exists** —
`master` now holds an unrelated project, and `git ls-remote` shows only
`master` and `refs/meta/config`. A plain `repo sync` therefore aborts with
revision-not-found.

`local_manifests/begonia.xml` removes that project. It only existed to
symlink `vendor/utils/pb_build.sh` → `vendor/pb/pb_build.sh`, a developer
helper script, and is not needed to compile the image. The same local
manifest also registers the `begonia` device tree so `repo sync` fetches it
along with everything else.

## Notes on the device tree

The tree is self-contained — it carries its own kernel, DTB, DTBO and
keymaster/gatekeeper blobs under `prebuilt/` and `recovery/root/`, so no
kernel build, proprietary vendor blobs, or hardware trees are needed.

`BoardConfig.mk` sets `ALLOW_MISSING_DEPENDENCIES`, `BUILD_BROKEN_DUP_RULES`
and `BUILD_BROKEN_ELF_PREBUILT_PRODUCT_COPY_FILES` to `true` to tolerate the
reduced `remove-minimal.xml` tree, and sets `PLATFORM_SECURITY_PATCH` to
`2099-12-31` to bypass AVB rollback checks.

## Repository layout

```
.github/workflows/build.yml   the CI recipe
local_manifests/begonia.xml   device tree + upstream fix
```