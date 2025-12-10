[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Fx4dm%2FHue-Tap-Dial-ZHA-Multi-Light-Edition%2Fmain%2Fhue_tap_dial_multi_light.yaml)

After struggling with this concept for a few days trying to troubleshoot why my LEDs were dimming regardless of whether I turned my Hue dial CW or CCW, I gave up and turned to Claude.  Claude basically rewrote the entire blueprint, found a bunch of optimizations, and generated almost all of this readme document.  This is my first commit on Github and I probably won't be updating it.  Feel free to fork it yourself or attach it to Claude and make it your own.  Maybe in the future I'll add the ability to toggle between hue and color temperature.

Also, I don't have any other Hue products, so I have no idea how smooth the native hue interface is.  I'm guessing this blueprint is worse.

# Hue Tap Dial – ZHA Multi-Light Edition

A comprehensive Home Assistant blueprint that allows you to control up to 4 LED lights with a single Philips Hue Tap Dial switch. Each button can be configured independently for either light control (hue/brightness) or custom automation actions.

## Features

### Dual Control Modes Per Button
Each of the 4 buttons can operate in one of two modes:

**Light Mode:**
- Single press: Turn on light + enter Hue mode (light blinks once)
- Double press: Enter Brightness mode (light blinks twice)
- Long press: Turn off light
- Dial rotation: Adjust hue or brightness of the selected light
- Triple/Quadruple/Quintuple press: Custom actions (optional)

**Button Mode:**
- Single/Double/Long press: Custom actions
- Triple/Quadruple/Quintuple press: Custom actions
- Dial rotation: No effect in this mode

### Visual Feedback
When you activate a light by pressing its button, the light blinks to confirm selection:
- **1 blink** = Hue mode active (adjust color)
- **2 blinks** = Brightness mode active (adjust brightness)

### Smart Rotation Control
The dial automatically controls whichever light button was last pressed. Each light maintains its own hue value, so you can:
- Adjust Light 1's color
- Switch to Light 2 and adjust its color
- Return to Light 1 and make fine adjustments from where you left off

### Customizable Sensitivity
Adjust how fast the dial responds:
- **Hue Sensitivity**: 1.0 - 5.0 (default: 2.0)
- **Brightness Sensitivity**: 1.0 - 5.0 (default: 2.0)

### Performance Optimized
- Uses queued mode for smooth operation
- Zero transition time for instant response
- Supports up to 10 simultaneous actions

---

## Requirements

### Hardware
- **Philips Hue Tap Dial Switch paired directly to ZHA** (Model: RDM002)
- **Zigbee LED Light Strips** (must support RGB color and brightness control)
- **Zigbee Coordinator** (USB dongle or built-in)
- **Recommended**: Zigbee router/repeater for improved reliability - The color changer can get really laggy unless you have a mains powered repeater nearby.  I use third reality Zigbee switches since they're cheap and useful, but any repeater will work

### Software
- **Home Assistant** (2023.1 or newer recommended)
- **ZHA (Zigbee Home Automation)** integration enabled
- Both the Hue Tap Dial and LED lights must be paired to your ZHA network directly.  I don't think this works if either light or the tap dial switch are paired to the Hue bridge

### Helper Entities
You must create 5 helper entities before using this blueprint: (See step 2 below)

1. **One Dropdown Helper** (shared mode selector)
2. **Four Number Helpers** (individual hue values for each button)

---

## Installation

### Step 1: Import the Blueprint

   - Go to **Settings → Automations & Scenes → Blueprints**
   - Click **Import Blueprint** (bottom right)
   - Paste this URL:
     ```
     https://github.com/x4dm/Hue-Tap-Dial-ZHA-Multi-Light-Edition/blob/main/Hue%20Tap%20Dial%20%E2%80%93%20ZHA%20Multi-Light%20Edition
     ```
   - Click **Preview** then **Import Blueprint**

### Step 2: Create Helper Entities

Before creating your automation, you must create 5 helper entities. These store the mode (hue/brightness) and individual hue values for each light.

