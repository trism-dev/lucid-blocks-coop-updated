# Lucid Blocks co-op updated: state
Read this first. Written 2026-10-24 from `playtest.md`, `tasks.md` and `process-log.md`; update it at the end of every session.

- **Engine / repo:** Godot 4.6.0 (double precision), GodotSteam 4.17.1, mod of an installed Steam game. Public fork `trism-dev/lucid-blocks-coop-updated` of `parkers0405/lucid-blocks-coop` (public on purpose: attribution and upstream history; the "private repo per game" rule does not apply). Branch `fix/current-build-compatibility`; feature branches only, never force, never push `main`.
- **Target:** Steam app 3495730, build 25451214, executable 5.1.0.0. Fingerprints and the user's authorization are in `authorization.md`. Recovered game files stay in ignored `work/` and are never committed.
- **Live build:** none released. `dist/lucid-blocks-coop.pck` is a local build; nothing is published.
- **What exists:** attributed fork; Windows build/install/rollback and raw mod-only packaging (42 fixture assertions reported under both PowerShell versions); restored current-game interfaces; committed startup checkpoint f10f19f.
- **Status:** ACTIVE: user confirmed adoption and authorized one-pass resumption plus discretionary commits. Current task: durable private-region checkpoint implementation; preserve unrelated .import changes.
- **Last verified:** post-move rerun 2026-10-24: 59 admission assertions, 4 native-guard assertions, and 61 script startup checks passed. Logs: work/build-logs/3c4219d3b78941089aea82b1b1f69113. Artifact SHA256 D9AB22FBB5B0F9C52F6D7A9F97933925799F951E47B608E49DD1AA0645DF9FEA. LAN admission tests require working loopback outside the sandbox; these are not real-world gameplay/Steam/save tests.
- **Open items (user steps first):** two authenticated Steam accounts for join/invite/reconnect checks; decide when a release or PR is wanted (none created).
- **Next step:** implement validated host-side private-region checkpoints and durable flush acknowledgments; then independent exploration/dimension and gameplay checks.
- **Where things are:** `design.md`, `architecture.md`, `tasks.md`, `playtest.md` (evidence), `process-log.md`, `authorization.md`, `mod/`, `native_patch/`, `scripts/`, `tests/`, `dist/`, `work/` (ignored). Engine rules: `..\guide.md`; Godot is not used with a `rojo`-style sync, so the Roblox rules do not apply.
