# project-reclaimer-egg
VibeCoded pterodactyl egg for project-reclaimer halo3

A Pterodactyl (and Pelican) egg for [Project Reclaimer](https://projectreclaimer.dev/host) Halo 3 dedicated servers.

- **Image:** `ghcr.io/pelican-eggs/yolks:wine_latest` (WineHQ stable, Xvfb, winetricks). The server is the official
  Windows build under Wine, the same way the official Docker image runs it.
- **Install:** downloads the server-only `project-reclaimer-dedicated-v<version>.exe` from the
  [official releases](https://github.com/ProjectReclaimer/project-reclaimer-releases/releases) and checks it against
  that release's `SHA256SUMS.txt`.
- **Game files are not included.** They come from your own Steam copy of Halo: The Master Chief Collection, and you upload them.

## Setup

1. **Import** [`egg-project-reclaimer.json`](egg-project-reclaimer.json): Admin → Nests → Import Egg.
2. **Create a server** from it.
   - **Allocation:** one port, any number (e.g. `49176`). Players connect on **UDP**. The same port on TCP is the remote console, which stays inside the container unless you change *RCON Address*.
   - **Memory:** 2 GB minimum, 3 GB is comfortable (about 0.6–0.9 GB in a game, up to 1.4 GB while a map loads, plus Wine).
   - **Disk:** about 10 GB for every multiplayer map (5.5 GB of maps, the Wine prefix, a 340 MB Forge library). Add about 4 GB for campaign co-op.
   - **CPU:** a full 63-player game wants about one core.
3. **Make the game files** on a Windows PC that has MCC with Halo 3 and its campaign installed:
   ```powershell
   .\project-reclaimer.exe dedicated init C:\reclaimer --from
   # only some maps: --maps "Guardian,Valhalla"    campaign co-op: --maps campaign
   ```
   Upload the **contents** of `C:\reclaimer\game` into the server's `game/` folder over SFTP, keeping the folders
   (`game/halo3/halo3.dll`, `game/halo3/maps/shared.map`, …). For **Flood Infection**, also copy
   `050_floodvoi.map` from MCC's `halo3\maps` into `game/halo3/maps`; `dedicated init` doesn't copy it.
4. **Start.** The first start installs the Visual C++ runtime into the Wine prefix and runs `dedicated init .`. That writes a
   commented `dedicated.toml`, `playlists/`, `content/` and `bans.json`. Then it runs `dedicated check` every
   15 seconds until the game files are in, so you can upload while it waits. The panel shows the server as running at
   `[egg] Starting Project Reclaimer`.
5. **Configure** `dedicated.toml` in the File Manager (server `name`, `playlist` or `map` + `game`), then restart. See
   the [configuration reference](https://projectreclaimer.dev/host).

## How the egg maps onto Project Reclaimer

- **One panel server runs one Reclaimer server:** the first `[[server]]` block in `dedicated.toml`
  (`RECLAIMER_DEDICATED_ONLY=1`). Other blocks are ignored. For more servers, create more panel servers.
- **Port:** every start rewrites the first `[[server]]`'s `port` to the allocation. Nothing else in the file is changed.
  The machine's discovery master (`[master]`, TCP 49175) needs no port: servers list themselves on the public
  masters anyway.
- **Variables:** each `RECLAIMER_DEDICATED_*` variable overrides the same setting in `dedicated.toml`, and an empty one
  keeps the file's value. The `[defaults]` ones (Max Players, Join Password, Anti-Cheat, …) lose to a setting in the
  `[[server]]` block itself. *Admins* adds to the file's list.
- **Other settings** from the [environment variable list](https://projectreclaimer.dev/host) (voice scopes,
  `LOAD_TIMEOUT`, `VPN_LISTS`, …) can be added as egg variables under the exact `RECLAIMER_DEDICATED_` name.
  A misspelt name stops the start.
- **Extra arguments:** anything after `reclaimer-start.sh` in the startup command is passed to `dedicated`,
  e.g. `exec bash ./reclaimer-start.sh --verbose`.

## Panel console

Set an **RCON Password** (8–128 characters) and restart. Each line typed in the console then goes to the server's remote
console through `dedicated rcon`, for example:

```
status    players    say <text>    kick <player> [reason]    ban <player> [time] [reason]    unban <target>
map <map>    mode <mode>    load "The Pit" "Capture the Flag"    endround    endgame    shuffle    admin add <player>
```

Quote names that have spaces. The console only answers once the server is up.

## Updating

- **Reinstall Server** (Settings tab) downloads the version in *Project Reclaimer Version* and rewrites
  `reclaimer-start.sh`. `dedicated.toml`, `game/`, `content/`, `playlists/`, `bans.json` and `data/` are kept.
- Or set **Auto Update** to `true`: the server installs each new stable release itself and restarts.
- Players can only join a server on exactly their version, so update when a release comes out.
- After a Steam update to MCC, copy the new engine file into `game/` once a Reclaimer release supports it.
  `dedicated check` names it.

## Notes

- Logs: the console, plus `data/<server>/server.log`.
- Workshop downloads need Steam, which the container doesn't have. Copy mod folders from
  `steamapps\workshop\content\976730` into `content/Mods/`, and name their maps by name rather than `workshop:` ID.
- **Stuck on "Waiting for 'dedicated check' to pass":** the lines above it say what's missing, usually
  `game/halo3/halo3.dll` in the wrong place.
- **Players see the server but can't join:** allow UDP on the allocation port in the node's firewall, and set
  *Public Address* if the node's outgoing IP isn't the allocation's.

The port sync and console bridge follow the approach of the community egg
[rcburke92/Pterodactyl-Egg-Halo-3-Project-Reclaimer](https://github.com/rcburke92/Pterodactyl-Egg-Halo-3-Project-Reclaimer).

Halo is a trademark of Microsoft. This egg is not affiliated with Microsoft, 343 Industries or Project Reclaimer.
