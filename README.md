# PBRP build for Redmi Note 8 Pro (`begonia`)

GitHub Actions recipe that compiles **PBRP** (PitchBlack Recovery Project)
`recovery.img` for the Xiaomi Redmi Note 8 Pro, codename `begonia`.

Everything is derived from the device tree and PBRP sources published by
**SaiKrishna1504**. The CI recipe is adapted from
[`pacman-Sy/pbrp-begonia-build`](https://github.com/pacman-Sy/pbrp-begonia-build).

## What this builds

| | |
|---|---|
| PBRP manifest | `PitchBlackRecoveryProject/manifest_pb` @ `android-12.1` |
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
   - `jobs` — default `4` (the runner has 4 vCPUs)
   - `ccache` — default `true`
   - `publish_release` — attach the image to a GitHub release
3. When the job goes green, download `pbrp-begonia-recovery` from the run's
   **Artifacts** section. `recovery.img` is the file to flash to the
   `recovery` partition.

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