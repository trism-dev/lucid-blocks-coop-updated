# Lucid Blocks co-op architecture

Status: implementation contract; current-version multiplayer is **not verified**. Based on `design.md` and `authorization.md`, read 2026-10-23. This document changes no game files.

## Target, constraints, and evidence

Pin the runtime to Steam app 3495730, build 25451214, game 5.1.0, Windows x86-64, **Godot 4.6.0 double precision**, GodotSteam 4.17.1. `work/recovered/project.godot`, `work/logs/probe.txt`, and `work/gdblocks-source/extension_api.json` establish these versions. The generic engine guide's installed 4.7.2 editor is not the build tool for this game. Exact native ABI/source equivalence remains unverified; record source commit, godot-cpp commit, API hash, compiler, engine hash, and build flags before native output is accepted. Target executable and original DLL hashes are in `authorization.md`.

Engine guide: `../guide.md`. Use headless import, per-script `--check-only`, and a SceneTree test runner with explicit `quit(code)`. Record output and assertions as well as exit code: Godot can ignore unknown flags. No assumed GUT/gdUnit installation and no automatic editor download. Do not hand-edit open scenes/resources; keep `.godot`, exports, original saves, and recovery output out of git.

Ownership and tool boundaries:

- This architecture stage writes only `architecture.md`. Implementation stages may edit reviewed mod GDScript, native patch source, build scripts, and tests, one writer at a time.
- `work/recovered/` is private reference material. GDRE recovered 396 scripts according to the recovery result supplied to this stage; its log explicitly warns that exported resources are inaccurate because the extractor was single precision. **Never package those recovered resources or substitute them for original game assets.** Decompiled scripts remain evidence to compare against, not proof of runtime correctness or permission to redistribute the commercial game.
- Keep the purchased installation and real saves unchanged during build and checks. Stage original runtime files privately under `work/`; isolate both test profiles. The installer alone owns deployment/rollback, with a manifest and verified backups. Do not run inherited `scripts/deploy_mod.sh` as a build check: it combines export and installation and enables native installation by default.
- Use the original executable to consume original resources. A matching double-precision editor/import pipeline is a prerequisite for source-based tests that need these resources. Do not silently fall back to another Godot version or precision.
- Native source under `work/gdblocks-source/` is a private build input. Store reviewed changes as an attributable source patch and pinned source revision under `native_patch/source/`, not a second copied engine tree. The official source README documents double-precision compilation and mod-only PCK export. Its root `SConstruct` owns native outputs.

## Observed implementation

The existing entry point is `mod/overrides/main/autoload/ref.gd`: it bootstraps `CoopNativePatch`, loads patch configuration, and creates `coop_mod/coop_manager.gd`. The current game's `main/autoload/mod_loader.gd` loads `.pck` files beside the executable and also recognizes `res://mods/<name>/mod_init.gd`. Preserve one manager bootstrap; do not introduce a second autoload.

`coop_manager.gd` already contains LAN/Steam lobby logic, state RPCs, instance keys, guest snapshots, storage synchronization, entity authority, and reconnect handling. Presence of those paths does not demonstrate functioning Steam transport. Current `Ref` resolves one `Main/World`, one player, and global scene services; spawning another world in that scene is not a small safe change.

Evidence anchors (line numbers refer to files read at this revision):

| Evidence | Consequence |
|---|---|
| `work/recovered/main/world/world.gd:302` through `load_chunk_data` | Terrain is gzip JSON in `chunks/<dimension>/cX.Y.Z.txt`; load uses native `load_region`. |
| `work/recovered/main/save_file/save_file_manager.gd:211` | Save backs up, writes register, writes terrain, then `data.txt`; this is not a multi-file atomic commit. |
| `work/gdblocks-source/extension/include/world.h:178`, `:228` | Native loader exposes one center; dictionary `save_data` is deprecated. Region read/write APIs exist. |
| `mod/overrides/coop_mod/coop_manager.gd:7182` | Guest terrain merges into old save dictionary; `_apply_world_patch_locally` uses `load_data`. This cannot be assumed to persist in current region files. |
| Same file `:12262`, `:12334` vicinity | Persistent profile request/submission accepts a caller-provided player key; identity must be derived from the admitted sender. |
| Same file `:12287` | Guest flush acknowledgement follows sending a patch; it is not a host disk-commit acknowledgement. |
| Same file `:5127` | Guest becomes world authority when host is in a different instance. Keep this narrow delegated model, but add explicit leases and persistence. |
| `native_patch/runtime_extension/extension/coop_native_patch.cpp:109` | Existing memory patch uses fixed function RVAs and short byte checks. Target-build compatibility is unverified. Proxy source still lists hash checking as future work. |
| `work/recovered/main/world/living_block/cache_cube/cache_cube.tscn:26` | Current cache cube capacity and maximum are both 21. |
| `mod/overrides/main/world/living_block/cache_cube/cache_cube.gd` | UI directly exposes `%Inventory`, refresh sends full storage snapshot, breaking drops its contents. Personal storage must bypass these paths. |
| `work/logs/probe.txt`, `probe-errors.txt` | Runtime starts as 4.6 double and loads game systems; Steam initialization fails to find its client directory in that probe. No `COOP_PROBE` result proves transport class availability. |

