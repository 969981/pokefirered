# Automated upstream sync and verified build releases

This fork tracks [pret/pokefirered](https://github.com/pret/pokefirered) on the upstream `master` branch.

## Schedule and manual triggering

- `.github/workflows/upstream-sync-release.yml` checks upstream every 10 minutes (GitHub Actions cron is best-effort).
- It merges any upstream changes into this fork's `master` without force-pushing or deleting local workflow changes.
- The workflow can be started manually via **Actions → Sync upstream and release verified build metadata → Run workflow**.
- Direct pushes to `master` also run compilation and release.
- A scheduled check with no upstream changes skips the expensive build.
- Sync, compile and publish happen in one workflow because a push made with `GITHUB_TOKEN` does not trigger other workflows.

If a merge conflict occurs, the workflow stops and requires review rather than force-overwriting local changes.

## Compiler and variants

The workflow installs `pret/agbcc`, GNU ARM binutils, GCC ARM and libpng development files.

It compiles and checksum-verifies `COMPARE=1` for both FireRed and LeafGreen, revisions 0, 1 and 10 (Switch). It additionally compiles FireRed revision 0 with `MODERN=1` and `COMPARE=0`.

The original upstream `.github/workflows/build.yml` is left unchanged.

## Release assets

Every successfully compiled commit gets a single `build-<commit>` GitHub Release with:

- `symbols-and-maps.tar.gz`: symbol tables and linker maps;
- `build-sha1.txt` and `build-sha256.txt`: checksums of locally built images;
- `build-report.txt`: source commit, build run and file sizes.

Compiled game `.gba` and `.elf` files are **not publicly attached** to Releases or uploaded as Actions artifacts. This avoids redistributing the complete proprietary game ROM. Developers can reproduce the builds locally using the steps in [INSTALL.md](../INSTALL.md).

The release job runs only after every compilation and checksum comparison succeeds. Existing tags are not replaced.
