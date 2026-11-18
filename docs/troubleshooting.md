# Troubleshooting cases

## Background generation and gameplay

Players experienced slow gameplay when two or more people joined during continuous world generation. The game and the generator shared the same two OCPUs. I configured Chunky to generate terrain within 1,000 blocks of spawn and to pause as soon as one player was online.

**Result.** During a real join, the task paused and its saved progress stayed steady. When the last player disconnected, the chunk count began increasing again. [Chunky release](https://www.curseforge.com/minecraft/mc-mods/chunky-pregenerator-forge/files/9430065) · [Player pause implementation](https://github.com/pop4959/Chunky/blob/master/common/src/main/java/org/popcraft/chunky/util/PlayerPauseTask.java)

## Console access during mod installation

A mod installation check failed while the game container's health check was passing. The console helper rejected commands run as root. After rollback, a retry also encountered an empty configuration directory left by the first attempt.

**Resolution.** The maintenance script now invokes the helper with the game's service UID and accepts an existing configuration directory. The retry completed, a player joined successfully and the website remained healthy throughout. [Container console commands](https://github.com/itzg/docker-minecraft-server/blob/master/docs/sending-commands/commands.md)

## Backup validator rejected valid filenames

The first encrypted snapshot reached the PC, but validation stopped on three systemd unit filenames. Their Linux escape sequences contained backslashes that the Windows validator interpreted as path separators.

**Resolution.** The validator now accepts the expected filename format in the systemd configuration directory and continues to reject path traversal. The scheduled task reused the encrypted snapshot, restored it and passed the file, database, certificate and world metadata checks. Minecraft and 1Panel stayed online during the retry. [Restore results](operations.md#verification-and-restore)
