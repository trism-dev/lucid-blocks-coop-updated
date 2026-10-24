# Verification evidence

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
