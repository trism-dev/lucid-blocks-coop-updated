# Implementation process and roadblocks

This is a running record for the requested final retrospective. Completion claims belong in playtest.md.

## Initial investigation, 2026-10-23
- Created the public trism-dev/lucid-blocks-coop-updated fork, preserving upstream history; checked out fix/current-build-compatibility at 57140c7abe02a921defa4184d474fb3e0ef841eb.
- Located Steam build 25451214 and recorded executable/native DLL fingerprints in authorization.md.
- GitHub CLI initially failed inside the sandbox; an authorized elevated network call succeeded.
- Recovered 396 GDScript files with official GDRETools 2.7.4. Recovery succeeded but flagged single/double precision resource conversion inaccuracies. Recovered resources are reference-only and must not ship.
- The developer's official modding page linked the matching Godot 4.6 double editor. Downloaded it locally and verified 4.6.stable.double.custom_build.
- Tool startup initially could not write its default user directory; redirected APPDATA into ignored work/tool-profile.
- A direct exported-game script probe started the game's normal startup instead of producing the intended probe output. Steam initialization failed in that sandbox. This is not a successful vanilla gameplay baseline.
- Source inspection found private-world patches still stored in legacy dictionaries, whereas current saves use region files. Existing flush acknowledgment does not establish disk persistence.
- Upstream already contains Steam lobby, private-instance, reconnect, and storage code. These require verification rather than wholesale replacement.
- Usage cutoff interrupted architecture/tooling agents. Files survived, but their test status must be re-established.

## Resumption, 2026-10-24
- Resumed on Sol with narrow local reads and sequential implementation work.
- Discovered editor-generated .import changes; these need separation from intentional changes before committing.
- Resumed the tooling writer to finish disposable install/rollback checks and ensure exports stage copies rather than mutating tracked imports.
- Installer fixtures passed 34 assertions on a fresh parent run; checkpoint 1fd78c6 was committed and pushed to the compatibility branch.
- Matching editor could read the original game pack, but native classes were initially absent because extension declarations referenced nested physical DLL paths. Staging the DLLs at those paths restored World, Steam, and SteamMultiplayerPeer in the probe.
- A two-file raw GDScript/remap sentinel package loaded through the original mod loader. A subsequent full mod-only diagnostic PCK was created (55,233,380 bytes), avoiding recovered-resource conversion.
- Full-package probe uncovered actual source incompatibilities: the world override omits LUCY/TORUS dimension constants and generator selection, and the entity override lacks get_water_flow used by current dropped items. Exit code 0 did not imply clean script compilation; the production builder must inspect logs before publishing an artifact.

## Cost lesson
Initial exploration returned an oversized GitHub tree and unnecessarily large instruction/source outputs. Output size is not an exact usage accounting, but this was avoidable overhead. Future reads should be bounded before tool output reaches the conversation; keep implementation state in these files.

## Startup checkpoint f10f19f
- Fixed missing world dimensions/generators, Entity water-flow API and dexterity, wrong OfflineMultiplayerPeer API call, and absent fallback capsule resources.
- Full normal-Main startup checked 61 scripts plus targeted assertions. Strict error gate initially found a leak also present in vanilla: on-screen keyboard KeyListHandler. Probe-only cleanup freed that member; no production suppression or base-game resource replacement.
- Worker final fixture results: 42 assertions in Windows PowerShell and pwsh. Candidate fingerprint and logs are recorded in playtest.md.

## Next persistence investigation
- submit_guest_world_patch must cross-check dimension/owner against sender state, not trust a supplied instance key alone.
- _merge_world_patch_into_save writes a compatibility dictionary, not current region files. World.save_chunk_data/load_chunk_data provide existing gzip JSON format and owner namespaces.
- Patch payloads use prefixed dictionaries. Never write those directly as region files: merge with complete target region content or transmit validated serialized region payloads.
- Existing flush acknowledgement is sent before durable storage. Host must acknowledge after successful atomic region checkpoint, and transition/disconnect must await that acknowledgement.
- Relevant manager functions: submit_guest_world_patch, _merge_world_patch_into_save, _await_guest_world_patch_flush_for_instance; world functions: save_chunk_data, load_chunk_data, _get_dimension_save_namespace. Runtime/source probe still needed for exact region merge and block-ID semantics.

## Session admission investigation
- Added shared SceneMultiplayer authentication before peer_connected/world RPC admission, LAN one-remote limit and Steam two-total lobby limit, and a native DLL validation guard before extension loading.
- Sandbox loopback produced misleading failures: ENet socket creation returned OK but packets never reached the other peer. An empty-project control with explicit polling failed inside sandbox and passed outside it. No firewall settings were changed.
- Elevated actual regression run proved matching connection and reconnect slot reuse, but found rejected-client failure callbacks were not invoked after local disconnect. Fixed explicit failure handling before disconnect and preserved the client/host role and rejection reason.
- Added typed validation checks and precise malformed-hello regression cases. Native guard assertions passed; production source startup passed independently. Final admission rerun is pending; these changes are not yet a verified checkpoint.
