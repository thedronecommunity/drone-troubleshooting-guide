# Category 8: Sensor Calibration

> Problems 36-40: Accelerometer, Gyro, Level, Barometer, Sensor Orientation

---

## Problem 36: Accelerometer Calibration Fails

### Symptoms
- "Calibration Failed" message
- Calibration never completes
- "Accels inconsistent" pre-arm error
- Required to hold impossible positions
- Works sometimes but fails randomly

### Common Causes
- **Moving during calibration** — must be perfectly still
- FC orientation not set before calibration
- Calibrating on unstable surface
- Temperature changing during calibration
- Damaged accelerometer chip

### Step-by-Step Solution

1. **Set FC orientation FIRST:**
   - Before calibration, set `AHRS_ORIENTATION`
   - Orientation affects how calibration data is stored
   - Arrow on FC should point forward

2. **Prepare stable surface:**
   - Perfectly level surface (use bubble level)
   - No vibrations (not on running AC unit, etc.)
   - Solid table, not wobbly

3. **Calibration procedure:**
   - Place level → click "Calibrate Accel"
   - DON'T TOUCH for 5+ seconds
   - Rotate to each position when prompted
   - Hold COMPLETELY still at each position

4. **The 6 positions:**
   - Level (right side up)
   - Level (upside down)
   - Nose down
   - Nose up
   - Left side down
   - Right side down

5. **If still failing:**
   - Try in different location (less interference)
   - Let FC warm up 2-3 minutes first
   - Ensure battery not connected (USB only)

### Commands/Settings

```bash
# ArduPilot Accel Calibration:
# Mission Planner → Setup → Mandatory → Accel Calibration
# Or Initial Setup → Accelerometer → Calibrate

# Check accelerometer health:
status
# Look for: "Prearm: Accels inconsistent"

# Accel offset parameters:
INS_ACCOFFS_X = [calibrated]
INS_ACCOFFS_Y = [calibrated]
INS_ACCOFFS_Z = [calibrated]

# If values are extreme (>3), calibration is bad

# FC Orientation (set before calibrating!):
AHRS_ORIENTATION = 0    # None (arrow forward)
AHRS_ORIENTATION = 4    # Yaw 180 (arrow backward)
AHRS_ORIENTATION = 6    # Yaw 270 (arrow left)

# Betaflight Accel Calibration:
# Setup tab → Calibrate Accelerometer
# Place quad level, click button, wait

# Reset calibration if corrupted:
set acc_calibration = 0,0,0,0,0,0
save
# Then recalibrate
```

### 💡 Pro Tip
Temperature affects calibration! If you calibrate in AC room and fly in hot sun, accuracy suffers. For best results, calibrate at similar temperature to flight conditions.

### ⚠️ Warning
A bad accel calibration means the drone doesn't know which way is "down." This causes drift in all flight modes and makes GPS modes unreliable. Don't skip this step!

---

## Problem 37: Gyro Calibration Fails at Boot

### Symptoms
- "Gyros still settling" message
- Long delay at boot before arming possible
- "Gyro calibration failed" error
- Pre-arm: "Gyros not healthy"
- Gyro values drifting on screen

### Common Causes
- **Moving the drone during boot** — gyro cals on startup
- Vibrations from environment
- Wind blowing the drone
- Soft mount too soft (bouncing)
- Damaged gyro chip

### Step-by-Step Solution

1. **Don't move during power-up:**
   - Gyro calibrates automatically at boot
   - Must be PERFECTLY still for 5 seconds
   - No touching, no wind, no vibrations

2. **Ensure stable surface:**
   - Don't power up while holding drone
   - Set down, then connect battery
   - Wait for boot to complete before touching

3. **Check for environmental vibrations:**
   - Car engine running nearby
   - AC/fan blowing
   - Unstable table/surface
   - People walking nearby

4. **Soft mount considerations:**
   - Too soft = gyro keeps moving
   - Too hard = motor vibrations couple in
   - Balance is key

5. **If gyro is damaged:**
   - Values jumping randomly
   - Calibration never settles
   - May need FC replacement

### Commands/Settings

```bash
# ArduPilot Gyro Status:
status
# Look for: "Prearm: Gyros not healthy"

# Gyro parameters:
INS_GYR_CAL = 1         # Calibrate on every boot (default)
INS_GYR_CAL = 0         # Never calibrate (use stored)

# Check gyro health in CLI:
test gyro
# Shows raw gyro values — should be stable when still

# Betaflight Gyro Status:
# CLI: status
# Shows gyro detection and health

# Gyro noise check:
# Betaflight: Enable Blackbox, analyze gyro traces
# Noisy gyro = vibration or damaged chip

# ArduPilot Gyro Filtering:
INS_GYRO_FILTER = 20    # Low-pass filter (Hz)
# Lower value = more filtering, less noise, more lag
```

