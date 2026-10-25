# Implementation tasks

Status is evidence-based; unchecked items are not shipped features.

## Current task

**PAUSED: authorized combat/fixture repair cycles exhausted.** Round1 passed32 assertions but failed strict offline Steam call; round2 exposed optional-base/required-subclass attack signature mismatch and premature headless softbody sampling. Persistence checkpoint b395d39 passed28 fixture/43 native/startup62 checks.

Latest: rendered verification50/2 failed under OpenGL; target requires Forward Plus. All authorized cycles exhausted; next requires renewed bounded Forward+/Vulkan verification. Native baseline compiled (work/native-baseline-output), but no runtime/installation claim. No task process remains.

## Steps

- [x] Create attributed public fork and branch from pinned upstream commit.
- [x] Record target and authorization in authorization.md.
- [ ] Recover current game locally and inventory override differences. Check: recovery log and compatibility report.
- [ ] Complete architecture.md from actual game interfaces. Check: authority, identity, persistence, and test contracts specified.
- [ ] Add Windows build/install/rollback scripts and regression checks. Check: build leaves install unchanged; round-trip install on disposable fixture.
- [x] Guard native loading and negotiate two-player compatibility. Check: mismatches rejected before world state.
- [ ] Fix current-version scripts and README defects. Check: current game load and per-defect regressions.
- [ ] Verify Steam joining/invites/reconnect. Check: two authenticated accounts and failure scenarios.
- [ ] Implement independent terrain and dimension authority. Check: separated players and dimension transitions.
- [ ] Persist private regions with acknowledged checkpoints. Check: restart, disconnect, visits, and migration.
- [ ] Implement bounded personal ender chest. Check: isolation, duplication prevention, restart, and access-block break.
- [ ] One-hour multiplayer soak and rollback; independent review. Check: dated playtest evidence and reviewed diff.

One writer at a time. Commit passing increments. After two failed repair attempts for the same verification stage, stop and reassess. Do not mark unavailable gameplay checks passed.