## Systems and single owners

Paths below are relative to the repository. New files are explicitly marked **planned**.

| System | Single owner | Owned state and collaborators |
|---|---|---|
| Bootstrap/capability admission | `mod/overrides/main/autoload/ref.gd` | Loaded extension handles and feature availability; starts manager only once. |
| Session, identity, transport, leases | `mod/overrides/coop_mod/coop_manager.gd` | Admitted peer map, session epoch, per-instance lease epoch, RPC validation, reconnect state, snapshot assembly. Calls store and existing world/entity APIs. |
| Native chunk residency | `native_patch/source/world_multi_region.patch` (**planned**, against official `extension/include/world.h` and `extension/src/world.cpp`) | Union of at most two validated load centers, resident chunks, jobs and eviction. Existing runtime hooks stay disabled on unknown binaries. |
| Scene world integration | `mod/overrides/main/world/world.gd` | Local active dimension, generator and world-to-native adapter; sends/checkpoints region data through manager. |
| Durable co-op state | `mod/overrides/coop_mod/coop_store.gd` (**planned**) | One host-side commit queue, generation manifests, player profiles, region snapshots, chest records and transaction receipts. Sole writer of co-op sidecar. |
| Vanilla save integration | `mod/overrides/main/save_file/save_file_manager.gd` (**planned override of current script**) | Calls existing backup/save flow through a checkpoint barrier, includes sidecar in backup/restore, preserves all current-version behavior. |
| Personal chest behavior | `mod/overrides/coop_mod/ender_chest.gd` (**planned**) | Validates chest operations and manages UI session/revision. Durable data belongs to store; peer identity belongs to manager. |
| Chest interaction/UI | Existing `main/world/living_block/cache_cube/cache_cube.gd` and `main/ui/menu/game_menu/inventory_widget/inventory_slot/inventory_slot.gd` under `mod/overrides/` (confirm exact slot path before task) | Ordinary cube behavior stays intact; designated access points route through chest owner, using existing widgets. No separate chest artwork. |
| Entity simulation/rendering | Existing entity and behavior scripts under `mod/overrides/main/entity/` | One lease holder simulates; proxies render. Manager determines instance membership/targeting and routes snapshots. |
| Build/install/rollback | Planned `scripts/build.ps1`, `scripts/install.ps1`, `scripts/rollback.ps1` | Build manifest, staged PCK/native artifacts, backups; no gameplay state. |

Keep existing helpers in place. Extract only the durable store and chest protocol because each has a distinct state owner and test boundary; do not split the large manager as an unrelated refactor.

## Authority and identity

The host owns the save, admitted membership, lease grants, chest/profile commits, and all final persistence acknowledgements. A runtime peer number, display name, IP address, or caller-provided `player_key` is never a persistent identity.

Steam identity must come from the authenticated Steam transport/session, bound to the sender connection. Lobby metadata is discovery data, not identity proof. Query the actual target's Steam singleton methods and transport class before choosing adapters. If `SteamMultiplayerPeer` is absent, the Steam feature is blocked until a compatible double-precision transport is built and tested; do not advertise success from lobby creation alone.

LAN uses a host-issued opaque reconnect credential mapped to a host-local player UUID, scoped to the save. Issue it on first admitted join, store only its digest on host, compare through the session handshake, and never put credentials in logs. Treat LAN as trusted-network co-op unless transport encryption is added and verified; no claim of security against a network observer. Avoid automatically merging an unverified LAN identity with a Steam identity. Legacy identity migration requires an unambiguous host-recorded association; retain ambiguous records rather than give them to a new peer.

