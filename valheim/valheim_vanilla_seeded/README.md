# Valheim (Seeded)

A brutal exploration and survival game for 1-10 players, set in a procedurally-generated purgatory inspired by viking culture. Battle, build, and conquer your way to a saga worthy of Odin’s patronage!

This egg is identical to `valheim_vanilla`, but adds a `World Seed` variable so you can pin the seed used to generate the world instead of getting a random one.

<https://store.steampowered.com/app/892970/Valheim/>

## Server Ports

| Port  | default |
|-------|---------|
| Game  | 2456    |
| Query | 2457    |

## World Seed

The `World Seed` variable (`WORLD_SEED`) is passed to the server as `-worldseed`. It only takes
effect the **first time** a world with the configured `World Name` is generated — once a world's
save files exist, changing `WORLD_SEED` has no effect on it.

To generate a new world with a different seed under the same `World Name`, delete that world's
save files from the `worlds_local` folder before restarting the server:

- `<World Name>.db`
- `<World Name>.fwl`

On next boot, the server will regenerate a fresh world using the current `WORLD_SEED` value.
Leave `WORLD_SEED` blank for a random seed (stock Valheim behavior).
