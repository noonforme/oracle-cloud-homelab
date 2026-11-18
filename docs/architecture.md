# Architecture

One OCI Ampere A1 VM hosts the website and the game. Each has its own Compose project, so a site update can leave the Minecraft process and its connected players undisturbed. [Architecture diagram](../README.md#architecture) · [Mermaid source](../diagrams/architecture.mmd)

## Compute and services

| Component | Observed configuration | Operational implication |
| --- | --- | --- |
| Host | Ampere A1 Flex, 2 OCPUs, 12 GB RAM, Ubuntu ARM64 | Both workloads share one machine and its failure domain. |
| Website | Separate Compose project using a digest-pinned Caddy ARM64 image | Website and game maintenance can be performed independently. |
| Minecraft | Pinned Java 25 image, Minecraft 26.1.2 and NeoForge 26.1.2.127 | The Java runtime, mod loader and installed mods must support the same game release. |
| Chunky | Version 1.5.66 with a 1,000-block radius around Overworld spawn | Generation pauses while a player is online and resumes when the last player leaves. |
| Administration | OpenSSH and 1Panel through Tailscale | Public application entry points are separate from administration. |
| Backup | Systemd snapshot job, restricted SSH pull and a Windows restic repository | Recovery data is retained outside the OCI VM. |

## Traffic flow

1. Visitors reach Caddy over HTTP/HTTPS. HTTP redirects to HTTPS for configured website names.
2. Minecraft clients use the public game TCP listener. The server authenticates the account and enforces a whitelist before allowing a player to join.
3. The owner's PC reaches key-based SSH and the panel through Tailscale. Public administrative connections were blocked during review.
4. The scheduled PC job requests a consistent snapshot through a dedicated restricted key. Services recover before transfer to the encrypted local repository.

The site serves a static landing page and assets directly from disk. The SQLite databases in the backup belong to 1Panel.

## Persistence and resource limits

Bind mounts keep the website files, TLS state, game world and configuration on the host's boot filesystem. Replacing a container preserves those files.

Caddy runs under a non-root UID with a read-only root filesystem, read-only source/configuration mounts, dropped capabilities and a 128 MiB memory cap. Minecraft has an 8 GiB container memory cap and a 6 GiB maximum Java heap. Its Java process runs under the game UID. Both containers have bounded JSON logs.

## Recovery boundary

A boot-disk failure affects both workloads and the two recent snapshots staged on the VM. The encrypted repository on the PC provides a recovery copy independent of that disk.

The archive contains application data, selected host configuration, package inventory and image references. Recovery uses a compatible OS and restores ownership, permissions and mounts for each service. [Backup contents and restore checks](operations.md)
