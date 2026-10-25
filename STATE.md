# Lucid Blocks co-op updated: state
Read this first. Written 2026-10-24 from `playtest.md`, `tasks.md` and `process-log.md`; update it at the end of every session.

- **Engine / repo:** Godot 4.6.0 (double precision), GodotSteam 4.17.1, mod of an installed Steam game. Public fork `trism-dev/lucid-blocks-coop-updated` of `parkers0405/lucid-blocks-coop` (public on purpose: attribution and upstream history; the "private repo per game" rule does not apply). Branch `fix/current-build-compatibility`; feature branches only, never force, never push `main`.
- **Target:** Steam app 3495730, build 25451214, executable 5.1.0.0. Fingerprints and the user's authorization are in `authorization.md`. Recovered game files stay in ignored `work/` and are never committed.
- **Live build:** none released. `dist/lucid-blocks-coop.pck` is a local build; nothing is published.
- **What exists:** attributed fork; Windows build/install/rollback and raw mod-only packaging (42 fixture assertions reported under both PowerShell versions); restored current-game interfaces; committed startup checkpoint f10f19f.
- **Status:** BLOCKED at authorized combat verification boundary: rendered cycle50/2 failed on OpenGL shader-instance/compilation errors; target uses Forward Plus. No task process running. Native baseline compiled successfully but is not runtime-validated or installed.
- **Last verified:** 2026-10-24: region28 fixture/native43/startup62 checks passed; logs work/build-logs/3cb5a2c872c84177ba14da97eae12259, SHA ED2B56B51DD598A66FF5EE40617FBAC7AC690D9D0BBE173733895201CAB28F3A. Prior admission59/nativeguard4 passed. These are not end-to-end Steam/gameplay/save-rejoin verification.
- **Open items (user steps first):** two authenticated Steam accounts for join/invite/reconnect checks; decide when a release or PR is wanted (none created).
- **Next step:** renewed bounded authorization to assess Forward+/Vulkan and verify original Gel visibility; then probe compiled native baseline before terrain changes. Safe implementation checkpoint b395d39. Candidate combat code is uncommitted.
- **Where things are:** `design.md`, `architecture.md`, `tasks.md`, `playtest.md` (evidence), `process-log.md`, `authorization.md`, `mod/`, `native_patch/`, `scripts/`, `tests/`, `dist/`, `work/` (ignored). Engine rules: `..\guide.md`; Godot is not used with a `rojo`-style sync, so the Roblox rules do not apply.
