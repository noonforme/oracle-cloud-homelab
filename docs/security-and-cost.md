# Security and cost controls

## Administrative access

The management PC reaches OpenSSH and 1Panel over Tailscale. OpenSSH authenticates with keys. Private listener bindings and persistent IPv4 and IPv6 firewall rules restrict administrative access. A connection test on 18 November 2026 succeeded from the management PC and confirmed that public SSH and panel connections were blocked.

OpenSSH disables direct root login, password login and forwarding. Visitors and approved Minecraft players reach their respective public services without access to the management network.

The website runs as a non-root user with a read-only container root, read-only content and configuration mounts, dropped capabilities, memory and CPU limits, and bounded logs. Minecraft uses a non-root game process, authenticated accounts and an enforced whitelist. Administrators use the local container console with RCON disabled.

## Backup protection

The backup key can request only a fixed server backup operation from the source PC's private address. It cannot request a shell, PTY or forwarding. Pinned host-key verification and a dedicated identity avoid dependence on an interactive SSH agent.

Restic encrypts the PC repository. Local credentials and client state have explicit owner/SYSTEM filesystem access. During verification the client restores a plaintext archive into private scratch space. It removes that directory after successful validation and can retain it for diagnosis on failure. Server snapshot staging is root-only.

## Always Free target on Pay As You Go

I use a Pay As You Go tenancy and target Always Free usage. The VM uses two A1 OCPUs, 12 GB RAM and approximately 50 GB of boot storage. Eligibility depends on the home region, aggregate tenancy usage and Oracle's current allowances. [Oracle Always Free resources](https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm)

## Budget and transfer usage

OCI showed an estimated **€0.00 for 1–18 November 2026**. The root compartment has a **€1 monthly budget** with an actual-spend alert configured at **1% (€0.01)**.

Oracle evaluates budget alerts every 24 hours. Resources continue running when a threshold is crossed, so spending can accrue between evaluations. [Oracle budget behavior](https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/budgetsoverview.htm)

A 10 Mbit/s egress shaper limits the VM's outbound transfer rate. The backup uses existing PC storage. Each new snapshot streams a complete archive from OCI, so the transfer still counts toward outbound usage.
