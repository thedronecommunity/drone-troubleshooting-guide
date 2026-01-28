# Category 9: Arming & Pre-Flight

> Problems 41-45: Pre-Arm Errors, RXLOSS, Throttle, Arm Switch, Safety Switch

---

## Problem 41: Multiple Pre-Arm Check Failures

### Symptoms
- Can't arm — multiple errors shown
- "PreArm: ..." messages in GCS
- Arming blocked but unclear why
- New build won't arm at all
- Different errors each attempt

### Common Causes
- **Normal for new builds** — multiple things need setup
- GPS not locked
- Compass not calibrated
- Radio failsafe not configured
- Battery failsafe not configured
- RC not calibrated

### Step-by-Step Solution

1. **View all pre-arm messages:**
   - Mission Planner: Messages tab shows all errors
   - Betaflight: CLI → `status` shows arming flags
   - Address each one systematically

2. **Common checks to verify:**
   - GPS: 8+ satellites and 3D lock
   - Compass: Calibrated, consistent
   - RC: All channels responding correctly
   - Battery: Voltage reading sensible
   - Failsafe: Configured and tested

3. **For brand new builds:**
   - Complete initial setup wizard first
   - Calibrate: Accel, Compass, RC
   - Configure: Failsafes, battery monitor
   - THEN try arming

4. **If you just want to test motors:**
   - Temporarily disable some checks (not for flight!)
   ```
   ARMING_CHECK = 0    # Disable all (DANGEROUS for flight)
   ```

5. **After fixing, always re-enable checks:**
   ```
   ARMING_CHECK = 1    # Enable all checks
   ```

### Commands/Settings

```bash
# ArduPilot Pre-Arm Messages:
# Mission Planner → Messages tab
# Or: Data → Messages

# View arming check status:
# CLI: arm check
# Shows each check and pass/fail

# Disable specific checks (for testing):
ARMING_CHECK = 1      # All enabled (default, safest)
ARMING_CHECK = 0      # All disabled (DANGEROUS)
ARMING_CHECK = -9     # All except GPS
ARMING_CHECK = -17    # All except INS

# Common pre-arm causes:
# "PreArm: RC not calibrated" → Calibrate RC
# "PreArm: Compass not calibrated" → Calibrate compass
# "PreArm: Bad GPS Position" → Wait for GPS lock
# "PreArm: Accels inconsistent" → Recalibrate accelerometer
# "PreArm: High GPS HDOP" → Wait for better GPS
# "PreArm: Check FS_THR_VALUE" → Configure throttle failsafe

# Betaflight Arming Prevention Flags:
# CLI: status
# Look for: "Arming disable flags:"
# Each flag explains why arming blocked
```

### 💡 Pro Tip
Pre-arm checks exist to prevent crashes! Don't disable them to "make it work" — fix the underlying issues. Each check prevents a specific failure mode that could cause a crash or flyaway.

### ⚠️ Warning
Disabling arming checks for a quick test is tempting, but NEVER fly with `ARMING_CHECK = 0`. Re-enable all checks before any flight. Running with disabled checks has caused many accidents.

---

## Problem 42: RXLOSS Flag Won't Clear

### Symptoms
- "RXLOSS" shown as arming prevention flag
- RC looks connected but flag persists
- Sticks respond but can't arm
- Intermittent RXLOSS during flight
- Flag clears then returns

### Common Causes
- **Failsafe values not configured correctly**
- Receiver output during failsafe is above threshold
- RSSI signal too weak
- RC connection unstable
- min_check value too high

### Step-by-Step Solution

1. **Understand RXLOSS:**
   - FC sees low throttle AS failsafe signal
   - Must distinguish real low throttle from no signal
   - Configure rx_min_usec and failsafe properly

2. **Configure failsafe on receiver:**
   - Set receiver to output "no pulses" on signal loss
   - OR set receiver to output <980µs on channel 3
   - FC can then detect loss vs intentional low

3. **Check Betaflight failsafe settings:**
   ```
   Receiver → RX_MIN_USEC = 885
   Receiver → RX_MAX_USEC = 2115
   ```

4. **Test failsafe actually works:**
   - Turn off transmitter
   - Channels should go to failsafe values
   - Throttle should drop below rx_min_usec

5. **If RSSI based:**
   - Check RSSI channel configuration
   - RSSI low = perceived signal loss
   - May need to adjust RSSI thresholds

### Commands/Settings