Handshake before snapshots: protocol/schema version, target game build, native capability/version, maximum players **2 total including host**, session nonce, authenticated identity, and compatibility fingerprint. Reject mismatches, duplicate live identity, malformed messages and a third participant before sending world or profile data. Derive `(save UUID, instance ID, player UUID)` from admission and host state. Public dimensions use a shared dimension ID; personal Apotheosis uses `(dimension ID, owner UUID)`. Treat instance IDs as typed data; derive filesystem paths from canonical validated IDs, never raw network paths.

Every mutating RPC checks admitted sender, session/lease epoch, current instance, monotonically accepted request ID, payload sizes, finite coordinates, bounded coordinate/radius, item IDs/counts/durability, interaction distance and permission. Reject arbitrary scene/resource paths: entity types must be an allowlisted game type. Limit snapshot chunk count, decompressed size, outstanding transfers and assembly lifetime; receive serialized data without executable Object construction. Return only the requesting player's private profile/chest. Visiting Apotheosis permits agreed world interaction, not reading the owner's chest.

## Independent loading and dimensions

Two players in the same instance use one authority and a **union of two normal-radius chunk neighborhoods**, not one enormous midpoint sphere and not alternating single-center residency. Deduplicate overlapping chunks and decoration jobs; retain chunks until neither player requires them, and save dirty state before eviction. Bound the number of centers to admitted players and the total pool to the measured two-player budget. Avoid deleting or reusing chunks while worker jobs reference them. The source draft `native_patch/source/world_multi_region_rewrite.md` is a starting sketch, not a proven patch for this version.

Prefer a source build of the published native extension with the current API over expanding fixed-address hooks. First prove the unmodified pinned native source can replace the original in a disposable runtime with identical game-visible behavior. Then add `set_loaded_region_centers` while keeping the single-center method for vanilla callers. If source parity fails, stop the native task with evidence; do not install the old hook binary as a fallback. If hooks must later be retained, require exact original DLL SHA-256, full expected instruction bytes for every site, read-only preflight of all sites before any write, and complete rollback on partial failure.

Each process still has one active `Ref.world`. When players occupy different instances, the host grants a simulation lease to the process in each instance. The guest can simulate its loaded dimension, but cannot write the host save directly. It submits validated, sequenced region/entity checkpoints to the host store. This is a cooperative trust model: a malicious leased simulator can invent plausible world events. Full authoritative anti-cheat would require a larger multiworld server redesign and is excluded.

When the host enters a guest-simulated instance, freeze that instance's mutations, request its final checkpoint, commit it on host, revoke the old lease, increment epoch, load the committed snapshot, then resume under host authority. Do not allow two writers during transfer. If checkpoint/ack fails, retain the last durable generation and report the incomplete transition. The same barrier applies when leaving a private instance. Disconnection expires leases; stale packets and old-epoch replay cannot change the save. A rejoining guest starts from host state, never overwrites it with a local stale profile. Do not promote a disconnected network guest to offline authority for that host world.

## Persistence and transactions

Vanilla paths and formats remain compatible for ordinary worlds. Add a versioned sidecar inside the host save directory: `coop/`. The store owns immutable generation payloads plus a committed manifest containing schema version, save UUID, generation, per-instance region revisions/epochs, profiles, chest data, access-block designations, checksums and replay receipts. Region payloads reuse current native `get_region_data` / `load_region` serialization, including water, fire and other actual native fields; do not synthesize a substitute from block IDs alone. Save keys include instance identity and region coordinate, so private Apotheosis regions cannot overwrite the public dimension or another owner.

Commit protocol:

1. Host validates and serializes requests through a single queue. Each command carries request ID, expected revision and lease epoch. Build the entire next generation from the last committed state.
2. Write changed payloads to new generation files, flush/close, reopen and validate size/checksum/format. Write a manifest referencing a complete set of payloads; shared unchanged payloads may remain referenced.
3. Publish the manifest using a verified Windows same-volume replace operation, retaining the previous committed manifest. Never delete the old manifest before publishing the new one. Until replacement semantics are proven by crash tests, this task remains blocked; a successful `store_buffer` alone is not a durability guarantee.
4. Only then emit the host commit acknowledgement with generation/revision/request ID. A repeated request returns the stored result and does not apply twice. On restart, load the newest complete valid manifest, or the previous valid generation; ignore orphan staging files.