#### Create the Shared Mode Dropdown:

1. Go to **Settings → Devices & Services → Helpers**
2. Click **Create Helper** → **Dropdown**
3. Configure:
   - **Name**: `Hue Dial Mode` (Or something similar)
   - **Options**: Add these two options (one per line):
     ```
     hue
     brightness
     ```
   - **Icon**: `mdi:palette` (optional)
4. Click **Submit**

#### Create the Individual Hue Value Helpers:

Repeat these steps **4 times** (once for each button):

1. Go to **Settings → Devices & Services → Helpers**
2. Click **Create Helper** → **Number**
3. Configure:
   - **Name**: `Hue Dial Button X Helper` (replace X with 1, 2, 3, or 4)
   - **Minimum**: `0`
   - **Maximum**: `360`
   - **Step size**: `1`
   - **Display mode**: Slider
   - **Unit of measurement**: `°`
   - **Icon**: `mdi:palette` (optional)
4. Click **Submit**

After completing these steps, you should have these 5 helpers:
- `input_select.hue_dial_mode`
- `input_number.hue_dial_button_1_helper`
- `input_number.hue_dial_button_2_helper`
- `input_number.hue_dial_button_3_helper`
- `input_number.hue_dial_button_4_helper`

### Step 3: Create Automation from Blueprint

1. Go to **Settings → Automations & Scenes → Automations**
2. Click **Create Automation** → **Use a blueprint**
3. Select **Hue Tap Dial – ZHA Multi-Light Edition**
4. Configure the automation:

#### Required Settings:
- **Hue Tap Dial**: Select your RDM002 device
- **Hue Dial Mode**: Select `input_select.hue_dial_mode`
- **Hue Dial Button 1-4 Helper**: Select the corresponding helper for each button

#### Button Configuration (repeat for each button):
- **Button X Mode**: Choose "Light Mode" or "Button Mode"
- **Button X - LED Light**: Select the light to control (only used in Light Mode)
- **Button X - Single/Double/Long Press Action**: Configure custom actions (Button Mode) or leave empty
- **Button X - Triple/Quadruple/Quintuple Press Action**: Optional custom actions (available in both modes)

#### Optional Settings:
- **Hue Sensitivity**: 1.0 - 5.0 (default: 2.0)
- **Brightness Sensitivity**: 1.0 - 5.0 (default: 2.0)

5. Click **Save** and name your automation

---

## Configuration Guide

### Understanding Button Modes

#### Light Mode
When a button is set to Light Mode:
- It controls one specific LED light
- Single press: Turn on + Hue mode (light blinks once)
- Double press: Brightness mode (light blinks twice)  
- Long press: Turn off
- Dial rotation adjusts the selected light's hue or brightness
- Triple/Quadruple/Quintuple presses can still trigger custom actions

**Use Case**: Direct control of your LED strips

#### Button Mode  
When a button is set to Button Mode:
- All press types (single, double, long, triple, quadruple, quintuple) trigger custom actions
- Dial rotation has no effect
- No light control functionality

**Use Case**: Trigger scenes, scripts, or other automations

### Sensitivity Settings

Both sensitivity settings use the same scale: **1.0 to 5.0** in 0.1 increments.

**Hue Sensitivity** (Color Changes):
- 1 is very slow/precise
- 5 makes it easy to find a color, but you might not get the exact color you're looking for if you're super picky

**Brightness Sensitivity**:
- 1 is very slow/precise
- 5 makes it easy to turn up/down quickly, but most LEDs have poor brightness linearity below ~10, so if you're trying to get an exact brightness of 2 or 3, you won't get that here.

**Tips**:
- Start with default (2.0) for both and adjust based on preference
- Lower values provide more precision but require more dial rotation
- Higher values are faster but less precise
- You can set different sensitivities for hue vs brightness based on your workflow

### Custom Actions (Advanced)

Each button supports up to 6 press types:
- Single press (short press and release)
- Double press (two quick presses)
- Long press (press and hold)
- Triple press (three quick presses)
- Quadruple press (four quick presses)
- Quintuple press (five quick presses)