```bash
# Betaflight RXLOSS Configuration:
set rx_min_usec = 885
set rx_max_usec = 2115
set failsafe_throttle = 1000    # Below arm threshold
set failsafe_procedure = DROP   # Motors off on failsafe

# Test failsafe:
# 1. Turn off transmitter
# 2. Watch Receiver tab
# 3. Channels should go to failsafe values (or disappear)
# 4. Throttle must be below min_check value

# ArduPilot Failsafe Settings:
FS_THR_ENABLE = 1      # Enable throttle failsafe
FS_THR_VALUE = 975     # Throttle below this = failsafe
# Set receiver to output <975 during signal loss

# Check current RC values:
# Betaflight: Receiver tab
# Look at throttle minimum value
# Must be above rx_min_usec when connected
# Must be below failsafe_throttle when TX off

# RSSI Configuration:
RSSI_CHANNEL = 0       # No RSSI via aux channel
RSSI_TYPE = 0          # Disabled
# Or configure properly if using RSSI for failsafe
```

### 💡 Pro Tip
The key insight: FC needs to tell the difference between "transmitter off" and "pilot holding throttle at minimum." Configure receiver failsafe to output a unique value (like 900µs) that you'd never see during normal flight.

### ⚠️ Warning
RXLOSS flag is a SAFETY feature! It prevents arming when signal is unreliable. Flying through RXLOSS warnings is extremely dangerous — you could lose control at any moment.

---

## Problem 43: "Throttle Not at Zero" Prevents Arming

### Symptoms
- "Throttle not at zero" or "Throttle too high"
- Can't arm even with stick at bottom
- Throttle channel shows 1050-1100 instead of 1000
- Only happens sometimes
- Trim on throttle affects arming

### Common Causes
- **min_check value too high** for your transmitter's minimum
- Throttle trim not centered
- RC endpoints not calibrated
- Throttle stick not reaching minimum
- Different min values after radio update

### Step-by-Step Solution

1. **Check throttle value in Receiver tab:**
   - Pull throttle to minimum
   - Note the value shown (e.g., 1050)
   - min_check must be ABOVE this value

2. **Adjust min_check value:**
   ```
   set min_check = 1050    # Set slightly above your minimum
   ```

3. **Or fix at radio:**
   - Center throttle trim
   - Ensure throttle reaches 1000 at bottom
   - Calibrate RC endpoints

4. **RC Calibration:**
   - Betaflight: Receiver → Stick Calibration
   - Move all sticks to extremes
   - Save calibration

5. **Check throttle trim:**
   - Throttle trim should be centered (0)
   - Even small trim up can prevent arming

### Commands/Settings

```bash
# Betaflight Throttle Settings:
get min_check
# Shows current minimum threshold

set min_check = 1050    # Raise if needed
save

# Check current throttle value:
# Receiver tab → watch throttle bar at minimum position
# Value shown must be BELOW min_check to arm

# RC Calibration in CLI:
rxrange 2 1000 2000    # Channel 3 (throttle) range
# Or use Receiver tab calibration

# ArduPilot Throttle Arming:
RC3_MIN = 1000         # Radio-reported minimum
ARMING_RUDDER = 1      # Enable rudder arming
# For throttle arming to work, RC3 must be at RC3_MIN

# Check RC input values:
# Radio tab shows current channel values
# All channels should reach endpoints defined in RCx_MIN/MAX
```

### 💡 Pro Tip
If your radio doesn't reach 1000µs at minimum throttle, that's a radio calibration issue. Fix it in the radio (endpoints/travel adjustment) rather than raising min_check to compensate.

### ⚠️ Warning
Setting min_check too high means you can arm at higher throttle — dangerous because motors spin up immediately on arm. Always verify safe arming with minimal/no motor spin.

---

## Problem 44: Arm Switch Not Working

### Symptoms
- Configured arm switch but can't arm
- Switch changes mode in Modes tab but no arm
- Armed once, now won't arm again
- Works with stick commands but not switch
- Switch position detection intermittent

### Common Causes
- **Switch not mapped to AUX channel correctly**
- Aux channel not reaching threshold values
- Mode range too narrow in configurator
- Wrong aux channel selected
- Radio switch not configured

### Step-by-Step Solution

1. **Verify switch output in radio:**
   - Check which channel the switch outputs to
   - Usually AUX1 = Channel 5, AUX2 = Channel 6, etc.

2. **Watch Receiver tab while flipping switch:**
   - Note which AUX channel moves
   - Note the values at each position (e.g., 1000, 1500, 2000)

3. **Configure Modes tab:**
   - Select "ARM" mode
   - Add range on correct AUX channel
   - Range must include switch position value
   - Yellow marker shows current switch position

