# Nova libass patch bundle

This bundle lets you build and iterate on the libass subtitle work without sharing GitHub credentials.

## What is included

- `patches/aos-avos-libass-renderer.patch`: native `aos-avos` changes that add the optional libass renderer path for embedded and external ASS/SSA subtitles.
- `patches/aos-AVP-remote-build.patch`: parent `aos-AVP` changes that add this README and a GitHub Actions workflow.
- `.github/workflows/libass-patch-build.yml`: a workflow that checks out your patched `aos-avos` fork, builds Android libass prebuilts with vcpkg, then builds self-signed Nova APKs.

## One-time GitHub setup

1. Fork `https://github.com/nova-video-player/aos-AVP`.
2. Fork `https://github.com/nova-video-player/aos-avos`.
3. Keeping the fork names as `aos-AVP` and `aos-avos` makes the workflow defaults work automatically.

If your `aos-avos` fork has a different owner/name, enter it in the workflow's `avos_repo` input, for example `yourname/my-aos-avos`.

## Apply and push the native patch

From a folder that contains the generated `patches/` directory:

```bash
git clone https://github.com/YOUR_NAME/aos-avos.git
cd aos-avos
git switch -c libass-subtitles origin/ffmpeg_n8_0
git apply ../patches/aos-avos-libass-renderer.patch
git status --short
git add Include Source jni codecs.mk common.mk
git commit -m "Add optional libass subtitle renderer"
git push -u origin libass-subtitles
```

## Apply and push the parent workflow patch

```bash
git clone https://github.com/YOUR_NAME/aos-AVP.git
cd aos-AVP
git switch -c libass-subtitles origin/nova
git apply ../patches/aos-AVP-remote-build.patch
git status --short
git add .github/workflows/libass-patch-build.yml core.mk LIBASS_PATCH_BUNDLE_README.md
git commit -m "Add remote build workflow for libass subtitle patch"
git push -u origin libass-subtitles
```

## Run GitHub Actions

1. Open your `aos-AVP` fork on GitHub.
2. Go to **Actions**.
3. Select **Libass Patch Build**.
4. Click **Run workflow**.
5. Use:
   - `avos_ref`: `libass-subtitles`
   - `avos_repo`: leave blank if your fork is `YOUR_NAME/aos-avos`
   - `gradle_task`: `assembleNoamazonRelease`
   - `enable_libass`: `true`

The workflow also runs automatically when you push to the `libass-subtitles` branch. If the build succeeds, download the `nova-libass-patch-build` artifact and use the `signed-apks` files for sideload testing.

## How to judge the CI result

The workflow writes `build-inputs.log`, `libass-diagnostics.log`, and `gradle-build.log` into the artifact. A correct libass-enabled build must show:

- `gradle-build.log`: `-DCONFIG_LIBASS` and `subtitle_libass.c`.
- `libass-diagnostics.log`: `libavos.so` has `DT_NEEDED` for `libass.so`.
- `libass-diagnostics.log`: every ABI has `libass.so`, `libfreetype.so`, `libfribidi.so`, and `libharfbuzz.so`.
- `libass-diagnostics.log`: every generated APK contains those libraries under `lib/$ABI/`.

If any of those checks fail, the workflow exits with an error instead of producing a misleading green build.

## Current technical status

- Embedded `AV_CODEC_ID_ASS` and `AV_CODEC_ID_SSA` packets are preserved and routed to libass when `CONFIG_LIBASS` is enabled.
- External `.ass` and `.ssa` sidecar files are routed through libass when `CONFIG_LIBASS` is enabled.
- The Nova video rendering path is intentionally untouched.
- The feature is still gated behind `LIBASS=ON`.
- GitHub Actions builds Android libass, FreeType, FriBidi, HarfBuzz, and their generated shared-library dependencies with vcpkg for each Nova ABI.
- The parent makefile copies those libass/vcpkg `.so` files into `MediaLib/libs/$ABI` so Gradle packages them into the split APKs.

## Expected next iteration

Run the workflow with `enable_libass: true`. If it fails, paste the uploaded `gradle-build.log` or the failing Actions section back into Codex so the native link/package issue can be tightened.