In **Light Mode**, only triple/quadruple/quintuple presses are available for custom actions (single/double/long are reserved for light control).

In **Button Mode**, all six press types are available for custom actions.

**Example Custom Actions**:
- Toggle other lights or light groups
- Activate scenes (movie mode, party mode, etc.)
- Control media players (play/pause, volume)
- Trigger scripts or other automations
- Adjust thermostat settings
- Send notifications

---

## Usage Examples

### Example 1: Single Light Control
**Setup**: Button 1 in Light Mode controlling your desk LED strip

**Usage**:
1. Single press Button 1 → Desk light turns on, blinks once (Hue mode)
2. Rotate dial → Change desk light color
3. Double press Button 1 → Desk light blinks twice (Brightness mode)
4. Rotate dial → Adjust desk light brightness
5. Long press Button 1 → Desk light turns off

### Example 2: Multi-Light Setup
**Setup**: 
- Button 1: Living room LED strip (Light Mode)
- Button 2: Bedroom LED strip (Light Mode)
- Button 3: Kitchen LED strip (Light Mode)
- Button 4: Custom scenes (Button Mode)

**Usage**:
1. Single press Button 1 → Living room light on, blinks once
2. Rotate dial → Adjust living room color to warm orange (stored at 30°)
3. Single press Button 2 → Bedroom light on, blinks once
4. Rotate dial → Adjust bedroom color to cool blue (stored at 240°)
5. Single press Button 1 → Back to living room (still at 30° orange)
6. Rotate dial → Fine-tune living room color from where you left off

### Example 3: Mixed Mode Setup
**Setup**:
- Buttons 1-2: Light Mode for main lights
- Button 3: Button Mode with scene triggers
  - Single press → "Movie Time" scene
  - Double press → "Bright" scene
  - Long press → All lights off

**Usage**: Direct light control on buttons 1-2, quick scene access on button 3

---

## Troubleshooting

### Issue: Blueprint won't import or shows errors

**Solution**: 
- Ensure you're using Home Assistant 2023.1 or newer
- Check that ZHA integration is installed and working
- Try re-importing with the raw GitHub URL
- Check Home Assistant logs for specific error messages
- If all else fails, copy the YAML blueprint to your \config\blueprints\automation path

### Issue: Dial rotation doesn't work

**Checklist**:
1. Is the button configured in Light Mode? (Dial only works in Light Mode)
2. Did you select a light for that button?
3. Did you create all 5 helper entities and map them when importing the blueprint?
4. Is the light on? The dial only affects lights that are already on
5. Did you press the button first to activate that light?

**Diagnostic Steps**:
- Check helper entities exist: Settings → Devices & Services → Helpers
- Verify light supports color: Check light attributes in Developer Tools → States
- Test helper manually: Try changing `input_select.hue_dial_mode` and see if dial behavior changes

### Issue: Colors don't match between lights

**Explanation**: Different LED manufacturers interpret hue values slightly differently. A hue of 120° might appear lime green on one strip and forest green on another.

**Solution**: This is normal behavior. Adjust each light individually to your preferred color.

### Issue: Lights don't blink when selecting them or blink too many times

**Possible causes**:
1. **Light doesn't support on/off commands quickly**: Some Zigbee lights have minimum on-times
2. **Zigbee mesh congestion**: Too many commands sent at once
3. **Light is already in the desired state**: Try turning the light off first, then press the button

**Solution**: 
- Ensure lights are Zigbee (not Wi-Fi based)
- Add more Zigbee routers to strengthen mesh
- Check automation traces to confirm blink commands are being sent

### Issue: Double press activates twice

**Explanation**: This can happen if the double press detection timing doesn't match the dial's timing.

**Solution**: This is a limitation of the Hue Tap Dial hardware. Try pressing slightly faster or slower to find the sweet spot.

### Issue: Lag when rotating dial

