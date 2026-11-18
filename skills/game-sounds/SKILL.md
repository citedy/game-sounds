---
name: game-sounds
description: Manage game sound effects — change volume, mute/unmute, switch sound packs, list available sounds
user_invocable: true
---

# /game-sounds — Game Sound Manager

Manage your game sound effects for Claude Code.

## Usage

When the user invokes `/game-sounds`, parse their arguments and execute the appropriate action:

### Commands

- `/game-sounds` — Show current config (active pack, volume, rotation, enabled events) and whether sounds are muted
- `/game-sounds volume <0.0-1.0>` — Set volume level
- `/game-sounds pack <name>` — Switch active sound pack (clears rotation)
- `/game-sounds list` — List available sound packs and their sounds
- `/game-sounds rotation add <pack>` — Add a pack to the session rotation list
- `/game-sounds rotation remove <pack>` — Remove a pack from the rotation list
- `/game-sounds rotation` — Show the current rotation list
- `/game-sounds rotation clear` — Clear rotation (reverts to single active_pack)
- `/game-sounds toggle <event>` — Enable/disable a specific event category
- `/game-sounds test [category]` — Play a test sound from the given category (default: session-start)
- `/game-sounds mute` — Silence all sounds until unmuted
- `/game-sounds unmute` — Turn sounds back on

## Implementation

Read and modify the config file at `$CLAUDE_PLUGIN_ROOT/config.json`.

For **volume**: Update the `volume` field (float 0.0 to 1.0).
For **pack**: Update `active_pack` field and set `pack_rotation` to `[]`. Verify the pack directory exists in `$CLAUDE_PLUGIN_ROOT/sounds/`.
For **rotation add**: Append pack name to the `pack_rotation` array (if not already present). Verify the pack exists.
For **rotation remove**: Remove pack name from the `pack_rotation` array.
For **rotation clear**: Set `pack_rotation` to `[]`.
For **rotation** (no subcommand): Display the current `pack_rotation` list.
For **toggle**: Flip the boolean in `enabled_events.<event>`.
For **test**: First run `test -e "$HOME/.local/state/game-sounds/muted" && echo muted || echo unmuted`. If it prints `muted`, sounds are muted: say so (and that `/game-sounds unmute` turns them back on) instead of playing a test. If it prints `unmuted`, run `bash $CLAUDE_PLUGIN_ROOT/scripts/play-sound.sh <category>`.
For **list**: Scan `$CLAUDE_PLUGIN_ROOT/sounds/` directories and list packs with file counts.
For **mute** / **unmute**: To mute, run `mkdir -p "$HOME/.local/state/game-sounds" && touch "$HOME/.local/state/game-sounds/muted"`; to unmute, run `rm -f "$HOME/.local/state/game-sounds/muted"`. The mute flag is that file, not a config.json field: it lives outside the plugin directory so plugin updates don't reset it.
For **status** (no arguments): Show the config.json settings, then run the same check as **test** (`test -e "$HOME/.local/state/game-sounds/muted" && echo muted || echo unmuted`) and report muted when it prints `muted`.

When `pack_rotation` is non-empty, each new Claude Code session randomly picks one pack from the list for the entire session. Use the Bash tool to read/write config.json and the Read tool to display current settings.

### Example responses

**Status**: "🎮 Game Sounds: Warcraft pack, volume 0.5, rotation: [mario, zelda, starcraft]"
**Status (muted)**: "🎮 Game Sounds: Warcraft pack, volume 0.5, rotation: [mario, zelda, starcraft], 🔇 muted"
**Volume change**: "🔊 Volume set to 0.3"
**Pack switch**: "⚔️ Switched to starcraft pack (rotation cleared)"
**Rotation add**: "🎲 Added mario to rotation (now: mario, zelda)"
**Rotation remove**: "🎲 Removed mario from rotation (now: zelda)"
**Mute**: "🔇 Sounds muted (run /game-sounds unmute to turn them back on)"
**Unmute**: "🔊 Sounds unmuted"
