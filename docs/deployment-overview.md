# Deployment overview

The deployment uses two Compose projects on an ARM64 host. Each project defines its own image, bind mounts, resource limits and health check. [Configuration examples](../examples/README.md)

## Prerequisites

An authorized OCI tenancy, compatible ARM64 capacity/images, private administrative access, persistent application directories and a recovery destination are required. Check current account-wide usage and Oracle entitlements before changing resources.

## Workflow

1. **Prepare the host and access boundary.** Establish compute/storage and private administration. Confirm a recovery route before restricting access. Current management uses Tailscale, OpenSSH keys and a private panel listener.
2. **Separate the Compose projects.** Pin images and retain application data outside replaceable container filesystems.
3. **Serve the website.** Mount static source/configuration read-only, preserve certificate/runtime state separately, configure DNS/application ingress, then verify HTTPS, redirects and assets.
4. **Configure Minecraft.** Pin game, loader and Java compatibility together. Enable online authentication and the whitelist, check startup/health and join with a compatible client.
5. **Add bounded pre-generation.** Verify the mod artifact and dependencies, take a cold recovery copy, install Chunky and check the world/radius. Configure native pause at one online player and automatic continuation.
6. **Establish independent recovery.** Install the consistent snapshot job and restricted pull key. Schedule encrypted PC backups, then restore and validate a copy before accepting the job.

## Change discipline

Resolve compatibility before maintenance. Check players and free space, preserve previous configuration and prepare recovery. Start only the affected service, then check behavior and the other workload.

During the Chunky installation, a failed console check triggered a rollback to the previous game configuration. The website stayed healthy during the rollback and retry. [Installation case](troubleshooting.md#console-access-during-mod-installation)

## Service acceptance

| Service | Acceptance check |
| --- | --- |
| Website | Request the configured site name over HTTP, follow the redirect to HTTPS and load an asset through the public route. |
| Minecraft | Check container health, join with an approved account and compatible client, then verify that generation resumes after disconnect. |
| Administration | Open a fresh key-based SSH session and reach the panel from the management PC over Tailscale. |
| Backup | Restore the archive into a private directory, verify its manifest and open the restored panel databases. |
