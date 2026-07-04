# Nova libass patch bundle

This bundle lets you build and iterate on the libass subtitle work without sharing GitHub credentials.

## What is included

- `patches/aos-avos-libass-renderer.patch`: native `aos-avos` changes that add the optional libass renderer path for embedded and external ASS/SSA subtitles.
- `patches/aos-AVP-remote-build.patch`: parent `aos-AVP` changes that add this README and a GitHub Actions workflow.
- `.github/workflows/libass-patch-build.yml`: a workflow that checks out your patched `aos-avos` fork while using upstream Nova submodules for everything else.

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
git add Include Source codecs.mk common.mk
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
git add .github/workflows/libass-patch-build.yml LIBASS_PATCH_BUNDLE_README.md
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
   - `enable_libass`: `false` for the first run

The first run proves the patched source still builds with the old subtitle behavior. After Android libass prebuilts/deps are added, run again with `enable_libass: true`.

## Current technical status

- Embedded `AV_CODEC_ID_ASS` and `AV_CODEC_ID_SSA` packets are preserved and routed to libass when `CONFIG_LIBASS` is enabled.
- External `.ass` and `.ssa` sidecar files are routed through libass when `CONFIG_LIBASS` is enabled.
- The Nova video rendering path is intentionally untouched.
- The feature is still gated behind `LIBASS=ON`.
- Android libass prebuilts and dependencies are not included yet.

## Expected next iteration

After the first CI run, paste the failed log or artifact result back into Codex. The next real engineering step is adding Android builds/prebuilts for libass and its dependencies, then enabling `LIBASS=ON` in CI.
