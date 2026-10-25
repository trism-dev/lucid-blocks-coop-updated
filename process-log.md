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

## Resume after engine-layout adoption
- User confirmed resumption and discretionary commits. Post-move admission59/native4/startup61 rerun passed with the same artifact hash as pre-move; logs work/build-logs/3c4219d3b78941089aea82b1b1f69113.
- Session checkpoint committed as 6ead68a and pushed on fix/current-build-compatibility. Parent pipeline guide adoption committed separately as ddcb646; no PR or release created.
- Independent read-only admission review found a reconnect overlay remaining in RECONNECTING state after terminal failure. Queued a production-handler regression and fix at the next source boundary; peer admission blocking itself passed.
- Known-issue inventory found Entity/Gel attacked signature mismatch (current callers can pass damage_position), and mini-gel position approximation omitting current mini indices. These are queued after persistence. Manikin guest targeting already exists and needs damage-routing gameplay verification.
- Checked Windows/WSL compiler availability without installing anything. No native compiler found; WSL Ubuntu has Python3. Existing distant-guest authority is a candidate for independent exploration, to verify after durable checkpoints before adding a native toolchain.
- Region work confirmed water-awake payloads are64bytes per16-cubed chunk, not4096. Helper/integration work is active; no persistence pass is claimed yet.

## Region checkpoint verification and review
- Initial full gate passed21 fixture checks,15 native/ACK checks,62 startup checks,42 installer assertions. Logs work/build-logs/750a3ec9899a4d55b105a45bd3ad4052; candidate SHA1A4E16D05EF1DF2B4DB349C687ED41789D8093992A6DFA11C2AF15763E7F25C8. Not committed because review found transition-level gaps.
- Independent review traced real Main/SaveFileManager callers and found two data-loss paths not covered by serializer tests: host checkpoint failure logged without preventing world disposal, and destination owner written into the register before outgoing-world serialization.
- Reopened the same task for a bounded repair, prioritizing failed-host-transition and source-owner routing regressions rather than adding more low-level serializer checks. No personal game save or original installation was modified.
- Repair2 passed21 fixture/28 runtime/startup62, candidate SHA2A4EBF7D38DCF47A6F59E6F108D3BA0D40A48AF968229856646AA747010BADF4; logs work/build-logs/fc6b14519b974ba494850cee550139d4. ObjectDB warning remains in expanded harness; no claim of leak-free shutdown.
- Re-review confirmed host failed-save exits and normal owner handoff resolved. It found a remaining P1 legacy branch: SaveFileManager calls load_chunks_old then clear_old_chunk_data, bypassing queued destination handoff and potentially clearing wrong-owner data. Current passing tests do not cover that path.
- Stopped at the required two-round repair cap, without committing/installing persistence. Next bounded repair must consume destination handoff in legacy loading, select intended legacy/owned data, preserve migration sources until success, and test cross-owner legacy transitions. Existing safe checkpoint remains6ead68a.
- Coordination inefficiency: a message to a completed reviewer did not start a new turn; explicitly reopened it once identified. No source changes occurred while waiting. Future rechecks must use followup_task for completed agents.

Legacy repair1 passed28 fixtures/43 runtime/startup62, logs3cb5a2c872c84177ba14da97eae12259. Targeted reviewer resolved legacy P1; originals retained until an explicit future cleanup. Next: combat/Gel/UI regressions. No personal saves/install touched.
