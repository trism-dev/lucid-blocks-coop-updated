# Lucid Blocks co-op updated (Godot mod)
Start with `STATE.md`. Engine rules: `..\guide.md` (`C:\Work\Code\GameDevelopment\Godot\guide.md`). Authorization and scope: `authorization.md`.

## Rules
- Original game install stays read-only. Recovered/extracted files and test saves live in ignored `work/`; never commit them or commercial game files.
- Verify with headless import, per-script `--check-only` and the SceneTree tests in `tests/`; record real output in `playtest.md`. A check that cannot run is BLOCKED, not passed.
- Git: public fork. Feature branches only (`fix|feat|chore|docs|build/*`), never force, never push `main`; PR, release and publish need the user's approval.
- Do not touch the user's uncommitted `.import` changes: commit only the files you changed.
