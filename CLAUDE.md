# ProsperoStore

## Git

- Work on `main`, no branches or pull requests. Commit there, but do not push unless the user asks; suggest the push when a change is ready.
- Never add Claude as a collaborator: no `Co-Authored-By` trailer, no "Generated with Claude" line, in commits, tags, or PR text.

## Verifying a change

- Do not build or run anything on the PC to check a change: no host renderer (`tools/store-host.sh`), no host tests (`tools/store-test.sh`, `make test`), no compiling a piece of the app with the host compiler. The CI runs the host tests.
- The app is verified on the PS5. Build it for the console with `make app`, or with `make deploy` when asked (`PS5_HOST` is the console's address), and then wait for the user to try it and say what they see. Do not call a change done on the strength of a PC check.
- Layout is worked out from the code's own numbers (positions, sizes, the fonts' measures), not by rendering it on the PC.

## Lint

The CI runs `LINT_SKIP_TIDY=1 make lint` (`tools/lint.sh`). Run `LINT_SKIP_TIDY=1 bash tools/lint.sh` before every commit; it must exit 0. A failed lint also fails the build that publishes a release.

- **Formatting:** clang-format 18 with the repo's `.clang-format`, over `src/` (except `src/third_party`), `tooling/native`, `tests/` and `examples/`. Run `bash tools/run_clang_format.sh` to fix, `--check` to verify. Do it after every edit, including the lines of existing files you touch.
- **Header on every source file:** the first 20 lines of each `.c .cc .cpp .h .hpp .ld .py .ps1 .sh .yml .yaml`, `Makefile`, `.clang-format`, `.clang-tidy` and `.env.example` must contain both `Copyright (C) 2026 BlackBearReloaded` and `SPDX-License-Identifier: GPL-3.0-or-later`. New files need them. `third_party/` and `src/third_party/` are exempt.
- **No C# files:** `.cs` and `.csproj` fail the lint.
- **`param.json`:** `titleId` is `PPSA` plus five digits; `contentVersion` is `NN.NNN.NNN`; `masterVersion` is `NN.NN`; `downloadDataSize` is a non-negative integer; `titleName` of the default language is not empty.
- **No local paths or personal names:** a Windows user path, a WSL `/mnt/c/Users/...` path, a `/home/<name>` path of the original author or that author's name in any tracked file fails the lint. The exact patterns are in `tools/lint.sh`; do not copy them into other files, since the check would match them there. Use placeholders such as `<user>` in examples.
- **Whitespace:** `git diff --check` must be clean (no trailing whitespace).
- **Shell scripts:** every `tools/*.sh` must pass `bash -n`. `tools/validate-assets.sh` also runs.
- **Static analysis:** `tools/run_clang_tidy.sh` runs only without `LINT_SKIP_TIDY=1`. The CI skips it.

## Version

A release tag is `vN.NNN.NNN` and must match `contentVersion` in `sce_sys/param.json` (`01.000.100` is tag `v1.000.100`); the workflow refuses a mismatch. A version bump changes the same five files as the `Version 01.000.090` commit: `sce_sys/param.json`, `src/platform/ps5/store_http.cpp`, `src/diag/diagnostics.cpp`, `src/net/curl_request.cpp` and `.github/ISSUE_TEMPLATE/bug_report.yml`. Tags here are lightweight.

## Releasing by hand

The `Build` workflow (`tooling.yml`) is disabled, so a `v` tag publishes nothing by itself. When the user wants to make a release on GitHub, suggest `make release-zip` first: it runs `make app` and then `tools/store-zip-modes.py`, which stores every entry of `dist/PPSA99000.zip` as 0777. Without that the console refuses to start the app (CE-107750-0) when a tool keeps the archive's permissions; a plain `make app` or `make deploy` leaves the zip at 0644. Then: `SHA256SUMS` from that zip, a lightweight tag on the version commit, and `gh release create` with the zip and `SHA256SUMS`. The release notes (`--notes`) are written in English and list the improvements since the previous release, taken from the commits between the previous `v` tag and the new one (`git log <previous tag>..HEAD`); never just the version number. Only suggest the steps; each one that reaches GitHub waits for the user's yes.

When the user says "make release" (or "hacer un release"), they mean `make release-zip`: run that, not a plain `make app`.

## After a release

When the CI of a `v` tag succeeds and the GitHub release is published, suggest launching the catalog's **Release updates** workflow: `gh workflow run "Release updates"` in `darkxex/ps5-homebrew-catalog`. The store finds its updates from the catalog record `apps/PPSA99000.json`, not from GitHub, so installed stores see the new version only after that workflow's bump pull request is merged and the site is rebuilt. It also runs by itself daily at 07:37 UTC. Only suggest it: launching it acts on a public repository, so wait for the user's yes.
