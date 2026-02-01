# Nullbind Logic Documentation

## Overview

The CSAFAP config package implements **nullbind** (also known as **Snap-Tap**) functionality for CS2, which modifies movement input behavior to automatically cancel opposing directional inputs. This feature is particularly useful for advanced movement techniques like counter-strafing and quick directional changes.

## What is Nullbind/Snap-Tap?

Nullbind is a movement input system where pressing an opposing directional key automatically nullifies the original key's input. For example:
- While holding **D** (moving right), pressing **A** immediately stops the rightward movement and starts moving left
- While holding **W** (moving forward), pressing **S** immediately stops forward movement and starts moving backward

This helps eliminate the human error of not releasing the initial strafe key quickly enough during counter-strafing.

## Implementation Architecture

The config package provides **two different implementations** of nullbind:

### 1. Ticker Version (AveYo's Implementation)
**File:** `CSAFAP/addons/AveYo.cfg`

This is the recommended implementation that uses CS2's ticker mechanism with the `.vtest` file.

**Requirements:**
- Launch option: `-testscript "../../csgo/cfg/CSAFAP/addons/.vtest"`
- Binds: `bind W +nullW`, `bind A +nullA`, `bind S +nullS`, `bind D +nullD`

**How it works:**
```cfg
// Example for W/S axis (forward/backward)
alias +nullW alias W! W+1; alias -nullW alias W! W-1
alias W+1 "W<; alias W! W+2"
alias W+2 "alias W!; alias S< forward -9999 f u; alias S> +forward; +forward"
alias W-1 "forward -9999 f u; alias W! W-2"
alias W-2 "alias W!; alias S< +back; alias S>; W>"
```

**Key mechanism:**
- Uses multi-stage aliases that trigger sequentially via the ticker
- When pressing W: activates forward movement AND sets up S to cancel it
- When pressing S while W is held: the `S<` alias cancels forward movement first, then applies backward movement
- Uses the command format `forward -9999 f u` to reliably cancel movement states

### 2. No-Ticker Version (Legacy Implementation)
**File:** `CSAFAP/movement/nullWASD_noticker.cfg`

This older implementation doesn't require the ticker mechanism but is less reliable.

**How it works:**
```cfg
// Uses rightleft and forwardback commands with sensitivity manipulation
joy_side_sensitivity 1
joy_forward_sensitivity 1

alias "+a" "rightleft -1 0 0; alias +d ad"
alias "ad" "rightleft 1 0 0; alias -d add; alias -a ada"
alias "-a" "rightleft 0 0 0; alias +d right1; alias -d right2; alias -a left2; alias +a left1"
```

**Key mechanism:**
- Uses `rightleft` and `forwardback` commands to control movement
- Creates complex alias chains that redefine key behaviors dynamically
- When a key is pressed, it redefines the opposing key's behavior
- Less reliable than ticker version but doesn't require special launch options

## User Interface Integration

### Enabling/Disabling Nullbind

The config provides a seamless toggle system through the crosshair wheel:

**Default keybind:** `K` (bound to `+crosshair_action`)

**File flow:**
1. **Press K (first time):** Loads movement labels → shows radio wheel
2. **Release K:** Switches to command mode
3. **Press K (second time):** Loads movement commands → executes selected tile
4. **Release K:** Closes radio wheel and resets

**Radio wheel tile:** Tab 1, Position 4
- **When nullbind is OFF:** Shows "#CFG_MOVEMENT_NULL_ON" → executes `nullWASD.cfg`
- **When nullbind is ON:** Shows "#CFG_MOVEMENT_NULL_OFF" → executes `default.cfg`

### State Management

The system tracks nullbind state using dynamic aliases:

```cfg
// In nullWASD.cfg (when enabled)
alias null_labels exec_null_on_labels
alias null_cmd exec_null_on_cmd

// In default.cfg (when disabled)
alias null_labels exec_null_off_labels
alias null_cmd exec_null_off_cmd
```

This allows the radio wheel to display the correct state and toggle appropriately.

## File Structure

