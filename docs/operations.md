# Operations and recovery

I administer the VM over Tailscale and key-based OpenSSH. I check container status on the host and test the service from a client.

## Routine checks and changes

Run these read-only checks in a private management session. Run `docker compose ps` from the directory containing the Compose file for the service under review.

```sh
uptime
free -h
df -h
docker compose ps
docker stats --no-stream
systemctl is-active docker tailscaled
```

| Trigger | Action |
| --- | --- |
| Gameplay becomes slow | Reproduce the issue with the affected client and compare CPU, memory and tick behavior while it occurs. Check whether background generation is active. |
| A service fails | Inspect its status and recent logs privately. Confirm versions, mounts, disk space and dependencies before changing configuration. |
| Before an update | Identify a verified recovery point, check compatibility and retain the previous configuration and image reference. |
| After a change | Check website HTTPS, game health and a real client connection. Confirm that the other workload still works. |
| After a missed backup | On the Windows PC, check the last successful restore validation and its source snapshot time. This gives the age of the usable recovery point. Check sign-in status, Tailscale connectivity and the task result if the copy is overdue. |
| Periodic maintenance | Review security updates, certificate renewal, disk growth, backup age and OCI usage. |

## Player-aware generation

Chunky pre-generates a circle with a 1,000-block radius around Overworld spawn. It checks the online player count every second. When a player joins, it pauses the task and saves progress. Work already in flight can finish.

In the join/disconnect test, saved progress held steady while a player was online. After the last player left, the chunk count began increasing again. The game stays awake while empty during pre-generation, and Chunky is configured to continue a saved task after application restart.

See the [troubleshooting case](troubleshooting.md#background-generation-and-gameplay) and [sanitized configuration](../examples/chunky-config.example.json).

## Scheduled backup coverage

Windows Task Scheduler is configured to run the backup nightly, retry failures and catch up at sign-in. It allows one run at a time. **The owner's Windows account must be signed in. A locked desktop works.** If the PC or Tailscale is unavailable, the newest validated recovery point can become more than a day old.

| Component | Captured recovery data |
| --- | --- |
| Minecraft | World, mods, configuration, server directory and Compose inputs |
| Website | Static content, Caddy/Compose configuration, TLS certificates, ACME state and runtime configuration |
| 1Panel | Persistent panel data and standalone consistent copies of all seven discovered SQLite databases |
| Host | Selected SSH, network, firewall and service configuration, operational helpers, administrative access state, package inventory and image references |

The server records which services are active, cleanly stops Minecraft and the panel, captures their data, and restarts the services that were running. Caddy stays online. Before accepting the snapshot, the job compares the captured site content with the source files and checks the certificate/key pairs.

Snapshot creation runs in a systemd unit independently of the SSH session. A recovery action uses the saved service states after the job exits. A boot-time check also looks for an interrupted capture.

A dedicated SSH key is limited to a fixed backup operation from the source PC. Each new capture transfers the full archive into the encrypted restic repository. Deduplication saves space on the PC. A same-day validation retry can reuse the stored archive and skip the transfer.

After a restored copy passes its checks, the retention policy keeps up to seven daily and four weekly recovery points as they accumulate. The server also keeps the two most recent local snapshots on its boot disk.

## Verification and restore

On 18 November 2026, the Windows scheduled task captured the archive. After a correction to the filename validator, the task restored and validated the same snapshot. Reusing that snapshot avoided a second interruption to Minecraft and the panel. [Restore validation case](troubleshooting.md#backup-validator-rejected-valid-filenames)

| Check on the restored copy | Result |
| --- | --- |
| Manifest | SHA-256 matched for all 353 regular files. |
| Panel databases | The snapshot job used SQLite's backup API to create standalone copies. All seven passed `PRAGMA quick_check` when reopened from the restore. |
| Website TLS | Two restored certificates loaded with their matching private keys. |
| Minecraft world | The validator decompressed the world metadata file and verified its root tag. |

Repository pruning and a full read of the stored data also passed. The game and panel stop/copy/restart sequence took about **34 seconds** during capture. The website container stayed running throughout.

For application recovery, restore into an isolated directory first. Preserve Linux ownership, permissions and links when placing data on a compatible host. Check mounts and service health before directing clients to recovered services. Keep restored host keys and node identities isolated from the running VM.

The restore requires access to the restic repository and its password. [restic backup input](https://restic.readthedocs.io/en/stable/040_backup.html#reading-data-from-a-command) · [Restoring snapshots](https://restic.readthedocs.io/en/stable/050_restore.html) · [Retention](https://restic.readthedocs.io/en/stable/060_forget.html) · [SQLite backup API](https://sqlite.org/backup.html)