4. **Ensure range is wide enough:**
   - Don't set exactly at switch values
   - Leave margin: 900-1300 instead of 1000 exactly
   - Accounts for slight variations

5. **Test in Modes tab:**
   - Flip switch
   - "ARM" should highlight when switch active
   - If not highlighting, adjust range or channel

### Commands/Settings

```bash
# Betaflight Arm Switch:
# Modes tab → Add Range → ARM
# Select AUX channel (1-4 typically)
# Set range to cover switch position

# Check aux channel values:
# Receiver tab → watch AUX channels while flipping switches
# Note exact values for each position

# Typical 3-position switch values:
# Position 1: 1000
# Position 2: 1500
# Position 3: 2000

# Set ARM range example:
# AUX1, Range: 1700-2100 (activates in position 3)

# Verify in Modes tab:
# Yellow marker shows current position
# Mode name highlights when active

# Alternative: Stick command arming
# Configuration → Arming → Enable Stick Arming
# Throttle down + Yaw right to arm (default)
```

### 💡 Pro Tip
Use a 3-position switch and assign ARM to only the highest position (1700-2100). This prevents accidental arming when switch is in middle position. Also, middle position can be used for other modes.

### ⚠️ Warning
Test arm switch before every session! If your switch breaks or config gets reset, you could arm unexpectedly (or not be able to disarm). Always verify arm/disarm works before takeoff.

---

## Problem 45: Hardware Safety Switch Issues (ArduPilot)

### Symptoms
- Can't arm even after pressing safety switch
- Safety switch LED doesn't change
- "PreArm: Hardware safety switch" message
- Safety switch broke, can't arm at all
- No safety switch on build but error appears

### Common Causes
- **Safety switch not pressed long enough** (2+ seconds)
- Safety switch not connected
- Safety switch configured but not installed
- Defective safety switch
- BRD_SAFETY parameter not set correctly

### Step-by-Step Solution

1. **Proper safety switch use:**
   - Press and HOLD for 2+ seconds
   - LED goes from blinking to solid
   - Then you can arm

2. **If no safety switch installed:**
   - Disable it in parameters:
   ```
   BRD_SAFETY_DEFLT = 0    # No safety switch
   ```

3. **Check switch connection:**
   - Connected to SAFETY port on FC
   - 3-pin connector: Signal, VCC, GND
   - Some FCs have switch built-in

4. **If switch is defective:**
   - Test with multimeter (continuity when pressed)
   - Replace switch if broken
   - Or disable in parameters if not needed

5. **Force disabling (emergency):**
   - Can be done via MAVLink command
   - Or set parameter via USB

### Commands/Settings

```bash
# ArduPilot Safety Switch Parameters:
BRD_SAFETY_DEFLT = 1    # Safety switch enabled (default)
BRD_SAFETY_DEFLT = 0    # Safety switch disabled (no hardware switch)

# Safety button behavior:
BRD_SAFETY_MASK = 0     # Safety disables all outputs
BRD_SAFETY_MASK = 0x3FF # Safety doesn't disable any outputs

# Check safety status:
# Mission Planner → Flight Data
# "Safety" indicator shows: SAFE (cannot arm) or ARMED (can arm)

# Force safety off via MAVLink:
# Mission Planner → Actions → Safety Off
# Or use arm command with force flag

# LED Status:
# Blinking = Safety ON (cannot arm)
# Solid = Safety OFF (can arm)
# No LED = Check power to switch

# Cube/Pixhawk safety switch location:
# External GPS/Safety module includes switch
# Or dedicated safety switch port on FC
```

### 💡 Pro Tip
For bench testing, disable safety switch (`BRD_SAFETY_DEFLT = 0`). For field flying, re-enable it — the extra step prevents accidental motor startup while handling the drone.

### ⚠️ Warning
The safety switch is a physical safeguard! Disabling it means motors can spin immediately on arming. Only disable if you're sure you want to accept that risk. Many pilots have been injured by unexpected motor startup.

---

## Quick Reference Card

| Symptom | First Check | Likely Fix |
|---------|------------|------------|
| Multiple pre-arm | Read all messages | Fix each issue |
| RXLOSS won't clear | Failsafe config | Set receiver failsafe |
| Throttle not zero | min_check value | Lower min_check |
| Arm switch fails | Modes tab range | Adjust range |
| Safety switch | Held 2 seconds? | Hold longer or disable |

---

[← Back to Sensor Calibration](08-sensor-calibration.md) | [Next: Flight & Tuning →](10-flight-tuning.md)