A save/quit acknowledgement covers region data, preserved entities, player inventory, chest and transfer receipts in one co-op generation. Include co-op shared-region changes that must survive a crash before vanilla saves; reconcile/materialize these into vanilla files at the controlled save barrier. Intercept vanilla restore/load so restored `data.txt` cannot be combined with a newer unrelated co-op generation. Backups must contain a consistent manifest and every referenced payload. Never silently discard corrupt or unsupported schemas, overwrite a valid old save, or delete migration sources.

Migration reads old `coop` player/dimension data and legacy chunk dictionaries into a new generation, validates against actual region schema, and keeps a backup plus migration marker. An absent legacy region is not evidence it was empty. If old private progress is already missing, report that loss; do not claim reconstruction. Guest local files are cache/reconnect hints only.

## Personal ender chest

Host `/enderchest` designates an existing **empty standard cache cube**, checked on authority in the current instance. Cabinet and hopper styles are not eligible. Persist designation by instance plus block coordinate; no recipe, inventory growth or new resource is needed. Capacity is **21**, matching the current cache cube, with normal item stack limits. Store contents under `(host save UUID, player UUID)`, independent of any access block.

Opening creates a short-lived server-issued interaction session tied to player, block, instance and chest revision. Existing UI receives that player's snapshot. Client sends a slot transfer intent, not an authoritative full inventory replacement. Host validates source slot/count, destination capacity/stack compatibility, expected player-inventory and chest revisions, and item conservation. Commit both inventories and the idempotent receipt together, then acknowledge and update UI. The host player's UI must use the same path. Reject stale sessions after movement out of reach, dimension change, death, unload, disconnect or block destruction.

Destroying an access block closes active UI and removes only its designation. Personal items never enter the block `%Inventory`, ordinary drop routine, hopper path or world chest broadcast. Another designated cube opens the same player's chest. Ordinary cache cubes retain their current drop and synchronization behavior. Audit drag/drop, split, shift-transfer, cursor-held item, fusion/consumption and close paths before implementation; a UI-only reroute without inventory transaction ownership would duplicate items.

## Ordered implementation and verification contracts

Run from the repository root. New commands/files below are **required planned hooks**, not claims that tests already exist or pass. `tasks.md` is the execution ledger. Confirm actual paths before implementation; any added caller must be named in that task before editing.

