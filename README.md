# TrueMoozic for Kenshi

Boomboxes, walkmans and cassette tapes for **Kenshi** — a port of the Project Zomboid mod
*True Moozic*. Carry a boombox, load one of your own albums and let the whole squad
(and anyone else nearby) hear it.

> Requires [RE_Kenshi](https://github.com/BFrizzleFoShizzle/RE_Kenshi) and **KenshiLua**
> for playback and the player window. The items themselves are plain FCS data.

## Features

- **7 boomboxes** (3×2, loud — positional audio everyone nearby hears) and
  **8 walkmans** (2×2, private — only audible while the wearer is the selected character)
- **Battery Packs** (stack to 10) that fully recharge a device; charge is stored on the item
  and survives saves
- **Cassette Carrying Case** — backpack-slot container, 8×8 = 64 cassette-only slots,
  halves the weight of what's inside
- **One cassette per album folder** in `CustomMoozic/` — your own music, in `.ogg`, `.mp3`,
  `.wav` or `.flac`
- **MoozicBuilder.exe** — drag-and-drop album manager that writes the folder, the songlist
  and rebuilds the mod so vendors know about the new tape
- Devices and tapes are sold by general / tech vendors and bars, looted from houses, labs,
  ruins and bandit camps, and placed on shelves via placement groups
- Up to 4 devices playing at once; a device switches itself off if it leaves your squad's
  inventories (dropped, sold, stashed)

## Playing music

1. **Right-click** a Boombox or Walkman in any inventory window → **Interact**.
   (No hotkeys, nothing depends on Kenshi's dev settings. Do this from your own inventory,
   not while a trade screen is open — Kenshi treats right-click in a shop as buy/sell.)
2. The player window:

   | Control | What it does |
   | --- | --- |
   | cassette slot | click to load the next cassette the character carries, click again to eject |
   | `<` / `>` | flip through every carried cassette (inventory, backpack, case) |
   | POWER | on / off |
   | PREV | previous track (restarts the current one if it has played > 3 s) |
   | PLAY / PAUSE | pause and resume |
   | NEXT | next track; after the last track, the next cassette |
   | VOL − / + | per-device volume |
   | BATT … SWAP | battery gauge; SWAP fits a Battery Pack from the inventory |

Cassettes never leave the inventory — the player just reads them.

- Boombox audio is positioned on whoever carries it: full volume within 5 m, silent at 90 m,
  the camera is the listener.
- Battery: ~3 real hours per charge on a boombox, ~8 on a walkman. Flat = power off until you SWAP.
- Tracks follow the album's `songlist.txt` order when present (shuffled per cassette by default;
  `CFG.shuffle = false` for list order).

## Adding your own music

### The easy way — MoozicBuilder

Run `tools/MoozicBuilder.exe`.

- **New album** tab: drop audio files (or a folder) into the list, put them in play order, type a
  folder name after the fixed `TM_` prefix, optionally set a cassette name and price, press
  **BUILD**. The files are copied to `CustomMoozic/TM_<name>/`, a `songlist.txt` is written and
  `TrueMoozic.mod` is rebuilt.
- **Existing albums** tab: select an album → **DELETE** removes the folder and rebuilds the mod
  so the cassette leaves the shops cleanly. **Rebuild mod only** re-syncs after hand edits.
- Afterwards: **restart Kenshi, then IMPORT the save** so vendors restock.

`MoozicBuilder.exe --rebuild` does a silent rebuild (log in `tools/MoozicBuilder.log`).
Keep the exe inside `tools/` — it loads `build_mod.py` from there at run time.

### By hand

One sub-folder = one cassette:

```
mods/TrueMoozic/CustomMoozic/<Album>/
    songlist.txt     optional - play order, one title per line
                     ("Artist - Title — 3:27"; header/===== lines and the
                     duration suffix are ignored)
    cassette.txt     optional - line 1 = cassette name, line 2 = price
    *.ogg *.mp3 *.wav *.flac
```

Without `cassette.txt` the folder name becomes the tape name (`TM_` stripped, underscores and
CamelCase spaced out): `TM_SpegHeddy` → **Cassette: Speg Heddy**.

| Change | What you need to do |
| --- | --- |
| new songs in an **existing** folder | nothing — `TrueMoozic.rescan()` in the console or reload the save |
| a **new** folder | MoozicBuilder or `python tools/build_mod.py`, restart Kenshi, import the save |
| delete a folder | MoozicBuilder or `python tools/build_mod.py` (the tape vanishes from shops) |

Cassette ids are derived from the folder name, so rebuilding never re-labels tapes already in a save.

## Installation

