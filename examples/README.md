# Sanitized configuration examples

These excerpts show the settings used to serve static files, preserve Caddy's TLS state and pause Minecraft world generation when a player joins. The site names are reserved placeholders.

| File | Purpose |
| --- | --- |
| [website.compose.example.yaml](website.compose.example.yaml) | Pinned Caddy image, persistent TLS state, restricted container and resource limits |
| [Caddyfile.example](Caddyfile.example) | Static holding-page routing with a reserved placeholder domain |
| [chunky-config.example.json](chunky-config.example.json) | Native pause threshold and saved-task continuation |

Compose resolves the bind mounts relative to the example YAML file. The expected inputs are `config/Caddyfile` and a `site/` directory containing the holding page and its assets. The writable storage directories must allow writes by the example UID. The reserved `.invalid` names are placeholders and cannot obtain public certificates.

From the repository root, use Docker Compose to check the example configuration without starting a container.

```sh
docker compose -f examples/website.compose.example.yaml config --quiet
```

The example's health check requests HTTP inside the container. External DNS, the public certificate and the HTTPS response require a separate client check.

The Chunky configuration sets the pause threshold to one player and enables continuation after restart. The deployed JVM uses `-Dchunky.playerPauseInterval=1` to check the player count every second. A separate saved task defines the generation area. The game container stays awake while empty so the task can progress. [Chunky 1.5.66 release](https://www.curseforge.com/minecraft/mc-mods/chunky-pregenerator-forge/files/9430065) · [Native player-pause logic](https://github.com/pop4959/Chunky/blob/master/common/src/main/java/org/popcraft/chunky/util/PlayerPauseTask.java)
