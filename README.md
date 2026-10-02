# CrystalOptimizer

Lag, FPS and RAM optimizations for crystal PvP sandbox servers, with Bedrock (Geyser/Floodgate) support.

**Requires:** Paper (or a fork like Purpur) 1.20+, Java 21. Not Folia compatible.

## Build

```
mvn clean package
```

Copy `target/CrystalOptimizer.jar` into `plugins/` and restart. (Not compiled or tested in the environment it was written in, so if Maven reports an error on your Paper version, send it over and it's usually a one-line fix.)

## What it does

| Area | Effect |
|---|---|
| Crystal explosions | No item drops, optional no block damage, per-chunk-per-tick limit on block calculations (player damage is never cancelled), cap on crystals per chunk |
| Ground entities | Removes old items, stray XP orbs and stuck arrows; caps items per chunk |
| Mobs | Blocks natural spawns, caps mobs per chunk (plugin/command/egg spawns exempt) |
| View/simulation distance | Per-player; Bedrock players get lower values; drops automatically when TPS falls and recovers afterwards |
| Chunks | Periodically requests unload of chunks away from all players to free RAM (skips force-loaded/ticketed chunks) |

Commands: `/crystalopt status`, `/crystalopt reload`, `/crystalopt clear` (permission `crystalopt.admin`).

Bedrock players are detected from their Floodgate UUID, so Floodgate must be installed for the Bedrock-specific distances to apply. Without it, everyone gets the Java values.

## Server-side tuning that matters more than any plugin

### JVM flags (biggest RAM/lag-spike win)

Use fixed heap sizes (same `-Xms` and `-Xmx`), and leave 1-2 GB of the machine's RAM free for the OS and Geyser.
Aikar's flags are the standard starting point; generate them for your heap size at https://flags.sh

### paper-world-defaults.yml (key names can vary slightly by version)

- `environment.optimize-explosions: true`
- `chunks.max-auto-save-chunks-per-tick: 8`
- `entities.spawning.alt-item-despawn-rate.enabled: true` (faster despawn for cobble, dirt, etc.)
- `entities.spawning.despawn-ranges`: shrink to roughly 32 soft / 64 hard
- `chunks.entity-per-chunk-save-limit`: limit `ender_pearl`, `arrow`, `experience_orb`

### spigot.yml / bukkit.yml

- `merge-radius.item: 3.5` and `merge-radius.exp: 4.0`
- `entity-activation-range`: animals 16, monsters 24, misc 8
- `spawn-limits`: set monsters/animals/ambient low (or 0 if you don't need mobs)
- `ticks-per.autosave`: raise if you see periodic lag spikes

### server.properties

- `view-distance=8` and `simulation-distance=5` as the ceiling (the plugin lowers per player)
- `network-compression-threshold=256`
- `sync-chunk-writes=false`

### Bedrock / Geyser

- Keep Bedrock view distance at 6 or below; Geyser translates every chunk, which costs CPU.
- In Geyser's `config.yml`, check your version's options for chunk caching and `scoreboard-packet-threshold`. Option names change between versions, so confirm against your Geyser version's docs.
- Run Geyser on the same machine or a low-latency link to the Java server.
- Bedrock crystal PvP will always feel slightly less crisp than Java because of translation and input differences. This plugin reduces server load, but it can't remove that gap.

## Tuning tips

- Still lagging in big fights? Set `explosions.block-damage: false`. Block calculations are the most expensive part of crystal spam.
- Players seeing blocks pop in? Raise `distances.java.view` / `bedrock.view` by 1, or lower `dynamic.max-reduction`.
- Use `spark` (`/spark profiler`) to confirm where lag actually comes from before tuning further.