**Solutions**:
1. **Add Zigbee routers**: Place mains-powered Zigbee devices between the dial and coordinator
2. **Check system resources**: High CPU/RAM usage can slow automation processing
3. **Reduce other automations**: Temporarily disable other automations to test
4. **Lower sensitivity**: Try reducing both sensitivities for smaller steps
5. **Check Zigbee signal strength**: Settings → Devices & Services → ZHA → [Your Dial] → Check "LQI" value

### Issue: Helper entities not found

**Error Message**: "Entity not found: input_select.hue_dial_mode"

**Solution**:
1. Go to Settings → Devices & Services → Helpers
2. Verify all 5 helpers exist with the **exact** names specified in Step 2
3. Entity IDs must match exactly (case-sensitive):
   - `input_select.hue_dial_mode`
   - `input_number.hue_dial_button_1_helper`
   - `input_number.hue_dial_button_2_helper`
   - `input_number.hue_dial_button_3_helper`
   - `input_number.hue_dial_button_4_helper`
4. If names don't match, either rename the helpers or update the automation configuration

### Issue: Hue mode resets to 0° when switching lights

**This is expected behavior** if you haven't rotated the dial for that light yet. Each button's helper starts at 0° (red) by default. Once you adjust a light's color, that value is stored in its helper.

**Workaround**: Set initial hue values:
1. Go to Settings → Devices & Services → Helpers
2. Click on the helper (e.g., `Hue Dial Button 1 Helper`)
3. Set the initial value to your preferred starting color (0-360°)
4. Common starting points: 0=Red, 30=Orange, 60=Yellow, 120=Green, 180=Cyan, 240=Blue, 300=Magenta

---

## Technical Details

### How It Works

1. **Event Detection**: Blueprint listens for `zha_event` from your Hue Tap Dial
2. **Command Parsing**: Extracts button endpoint, cluster ID, and command type
3. **Mode Routing**: Checks if button is in Light Mode or Button Mode
4. **Action Execution**: 
   - Light Mode: Controls light and updates helpers
   - Button Mode: Executes custom actions
5. **State Persistence**: Hue values and mode stored in helpers across sessions

### ZHA Event Structure

The Hue Tap Dial sends two types of events:

**Button Press Events** (Cluster 64512):
- Endpoint 1: Center button
- Endpoint 2: Upper right button  
- Endpoint 3: Lower left button
- Endpoint 4: Lower right button
- Commands: `button_X_press`, `button_X_short_release`, `button_X_double_press`, `button_X_hold`, `button_X_long_release`, `button_X_triple_press`, `button_X_quadruple_press`, `button_X_quintuple_press`

**Rotation Events** (Cluster 8):
- Commands: `step` or `step_with_on_off`
- Parameters:
  - `step_mode`: `0` or `StepMode.Up` (clockwise), `1` or `StepMode.Down` (counter-clockwise)
  - `step_size`: Rotation amount (typically 8-20, varies with rotation speed)
  - `transition_time`: Always 4 (not used by this blueprint)

### Performance Optimizations

- **Queued mode**: Processes rotation events in order without dropping any
- **Maximum queue size**: 10 events (prevents overflow during fast rotation)
- **Zero transition time**: Changes apply instantly without fade
- **Inline templates**: Reduces variable assignment overhead
- **Direct entity references**: Minimizes state lookups

### Helper Entity Structure

**Mode Helper** (`input_select.hue_dial_mode`):
- Options: `hue`, `brightness`
- Purpose: Tracks whether dial controls hue or brightness
- Shared across all lights (dial can only control one function at a time)

**Hue Value Helpers** (`input_number.hue_dial_button_X_helper`):
- Range: 0-360 (degrees on color wheel)
- Purpose: Stores each light's current hue value
- Individual per button (allows each light to maintain its own color)

---

## Limitations

