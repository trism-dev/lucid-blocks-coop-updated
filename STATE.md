# Lucid Blocks co-op updated: state
Read this first. Written 2026-10-24 from `playtest.md`, `tasks.md` and `process-log.md`; update it at the end of every session.

- **Engine / repo:** Godot 4.6.0 (double precision), GodotSteam 4.17.1, mod of an installed Steam game. Public fork `trism-dev/lucid-blocks-coop-updated` of `parkers0405/lucid-blocks-coop` (public on purpose: attribution and upstream history; the "private repo per game" rule does not apply). Branch `fix/current-build-compatibility`; feature branches only, never force, never push `main`.
- **Target:** Steam app 3495730, build 25451214, executable 5.1.0.0. Fingerprints and the user's authorization are in `authorization.md`. Recovered game files stay in ignored `work/` and are never committed.
- **Live build:** none released. `dist/lucid-blocks-coop.pck` is a local build; nothing is published.
- **What exists:** attributed fork; Windows build/install/rollback scripts with `tests/test_install.ps1` (34 assertions passed 2026-10-24); startup gate passed on the current game (`COOP_BUILD_CHECK_OK 61`, no ERROR or SCRIPT ERROR lines); restored dimension/generator, water-flow and stat APIs.
- **In progress:** two-player compatibility negotiation and native-loading guard (`tests/admission.gd`, `tests/native_guard.gd` are untracked).
- **Last verified:** 2026-10-24: startup and interface compatibility only. Multiplayer, persistence, exploration, Steam invites and ender storage are NOT tested.
- **Open items (user steps first):** two authenticated Steam accounts for join/invite/reconnect checks; decide when a release or PR is wanted (none created).
- **Next step:** reject mismatched peers before world sync, cap sessions at two, refuse unvalidated native patches (see `tasks.md`).
- **Where things are:** `design.md`, `architecture.md`, `tasks.md`, `playtest.md` (evidence), `process-log.md`, `authorization.md`, `mod/`, `native_patch/`, `scripts/`, `tests/`, `dist/`, `work/` (ignored). Engine rules: `..\guide.md`; Godot is not used with a `rojo`-style sync, so the Roblox rules do not apply.