### 💡 Pro Tip
If gyro calibration is slow but eventually works, you likely have vibration or movement during boot. Let the drone sit completely undisturbed for 30 seconds after power-on.

### ⚠️ Warning
A bad gyro makes the drone unflyable — it's the core sensor for all stabilization. If gyro is drifting or noisy even when still, the FC may be damaged.

---

## Problem 38: Drone Not Level After Calibration

### Symptoms
- Horizon in OSD/GCS is tilted
- Drone drifts in one direction when hands-off
- Level surface shows drone tilted
- Just calibrated but still not level
- Model preview tilted in Betaflight

### Common Causes
- **Calibrated on non-level surface**
- FC not mounted level in frame
- AHRS_TRIM not adjusted
- FC orientation set wrong
- Vibration dampeners compressed unevenly

### Step-by-Step Solution

1. **Use actual level for calibration:**
   - Use bubble level to verify surface
   - Phone apps are not accurate enough
   - Even small tilt = drift in flight

2. **Check FC mounting:**
   - FC should be parallel to frame
   - Stack height even on all corners
   - Not tilted by wire tension

3. **Adjust AHRS_TRIM:**
   - Fine-tune after calibration
   - Compensates for FC mounting angle
   - Small adjustments only

4. **Recalibrate on verified level surface:**
   - Find a known level surface
   - Verify with bubble level
   - Recalibrate accelerometer

5. **Betaflight Board Alignment:**
   - Configuration → Board and Sensor Alignment
   - Adjust degrees offset if FC mounted at angle

### Commands/Settings

```bash
# ArduPilot Level Calibration:
# Mission Planner → Setup → Accel Cal → Calibrate Level

# Manual AHRS trim adjustment:
AHRS_TRIM_X = 0.02     # Roll trim (radians)
AHRS_TRIM_Y = -0.01    # Pitch trim (radians)
# Positive = FC thinks that side is higher
# 0.01 rad ≈ 0.6 degrees

# Check current attitude:
# Mission Planner → Flight Data → Artificial Horizon
# Should be level when drone is level

# Betaflight Board Alignment:
# Configuration tab → Board and Sensor Alignment
# Roll degrees, Pitch degrees, Yaw degrees
# Adjust until model preview matches reality

# Betaflight CLI:
set align_board_roll = 0
set align_board_pitch = 0
set align_board_yaw = 0
save

# Verify with:
# Setup tab → watch horizon indicator while tilting drone
```

### 💡 Pro Tip
The "hover trim" test: hover in calm conditions, note which way drone drifts, adjust AHRS_TRIM opposite direction. Small adjustments (0.005 rad increments) until drift-free.

### ⚠️ Warning
Large AHRS_TRIM values (>0.05 rad) indicate the surface wasn't level during calibration or the FC is badly mounted. Fix the root cause rather than compensating with large trim values.

---

## Problem 39: Barometer Altitude Readings Wrong

### Symptoms
- Altitude jumps when drone is stationary
- Altitude reads wrong compared to actual height
- Alt hold oscillates up and down
- Altitude affected by wind/prop wash
- Different altitude reading each power cycle

### Common Causes
- **Barometer exposed to prop wash**
- No foam covering on barometer
- Sunlight heating the sensor
- Pressure changes from weather
- Damaged barometer chip

### Step-by-Step Solution

1. **Cover barometer with open-cell foam:**
   - Small piece of foam over baro chip
   - Blocks prop wash pressure changes
   - Open-cell (not closed) for accuracy
   - Most FCs come with this, verify it's installed

2. **Shield from sunlight:**
   - Direct sun heats sensor
   - Causes altitude drift
   - Cover FC or shade barometer

3. **Mount away from prop wash:**
   - Center of frame is worst
   - Under canopy/cover is better
   - Air pressure changes = altitude noise

4. **Calibration:**
   - Barometer calibrates automatically at boot
   - Records current pressure as "ground level"
   - Must be at actual ground level at boot

5. **Check for damage:**
   - Constant wild readings = damaged sensor
   - May need FC replacement

### Commands/Settings