| Order | Files touched by one writer | Exact check/hook and required evidence |
|---|---|---|
| A1: safe build/install baseline | `scripts/build.ps1`, `install.ps1`, `rollback.ps1`, `tests/tooling.Tests.ps1`, build manifest | `pwsh -NoProfile -File tests/tooling.Tests.ps1`: fixture install/rollback byte equality, wrong original hash rejected, interrupted install recovered, build never changes install. `pwsh -NoProfile -File scripts/build.ps1 -OutputDirectory work/stage`: output hashes and explicit file allowlist; no recovered resources or hidden native deployment. |
| A2: target runtime/admission | `main/autoload/ref.gd`, `coop_mod/coop_manager.gd`, `tests/runtime_probe.gd`, `tests/run.gd` under `mod/overrides/`; runtime patch guard source only if kept | `pwsh -NoProfile -File scripts/verify.ps1 -Scenario RuntimeProbe` (**planned orchestration script**): probe logs 4.6 double, API methods, Steam singleton/peer capabilities and original/staged hashes. `-Scenario Admission`: wrong build/schema, third player, spoofed player key and stale epoch rejected before snapshot. Unknown native DLL produces zero memory writes. |
| A3: current-script compatibility and known issues | Existing world/cache/entity/behavior overrides identified by diff against recovery; `coop_manager.gd`; scenario fixtures | `pwsh -NoProfile -File scripts/verify.ps1 -Scenario Compatibility`: exact override inventory and per-script parse output; guest leave -> solo -> rejoin without process restart, gel/small-gel visible before damage, Manikin targets valid nearest same-instance participant and releases invalid targets. Preserve upstream defects individually in results. |
| A4: native union residency | `native_patch/source/world_multi_region.patch`, source revision manifest, `main/world/world.gd`, manager load-center adapter, native scenario fixture | In pinned native source, `scons platform=windows precision=double target=template_release custom_api_file=extension_api.json`. Then `pwsh -NoProfile -File scripts/verify.ps1 -Scenario MultiRegion`: both distant neighborhoods remain loaded while moving; overlap has one chunk; edits survive eviction; water/fire/entities run near both; job lifecycle and peak memory recorded. |
| A5: durable store and dimension leases | Planned `coop_store.gd`, current-version save manager override, manager/world adapters, store fixtures | `pwsh -NoProfile -File scripts/verify.ps1 -Scenario Persistence`: kill host before/after each commit boundary, disk-full/write failure, corrupt latest generation, duplicate/out-of-order RPC, backup restore, legacy migration. Every acknowledged edit survives; unacknowledged edits may roll back without duplication. `-Scenario Dimensions`: both players continue independently, visit each Apotheosis, transfer lease, disconnect/rejoin, then restart host and compare region/entity checksums. |
| A6: chest transactions | Planned `ender_chest.gd`; manager command/RPC routing; store profile/receipt updates; cache cube and exact inventory-slot callers; chest fixtures | `pwsh -NoProfile -File scripts/verify.ps1 -Scenario EnderChest`: 21-slot bound, stack limits, two identities/two access blocks, private visibility, duplicate transfer/reconnect/crash, full inventory, drag/split/shift/cursor-close, destruction mid-transfer, no drop/hopper leakage, migration and ordinary cache regression. Assert total item conservation across inventory/chest/cursor/drops. |
| A7: Steam and soak | `scripts/verify.ps1`, scenario fixtures and `playtest.md`; only failed-feature owners may change production code | `pwsh -NoProfile -File scripts/verify.ps1 -Scenario Steam`: two authenticated Steam accounts, invite while running/from launch, full lobby, disconnect/rejoin and host loss. `-Scenario Soak -Minutes 60`: two isolated clients, repeated separation/travel/build/combat/chest/save/restart; record errors, authoritative revisions, hashes, memory and rollback result. Missing second account or transport is BLOCKED. |

`verify.ps1` must launch disposable staged runtime/profile pairs, return nonzero on failed assertion/timeout, capture stdout/stderr and engine/hash metadata, and tear down its own processes. For isolated GDScript unit contracts use the pinned double engine explicitly: `& $GodotDouble --headless --path work/test-project -s res://tests/run.gd -- persistence` (or `admission`, `ender_chest`); `$GodotDouble` is a resolved, version-verified executable path, not the generic PATH shim. Before running, import with `& $GodotDouble --headless --path work/test-project --import`; parse each changed script with `--script <res-path> --check-only`. The test project must supply required native classes/autoloads or clearly scope a pure-data fixture. A parse failure caused by absent game globals is not a successful compatibility test.

The Steam probe already attempted does not validate the proposed runner: its requested external script did not produce `COOP_PROBE` output. Prove the game-supported launch/mod-init test entry in A2 instead of assuming exported executables honor editor script flags. Steam runtime checks require the real client environment; isolated profile variables must not hide the Steam installation. Never let a test select a real save.

## Minimum safe next task, risks, and exclusions

**Next: A1, build-only Windows staging and disposable install/rollback tests.** Then A2 proves a matching double-precision test entry and native/Steam capabilities. Until those gates pass, do not install fixed-RVA hooks, migrate user saves, or claim current-build multiplayer support.

Primary blockers: matching source/runtime ABI is unproved; matching editor/import/export toolchain is unproved; recovered resource precision is explicitly wrong; existing patch addresses are unvalidated; current guest persistence uses the deprecated path; the runtime probe failed Steam initialization and did not establish transport capabilities. These are separate gates, not one generic "needs testing" item.

Risks: two normal load bubbles can approach twice the chunk budget; guest simulation cannot provide hostile-client anti-cheat; multi-file persistence needs crash/failure testing, not only clean save/reload; inventory UI has mutation paths outside a simple chest click; inherited broad script overrides may overwrite new current-version behavior. Keep diffs narrowly rebased and retain vanilla regressions.

Not building: dedicated server, more than two players, host migration, general multiworld scene architecture, new art/models/skins/recipes, hopper-to-personal-chest automation, hostile-internet LAN security, or unrelated refactors. Do not replace requested independent dimensions with forced group travel. No feature is complete merely because a static check passes.
