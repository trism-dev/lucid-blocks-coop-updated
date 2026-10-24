# Two-player co-op compatibility update

## Pillars
1. Both players can play independently, including different dimensions.
2. Host-owned saves preserve each player's world and inventory without duplication.
3. Joining and leaving work through Steam or LAN without restarting the game.

Target: Windows Steam build 25451214, executable 5.1.0.0. Upstream baseline 57140c7abe02a921defa4184d474fb3e0ef841eb.

Loop: host loads world; friend joins; explore/build/fight independently; visit personal Apotheosis; save, leave, and return with progress intact.

Include all upstream README Known Issues, Steam invites, independent terrain loading, simultaneous dimensions, personal Apotheosis visits, and bounded personal ender chest. Ender chest reuses an empty cache cube and standard inventory capacity, is designated by host with /enderchest, is personal per host save, and survives destruction of access blocks.

Exclude new art/models/skins, dedicated servers, more than two players, new recipes, hopper integration for ender chests, and unrelated issue-tracker requests.

Playtest questions: Can both players travel apart safely? Does every acknowledged edit survive restart? Can a guest leave/rejoin without restarting? Do gels and Manikins behave consistently? Can one player ever see or duplicate another's personal storage?
