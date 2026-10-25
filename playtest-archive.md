# Archived playtest evidence
## 2026-10-23: baseline inventory
- Steam manifest: app 3495730, build 25451214, depot manifest 7072406399823411834.
- Executable metadata: 5.1.0.0; SHA-256 fingerprints recorded in authorization.md.
- Original installation contained the executable and three DLLs; no mods folder.
- Fork created at https://github.com/trism-dev/lucid-blocks-coop-updated; local branch fix/current-build-compatibility.
- GDRETools v2.7.4 downloaded from the official GitHub release. Help verified using an isolated APPDATA profile under work/.
- Gameplay, multiplayer, save persistence, Steam invites, and rollback: NOT YET TESTED.


## 2026-10-24: resumed verification
- Global execution/progress preferences saved to C:/Users/trist/.codex/AGENTS.md.
- Windows installer fixtures: parent reran tests/test_install.ps1 with Windows PowerShell; 34 assertions passed. Worker also reports 34 under pwsh. Synthetic fixtures do not establish game compatibility.
- Original game pack with matching editor and nested physical DLL layout: runtime probe reports steam_peer=true, world=true, steam=true. Missing Main-node errors are expected for this scene-free interface probe; vanilla gameplay remains unverified.
- Worker verified a two-file raw-source/remap sentinel PCK loads through the original ModLoader. Full mod packaging and script compilation remain pending.
- Checkpoint 1fd78c6 pushed to origin/fix/current-build-compatibility; no PR or release created.

### Current-version startup gate passed
- Candidate: work/production-build-check/lucid-blocks-coop.pck, SHA256 4F25E1293B97179E49305CE817914D8E5CCFF4E197E0B6AF7AAC56AC71A15935.
- Logs: work/build-logs/6ddc9311575d4727a5270b10a99c7a29. Normal original Main startup prints COOP_BUILD_CHECK_OK 61; no ERROR or SCRIPT ERROR lines.
- Assertions: LUCY=8, TORUS=9, Entity.dexterity=0, water-flow method exists and returns zero on unloaded terrain, offline remote-peer check returns false.
- Fixes: restored dimension/generator interfaces and shared water-flow/stat APIs; corrected MultiplayerAPI peer lookup; skip absent debug spawn resources and support current capsule names.
- Base game itself leaks its on-screen keyboard KeyListHandler at shutdown. The unpublished test-only probe frees that exact helper before quitting; production mod does not alter base-game cleanup.
- This establishes startup/interface compatibility only. Multiplayer, persistence, exploration, Steam invitations, and ender storage remain unverified.


## 2026-10-24: pre-pause admission result and layout adoption
- Last run began before the user's pause request and was collected during adoption; no implementation resumed.
- PASS: admission59, native guard4, startup61. Logs: work/build-logs/9ea0857f63304b5894e5dfea7f0c5964/runtime.stdout.log.
- Artifact SHA256: D9AB22FBB5B0F9C52F6D7A9F97933925799F951E47B608E49DD1AA0645DF9FEA. Working code/tests remain uncommitted.
- Original-path evidence predates move to Godot/Lucid Blocks. No post-move runtime test performed during documentation-only adoption; rerun from new directory on approved resume.
- Workflow parse check: PARSE OK for C:/Work/.claude/workflows/game-dev.js. Current STATE.md is 11 nonblank lines; old project path absent, new nested repository present.
- Implementation paused pending user's confirmation and explicit resume. Real Steam invites, gameplay, private persistence and remaining features unverified.