1. Copy the `TrueMoozic` folder to `<Kenshi>/mods/`.
2. Make sure RE_Kenshi and KenshiLua are installed and working.
3. Enable **TrueMoozic** in the Kenshi launcher (or add it to `data/mods.cfg`).
4. Import your save (or start a new game) so vendors re-roll their stock.

Startup line in `mods/KenshiLua/plugin/KenshiLua.log`:

```
[TrueMoozic] v0.4.0: 7 boomboxes, 8 walkmans, N cassettes (N custom), audio=true gui=true
```

## Console (KenshiLua, Ctrl+Shift+L)

| Command | |
| --- | --- |
| `TrueMoozic.status()` | every registered device |
| `TrueMoozic.inspect()` | what the selected character carries / what the mod recognises |
| `TrueMoozic.albums()` | every cassette and the songs found for it |
| `TrueMoozic.rescan()` | re-read album folders after adding songs |
| `TrueMoozic.open()` | open the selected character's player |
| `TrueMoozic.setVolume(0.5)` / `TrueMoozic.stopAll()` | |
| `TrueMoozic.debug(20)` | re-arm detailed click logging |

## Troubleshooting

TrueMoozic writes its own log: `mods/TrueMoozic/TrueMoozic.log` (reset every game start).

- no `world running` line → the per-frame hook never fired (KenshiLua not active / mod not enabled)
- `right-click detected` never appears → the game is **paused** (the hook only runs while time moves)
- `windows=0` / `sectionWithMouse=nil` → the inventory window could not be queried; open an issue
  with the log attached

## Layout

```
TrueMoozic.mod                 FCS data: items + vendor/loot list additions
icons/                         inventory icons (converted from the PZ mod)
audio/sfx/                     boombox start/stop, empty-tape hiss
CustomMoozic/<album>/          your albums (one folder = one cassette)
native/TrueMoozicAudio.dll     miniaudio playback engine (LuaJIT FFI)
native/truemoozic_audio.c      its source (+ miniaudio.h, stb_vorbis.c)
scripts/init/truemoozic.lua    KenshiLua player (auto-loaded)
scripts/truemoozic_gui.lua     MyGUI player window + right-click menu
scripts/truemoozic_songs.lua   album folder scanner (songlist.txt aware)
scripts/truemoozic_audio.lua   FFI bridge to the DLL
scripts/truemoozic_items.lua   GENERATED: ids / names / icons / audio folders
tools/MoozicBuilder.exe        album manager GUI
tools/moozicbuilder.py         its source; tools/build_exe.py freezes it
tools/build_mod.py             regenerates TrueMoozic.mod + the Lua manifest
tools/kenshi_mod.py            .mod binary reader/writer
tools/smoke_lua.py             runs the Lua in a stub world (no Kenshi) to catch errors
tools/build_native.cmd         rebuilds the DLL (MSVC)
```

## Building from source

```powershell
python tools/build_mod.py          # rebuild TrueMoozic.mod (icons are reused; TM_REGEN_ICONS=1 re-converts them)
python tools/smoke_lua.py          # Lua sanity test, prints SMOKE OK
python tools/build_exe.py          # freeze MoozicBuilder.exe (needs pyinstaller, tkinterdnd2, pillow)
tools\build_native.cmd             # rebuild the audio DLL (needs MSVC)
```

Names, prices and spawn weights live in the `BOOMBOXES` / `WALKMANS` / `VENDORS` /
`PLACEMENT_GROUPS` tables at the top of `build_mod.py`; playback settings (volume, range,
battery life, shuffle, menu timeout) in the `CFG` table at the top of `scripts/init/truemoozic.lua`.

## How it works

- Items and vendor/loot additions are ordinary FCS records written by `kenshi_mod.py`; vanilla
  string ids are resolved by *name* at build time so the mod survives game updates.
- Playback is a small miniaudio DLL driven through LuaJIT FFI; boombox sound is 3D-positioned on
  the carrier every frame, the camera is the listener.
- The player window and right-click menu are MyGUI widgets created from Lua; input is polled
  per frame through KenshiLua's `onCharacterUpdate` hook.

## Roadmap

Vinyl, a HiFi building, NPC vendors with their own stock.

## Credits

- Original *True Moozic* mod for Project Zomboid (icons and foley sounds)
- [miniaudio](https://miniaud.io/) and [stb_vorbis](https://github.com/nothings/stb) (public domain / MIT)
- RE_Kenshi and KenshiLua for making scripting Kenshi possible

Example albums bundled in `CustomMoozic/` are placeholders — replace them with your own music.
No copyrighted music is distributed with this repository.