```
CSAFAP/
├── movement/
│   ├── nullWASD.cfg                 # Ticker-based nullbind (recommended)
│   ├── nullWASD_noticker.cfg        # Legacy no-ticker nullbind
│   ├── default.cfg                  # Resets to normal WASD movement
│   ├── null_on_labels.cfg           # Radio tile text when enabled
│   ├── null_off_labels.cfg          # Radio tile text when disabled
│   ├── null_on_cmd.cfg              # Command to disable (switch to default)
│   ├── null_off_cmd.cfg             # Command to enable (load nullWASD)
│   └── movement_labels.cfg          # Movement wheel label configuration
├── addons/
│   ├── AveYo.cfg                    # Contains nullbind implementation
│   └── loader.cfg                   # Loads all components
└── logic.cfg                        # Main logic and wheel bindings
```

## Configuration Options

### Launch-Time Loading (MAIN.cfg)

Users can enable nullbind at game launch by uncommenting:

```cfg
alias load_null_binds load_null      // Null binds (ticker version)
```

This will automatically apply nullbind when the game starts.

### Manual Toggle (In-Game)

Press the crosshair wheel key (default `K`) and select the nullbind tile to toggle between:
- **Default movement:** Standard WASD with no input cancellation
- **Nullbind movement:** Automatic opposing input cancellation

## Technical Details

### Movement Reset on Toggle

When switching between default and nullbind, the configs properly reset movement states:

```cfg
// From default.cfg
bind w +forward;
bind a +left;
bind s +back;
bind d +right;
bind mouse_x yaw;
bind mouse_y pitch;
rightleft 0 0 0;
forwardback 0 0 0;
```

This ensures no lingering movement commands remain active.

### Compatibility with Other Features

The nullbind system is designed to work alongside:
- **Jumpthrow binds:** Temporarily reverts to default movement for reliable throws
- **Auto line-ups:** Works with sensitivity adjustments
- **Follow-recoil configs:** Independent systems that don't interfere
- **Practice mode:** Fully compatible

### Known Limitations

From the FAQ (Q8 and Q12 in README.md):

1. **Nullbind behavior:** Only corrects not releasing the initial strafe key; you still need proper timing for counter-strafe release
2. **Jumpthrow compatibility:** W-bind in nullWASD may interfere with jumpthrows. Recommendation is to use default movement for jumpthrows
3. **Not automatic counter-strafe:** This is NOT an auto counter-strafe config (which was patched). It only nullifies opposite inputs.

## Benefits and Use Cases

### Best for:
- **Fast jiggle peeks:** Rapidly pressing A-D-A-D while holding one key
- **Pistol round movement:** Quick directional changes and AD spam
- **Eliminating human error:** Prevents holding both A and D simultaneously

### Not as useful for:
- **Perfect counter-strafing:** Still requires proper timing to stop exactly
- **Regular competitive play:** Standard movement is often sufficient

## Security and Legality

According to the README FAQ:

- ✅ **Allowed on MM/Premier:** Yes, uses only legal in-game commands
- ✅ **Allowed on FACEIT PUGs:** Confirmed by FACEIT Community Manager ([source](https://www.reddit.com/r/FACEITcom/comments/1ea2tgz/comment/leim41n/))
- ❌ **Not allowed in ESL/ESEA:** Prohibited due to alias command restrictions

The nullbind feature will NOT trigger a kick from official servers.

## Comparison: Ticker vs No-Ticker

| Feature | Ticker Version | No-Ticker Version |
|---------|---------------|-------------------|
| **Reliability** | High | Moderate |
| **Requirements** | Launch option + .vtest file | None |
| **Implementation** | AveYo's sequential aliases | rightleft/forwardback commands |
| **File** | `AveYo.cfg` | `nullWASD_noticker.cfg` |
| **Recommended** | ✅ Yes | Use only if ticker unavailable |

## Code Credit

- **Ticker implementation:** AveYo ([GitHub](https://github.com/AveYo/Gaming/tree/main/CS2))
- **Config integration:** FNScence (CSAFAP package author)

## Summary

The nullbind logic in CSAFAP is a sophisticated system that:
1. Provides two implementation methods (ticker-based and legacy)
2. Integrates seamlessly with the radio wheel UI
3. Maintains proper state tracking for enable/disable toggling
4. Works alongside other config features without conflicts
5. Offers legal movement enhancement for competitive play

The ticker-based version (AveYo's implementation) is the recommended approach due to its reliability and is automatically loaded when the required launch option is present.