```bash
# ArduPilot Barometer Settings:
BARO_PRIMARY = 0        # Use first barometer
GND_ABS_PRESS = [auto]  # Ground pressure (set at boot)
GND_ALT_OFFSET = 0      # Altitude offset

# Check barometer health:
status
# Shows barometer status and altitude

# EKF altitude source:
EK2_ALT_SOURCE = 0   # Barometer (default)
EK3_SRC1_POSZ = 1    # Barometer for EKF3

# Altitude noise filtering:
EK2_ALT_NOISE = 1.0  # Expected noise (meters)
EK3_ALT_NOISE = 1.0  # For EKF3

# Betaflight Baro Settings:
# Configuration → Barometer → Enable
# Used for alt hold modes

# Test barometer:
# Watch altitude in GCS while stationary
# Should be stable within ±0.5m
# Wave hand near baro — should see spike (normal)
```

### 💡 Pro Tip
The "breath test" checks baro function: gently blow near the FC (not directly on it). Altitude should briefly change. If no change, baro may be damaged or not enabled.

### ⚠️ Warning
Altitude hold modes rely entirely on barometer (unless GPS is used). A noisy or wrong barometer makes alt hold oscillate dangerously. Verify stable altitude reading before using alt hold.

---

## Problem 40: Sensor Orientation Mismatch

### Symptoms
- Tilting drone forward shows backward tilt in GCS
- Compass direction wrong by 90° or 180°
- GPS velocity direction wrong
- Controls seem reversed in stabilized modes
- Everything is backwards/inverted

### Common Causes
- **AHRS_ORIENTATION not set** for FC mounting
- Compass orientation different from FC
- FC mounted with arrow pointing wrong way
- External compass not aligned with frame
- Mixing orientations between sensors

### Step-by-Step Solution

1. **Identify how FC is mounted:**
   - Find arrow on FC
   - Note which way it points relative to frame front
   - Note any rotation from level

2. **Set AHRS_ORIENTATION:**
   | FC Arrow Points | AHRS_ORIENTATION |
   |-----------------|------------------|
   | Forward | 0 (None) |
   | Right | 2 (Yaw90) |
   | Backward | 4 (Yaw180) |
   | Left | 6 (Yaw270) |

3. **Verify orientation is correct:**
   - Tilt drone nose down → pitch should go negative
   - Tilt drone right → roll should go positive
   - Rotate clockwise → yaw should increase

4. **Set compass orientation separately if needed:**
   - External GPS/compass may need different orientation
   - `COMPASS_ORIENT` parameter
   - Usually 0 if GPS module arrow points forward

5. **After setting orientation, recalibrate:**
   - Accelerometer calibration
   - Compass calibration
   - Both use orientation setting

### Commands/Settings

```bash
# ArduPilot Orientation Settings:
AHRS_ORIENTATION = 0    # FC arrow forward, level
# Values: 0=None, 1=Yaw45, 2=Yaw90, 4=Yaw180, 6=Yaw270, 8=Roll180, etc.

# External Compass Orientation:
COMPASS_ORIENT = 0      # Compass arrow forward
# Only needed if external compass mounted differently than FC

# Verify orientation:
# Mission Planner → Flight Data → Status
# Watch pitch, roll, yaw while moving drone
# pitch = nose up/down (-90 to +90)
# roll = left/right wing down (-180 to +180)
# yaw = heading (0-360)

# Betaflight Board Alignment:
# Configuration tab → Board and Sensor Alignment
# Set Yaw degrees = 90 if FC rotated 90° CW
# Set Yaw degrees = 180 if FC faces backward

# After changing orientation:
# 1. Recalibrate accelerometer
# 2. Recalibrate compass
# Both calibrations depend on orientation setting!
```

### 💡 Pro Tip
The "table tilt test" is essential: place drone on table, tilt front down — FC should show pitch negative, tail down — pitch positive. If reversed, add 180 to your orientation setting.

### ⚠️ Warning
Wrong sensor orientation makes the drone fight against itself — stick forward causes backward flight, or worse. ALWAYS verify orientation with tilt test before flying!

---

## Quick Reference Card

| Symptom | First Check | Likely Fix |
|---------|------------|------------|
| Accel cal fails | Surface level? | Use bubble level |
| Gyro won't settle | Moving at boot? | Stay completely still |
| Not level | Calibration surface | Recal on level surface |
| Baro jumps | Foam cover? | Add foam over baro |
| Everything backwards | AHRS_ORIENT? | Set FC orientation |

---

[← Back to Telemetry](07-telemetry-ground-station.md) | [Next: Arming & Pre-Flight →](09-arming-preflight.md)