1. **Dial controls one light at a time**: You must press a button to select which light the dial adjusts
2. **Hue saturation fixed at 100%**: All colors are fully saturated (vivid). Cannot create pastel colors
3. **ZHA only**: This blueprint requires the ZHA integration. Not compatible with Zigbee2MQTT.  I don't use Z2M, so just take my blueprint and ask Claude to generate something similar for you.
4. **No light groups**: Each button controls a single light entity. To control multiple lights together, create a light group entity first
5. **Zigbee lights only**: Wi-Fi based lights won't work as they don't communicate via Zigbee
6. **Button Mode disables dial**: When a button is in Button Mode, rotating the dial has no effect
7. **No color temperature control**: The blueprint only supports RGB hue control, not white color temperature (cool white ↔ warm white)
8. **Blink effect compatibility**: Some lights may not support the rapid on/off commands used for the selection blink. This doesn't affect functionality, just the visual feedback

---

## Frequently Asked Questions

**Q: Can I use this with Zigbee2MQTT instead of ZHA?**  
A: No, this blueprint is specifically designed for ZHA. Zigbee2MQTT uses different event structures.  I don't use Z2M, so just take my blueprint and ask Claude to generate something similar for you.

**Q: Can one button control multiple lights?**  
A: Not directly. Create a light group entity first (Settings → Devices & Services → Helpers → Group → Light Group), then assign that group to the button.

**Q: Why do I need separate helpers for each button's hue?**  
A: This allows each light to "remember" its color. Without this, switching between lights would reset all colors to the same value.

**Q: Can I use Button Mode for some buttons and Light Mode for others?**  
A: Yes! Each button can be configured independently. Mix and match as needed.

**Q: What happens if I press a button that's in Button Mode?**  
A: The custom actions you configured for that press type will execute. The dial won't do anything because Button Mode doesn't use the dial.

**Q: Can I change a light's color temperature (warm white to cool white)?**  
A: No, this blueprint only controls RGB hue. Color temperature control would require a different approach and light attributes.

**Q: Does this work with light effects like "colorloop"?**  
A: I don't even know what colorloop is, but Claude really wanted me to add this.  This blueprint itself doesn't trigger effects, but you can configure custom actions (triple/quadruple/quintuple press) to activate light effects or scenes.

**Q: Can I add more than 4 lights?**  
A: No, the Hue Tap Dial only has 4 buttons. To control more lights, use light groups or custom actions that toggle between different lights.  Or just buy another Hue Dial switch

**Q: Why does the dial sometimes feel laggy?**  
A: This is usually caused by Zigbee mesh issues or Home Assistant CPU load. See the "Lag when rotating dial" troubleshooting section.  I was hoping to directly bind the Hue Dial switch to the zigbee lights directly, but doing so won't enable color changing using the dial.

**Q: Can I reset a light's stored hue value?**  
A: Yes. Go to Settings → Devices & Services → Helpers, find the button's helper (e.g., `Hue Dial Button 1 Helper`), and set it to your desired starting value (or 0 to reset to red).

---

## Version History

### v1.0.0 (Current)
- Initial release
- Support for up to 4 LED lights
- Configurable Light Mode and Button Mode per button
- Individual hue value storage per button
- Shared brightness/hue mode selection
- Visual feedback with light blinks (1 blink for hue, 2 blinks for brightness)
- Adjustable sensitivity (1.0-5.0 range)
- Support for triple/quadruple/quintuple press custom actions
- Performance optimizations for minimal lag

---

## Credits & License

Created by the Home Assistant community.

This blueprint is provided as-is under the MIT License. Feel free to modify, share, and adapt for your needs.

### Contributing

Found a bug or have a feature request? Please open an issue on GitHub:
https://github.com/x4dm/Hue-Tap-Dial-ZHA-Multi-Light-Edition/issues

Pull requests are welcome!

---

## Support

For questions, issues, or feature requests:
- **GitHub Issues**: https://github.com/x4dm/Hue-Tap-Dial-ZHA-Multi-Light-Edition/issues
- **Home Assistant Community Forums**: Search for "Hue Tap Dial ZHA Multi-Light"

When reporting issues, please include:
- Home Assistant version
- ZHA integration version  
- Hue Tap Dial firmware version
- LED light model and manufacturer
- Automation traces or relevant logs
- Description of expected vs actual behavior
