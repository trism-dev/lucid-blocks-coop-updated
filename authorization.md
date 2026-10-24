# Local modding investigation

Date: 2026-10-23. Owner: Tristan (trism-dev).

The user owns the installed Lucid Blocks copy and explicitly requested this compatibility update. The developer documents recovery with GDRETools and modification of the native extension at https://lucidblocks.com/modding.html. This investigation uses that documented modding workflow, without DRM bypass or redistribution of the recovered game.

Target: Steam app 3495730, build 25451214, executable file version 5.1.0.0. Original installation remains read-only during recovery; extracted files and private test saves stay in ignored work/.

SHA-256:
- lucid-blocks.exe: 336FECEEB499A4FA3832FEA2D58BBE3C52B0B111086CAC42134DA4A16B051F83
- libgdblocks.windows.template_release.double.x86_64.dll: B25C20E9AE4CF0C21CBC015CDF58A8F4C3F0C5B8AA1652F81A071F471CD2CFEA

Upstream mod baseline: parkers0405/lucid-blocks-coop at 57140c7abe02a921defa4184d474fb3e0ef841eb. Preserve upstream attribution and notices; no new license is assigned to inherited work.
