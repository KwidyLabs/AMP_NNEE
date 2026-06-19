# AMP Generic Module template — Neverwinter Nights: Enhanced Edition (704450)

A hand-built CubeCoders AMP "Generic Module" configuration for running a
**Neverwinter Nights: Enhanced Edition** dedicated server (NWServer), since AMP
does not ship a built-in template for it. Built against the
[Generic module documentation](https://github.com/CubeCoders/AMP/wiki/Configuring-the-'Generic'-AMP-module)
and cross-checked against existing templates in
[CubeCoders/AMPTemplates](https://github.com/CubeCoders/AMPTemplates).

## Files
- `nwn-ee.kvp` — the main Application/Console/Meta config
- `nwn-eeconfig.json` — the settings shown in the AMP UI, mapped to nwserver CLI flags
- `nwn-eeupdates.json` — SteamCMD download + first-time folder setup
- `nwn-eeports.json` — port reservation (UDP 5121)
- `manifest.json` — repository descriptor used when deploying via a Git repo

## Game-specific notes

1. **No anonymous SteamCMD download.** NWN:EE has no separate "dedicated
   server" Steam App ID — it is the same app ID (704450) as the full client,
   and Steam requires the account doing the download to actually own the game.
   `App.SteamUpdateAnonymousLogin=False` is set accordingly, so AMP's
   update/install step prompts for real Steam credentials (and a Steam Guard
   code) rather than logging in anonymously.

2. **32-bit Linux binary.** The Linux server (`bin/linux-x86/nwserver-linux`)
   is 32-bit x86. The simplest way to run it is as a *container* instance:
   AMP pulls `cubecoders/ampbase:debian`, the same base image CubeCoders use
   for other 32-bit Linux titles (e.g. Garry's Mod, Counter-Strike: Source),
   so the required i386 multiarch libraries are already present — no manual
   library install needed. On bare metal without a container you would install
   them yourself first, e.g. on Debian/Ubuntu:
   `dpkg --add-architecture i386 && apt update && apt install -y libc6:i386 libstdc++6:i386 lib32gcc-s1`.

3. **Only one port to forward:** UDP 5121 (configurable). This single UDP port
   is what needs forwarding for direct-connect multiplayer.

4. **Module Name = the module's name, not its filename.** In the **Module
   Name** setting, enter the name *without* the `.mod` extension — e.g. for
   `MyWorld.mod`, enter `MyWorld`. Entering `MyWorld.mod` makes nwserver look
   for `MyWorld.mod.mod` and fail with "Unable to load module". It's
   case-sensitive. Upload the `.mod` into `userdata/modules` (and any required
   haks into `userdata/hak`, custom `.tlk` into `userdata/tlk`) *before*
   starting. The field defaults to `ChangeMe` so the first SteamCMD install can
   run past the required-field check — replace it with your real module name
   before you start the server.

5. **AppReadyRegex** matches `Server: Module loaded` — the line the server
   prints once the module has finished loading and players can connect
   (confirmed from real `nwserver` output:
   `Server: Loading module "...".......... Server: Module loaded`). It is left
   unanchored on purpose: if a build prefixes a timestamp on stdout, an
   anchored `^...$` would silently never match. You can tighten it (e.g.
   `^Server: Module loaded$`) after watching one real boot.

6. **No player list.** There is no documented join/leave console line format
   for this build, so `UserJoinRegex`/`UserLeaveRegex` are left blank and AMP
   will not show a live player list — everything else (start/stop/console/
   settings) works normally. To add one: start the instance, connect/disconnect
   a client, copy the exact console lines, and wrap the player name in a .NET
   named group `(?<username>...)` (escaping regex metacharacters in the literal
   text). Note that vanilla `nwserver` may not print connects to stdout at
   default verbosity — if nothing appears, a player list is not possible
   without NWNX:EE, which is a server limitation rather than a template issue.

## Deploying into AMP

### Option A — configuration repository (recommended)

In AMP: **Configuration → Instance Deployment → Add Configuration Repository**,
enter:

```
KwidyLabs/AMP_NNEE:main
```

Click **Fetch**, then refresh the browser. "Neverwinter Nights: Enhanced
Edition" will appear under the Generic module list when creating a new instance.

> AMP's configuration-repository fetcher resolves the `user/repo:branch`
> shorthand against **github.com**. It does not fetch from self-hosted Git
> servers (Gitea/GitLab) by full URL — which is why this template is published
> on GitHub.

### Option B — local folder (no Git)

AMP can read templates from a local folder under the controller instance:
`.../Plugins/ADSModule/DeploymentTemplates/`. Create a subfolder named with the
prefix `LOCAL` and suffix `-main` (e.g. `LOCAL-main`), drop these files inside
it (owned by AMP's service user), and restart the controller (ADS) instance so
it re-scans. Note that local-folder discovery is less reliable across AMP
versions; Option A is the dependable path.

## After it's running

Use AMP's File Manager / SFTP to reach `userdata/` inside the instance
directory — that is where `modules/`, `hak/`, and (after the first successful
boot) NWN's other auto-created folders (`servervault/`, `tlk/`, `database/`,
etc.) live, kept separate from the SteamCMD-managed `704450/` game files so a
verify/update never touches your saves or module content.
