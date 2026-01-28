# Category 4: GPS & Compass Issues

> Problems 16-20: GPS Lock, Compass Cal, Toilet Bowl, Detection, Glitches

---

## Problem 16: GPS Not Getting Satellite Lock

### Symptoms
- GPS shows 0 satellites for extended periods
- Satellite count stuck at 1-4 (never reaching 8+)
- Takes 15+ minutes to get first lock
- GPS works sometimes but not consistently
- Position jumps around wildly

### Common Causes
- **Testing indoors** — GPS needs clear sky view
- GPS antenna facing wrong direction
- First cold start (can take 15+ minutes)
- Interference from other electronics
- Damaged GPS module or antenna
- Buildings/trees blocking sky view

### Step-by-Step Solution

1. **Go outside with clear sky view:**
   - GPS doesn't work indoors
   - Even a window blocks most signal
   - Need open sky, not under trees/buildings

2. **Verify antenna orientation:**
   - GPS antenna (usually ceramic patch) faces UP
   - Top surface toward sky
   - Some modules have arrow indicating forward

3. **Wait for cold start:**
   - First power-on can take 10-15 minutes
   - GPS needs to download almanac data
   - Subsequent starts much faster (30 seconds)

4. **Check for interference:**
   - Mount GPS away from VTX (video transmitter)
   - Away from high-current wires
   - GPS mast raises antenna above interference
   - Keep away from motors/ESCs

5. **Verify wiring and settings:**
   - Power: 5V (most GPS modules)
   - TX/RX correctly connected to FC UART
   - Correct baud rate (usually 38400 or 115200)

### Commands/Settings

```bash
# ArduPilot GPS Configuration:
GPS_TYPE = 1         # Auto-detect (most common)
GPS_TYPE2 = 0        # No secondary GPS
GPS_AUTO_CONFIG = 1  # Auto configure GPS
GPS_NAVFILTER = 8    # Airborne <2g (drones)
SERIAL3_PROTOCOL = 5 # GPS protocol (check your GPS UART)
SERIAL3_BAUD = 38    # 38400 baud (common for GPS)

# Betaflight GPS Configuration:
# Configuration tab:
# GPS → Enable
# GPS Protocol → UBLOX (most common)
# Baud Rate → 57600 (try 38400 if issues)

# Check GPS in Betaflight CLI:
status
# Look for GPS section with sat count
```

### 💡 Pro Tip
After getting your first GPS lock, the module saves satellite data. Future locks are MUCH faster (30-60 seconds). If you always have slow lock times, the GPS might not be saving data — check the battery backup on the module.

### ⚠️ Warning
Never arm and fly before GPS shows "3D Lock" with 8+ satellites. Flying GPS modes without proper lock causes flyaways. ArduPilot requires this for Loiter/Auto modes.

---

## Problem 17: Compass Calibration Always Fails

### Symptoms
- Compass calibration fails no matter how many attempts
- "Calibration Failed" or "Offsets too large"
- Compass works briefly then fails pre-arm
- Different readings every calibration attempt
- COMPASS_OFFS values unreasonably high (>600)

### Common Causes
- **Metal or magnets too close** to compass during calibration
- Calibrating indoors near electronics/metal
- GPS/compass module mounted too close to motors/wires
- Internal compass affected by FC components
- Bad compass sensor (rare)

### Step-by-Step Solution

1. **Move to proper calibration environment:**
   - Outdoors, away from buildings
   - No metal tables, cars, fences nearby
   - No electronics in pockets (phone, keys)
   - Stand in open grass/concrete area

2. **During calibration:**
   - Rotate smoothly, not jerky
   - All three axes: pitch, roll, yaw
   - Complete full 360° rotations
   - Keep slow, consistent movement

3. **Check compass mounting:**
   - Mount as far from motors as possible
   - Use GPS mast if available
   - Keep away from high-current wires
   - Away from iron/steel in frame

4. **If internal compass is bad:**
   - Disable internal, use external only:
   ```
   COMPASS_USE = 1
   COMPASS_USE2 = 0    # Disable internal
   COMPASS_USE3 = 0
   ```

5. **Lower acceptable offset threshold (temporary debug):**
   ```
   COMPASS_OFFS_MAX = 1000   # Default 600
   ```
   - If calibration passes now, you have interference
   - Find and fix the interference source

### Commands/Settings

```bash
# ArduPilot Compass Parameters:
COMPASS_USE = 1          # Enable compass 1
COMPASS_USE2 = 0         # Disable internal (if using external)
COMPASS_EXTERNAL = 1     # Using external compass
COMPASS_ORIENT = 0       # No rotation (check your GPS mount)
COMPASS_OFFS_MAX = 600   # Max offset (increase if failing)
COMPASS_LEARN = 1        # Enable auto-learning

# Check compass health:
# Mission Planner → Status tab
# Look for: compass_ofs_x, compass_ofs_y, compass_ofs_z
# Values over 600 indicate interference

# Betaflight Compass (Mag):
# Configuration → Magnetometer
# Select alignment if compass rotated

# Diagnose compass issues:
# Arm and watch compass heading
# Rotate drone, heading should track smoothly
# Jerky/jumping = interference
```

### 💡 Pro Tip
If compass keeps failing, try calibrating with motors running (props OFF!). This tests if motor magnetic fields are interfering. If it fails worse, your compass is too close to motors.

### ⚠️ Warning
A bad compass calibration causes "toilet bowl" effect in GPS modes and can lead to flyaways. If you can't get a clean calibration, don't fly GPS modes until you fix the interference issue.

---

## Problem 18: Toilet Bowl Effect in Loiter Mode

### Symptoms
- Drone circles/spirals in Loiter mode
- Can't hold position, drifts in circles
- Position hold works briefly then starts circling
- Gets worse the longer you hover
- Spiral diameter increases over time

### Common Causes
- **Compass/GPS heading mismatch** — compass pointing wrong direction
- Compass not calibrated properly
- Compass orientation parameter wrong
- Motor interference on compass
- GPS multipath (reflections from buildings)

### Step-by-Step Solution

1. **Verify compass orientation:**
   - Arrow on GPS/compass should point forward
   - Check `COMPASS_ORIENT` parameter
   - Common values: 0 (forward), 6 (270°)

2. **Recalibrate compass outdoors:**
   - Away from all metal
   - Slow, complete rotations
   - All three axes

3. **Enable motor compass compensation:**
   - ArduPilot: `COMPASS_MOT_CT = 2` (throttle compensation)
   - Helps cancel motor magnetic interference
   - Requires throttle-based calibration

4. **Check compass vs GPS heading:**
   - Mission Planner → Quick tab
   - Compare compass heading vs GPS heading
   - They should match when moving
   - If opposite, compass is 180° off

5. **Try disabling internal compass:**
   - If FC has internal compass + external GPS/compass
   - External (on mast) usually better
   - `COMPASS_USE2 = 0` (disable internal)

### Commands/Settings

```bash
# ArduPilot Toilet Bowl Fix:
COMPASS_ORIENT = 0      # Check correct orientation!
COMPASS_MOT_CT = 2      # Enable throttle compensation
COMPASS_USE = 1         # Use primary compass only
COMPASS_USE2 = 0        # Disable internal
COMPASS_USE3 = 0        # Disable third

# Motor compensation calibration:
# 1. Set COMPASS_MOT_CT = 2
# 2. Go to Setup → Compass → Motor Cal
# 3. Increase throttle slowly
# 4. Parameters saved automatically

# Check compass orientation:
# Fly forward, note GPS heading
# Should match compass heading ±10°
# If 180° off, add COMPASS_ORIENT = 4

# EKF parameters (advanced):
EK2_MAG_CAL = 3    # Learn compass biases in flight
EK3_MAG_CAL = 3    # For EKF3
```

### 💡 Pro Tip
The "walk test" diagnoses toilet bowl: walk forward holding the drone, compass heading should match direction of travel. If it shows you're walking backward while walking forward, compass is 180° off. Set `COMPASS_ORIENT = 4`.

### ⚠️ Warning
Flying with toilet bowl effect is dangerous — the circles get bigger and can lead to flyaway. Land immediately if you see circling behavior in Loiter and diagnose before flying GPS modes again.

---

## Problem 19: GPS Module Not Detected by Flight Controller

### Symptoms
- GPS status shows "No GPS" or "GPS: OFF"
- No satellite count displayed
- GPS worked before, suddenly nothing
- Brand new GPS module not detected

### Common Causes
- **TX/RX wires swapped** (GPS TX → FC RX, not TX→TX)
- Wrong UART enabled for GPS
- Wrong protocol selected (UBLOX vs NMEA)
- GPS not getting 5V power
- Wrong baud rate configured
- GPS module damaged

### Step-by-Step Solution

1. **Verify wiring (TX/RX swap!):**
   ```
   GPS TX → FC RX
   GPS RX → FC TX
   GPS VCC → FC 5V
   GPS GND → FC GND
   ```

2. **Enable GPS on correct UART:**
   - ArduPilot: `SERIALn_PROTOCOL = 5` (where n is your GPS UART)
   - Betaflight: Ports tab → Enable GPS on correct UART

3. **Set correct GPS type:**
   - ArduPilot: `GPS_TYPE = 1` (Auto) or `GPS_TYPE = 2` (UBLOX)
   - Betaflight: Configuration → GPS → Protocol: UBLOX

4. **Check baud rate:**
   - Most uBlox GPS: 38400 or 115200
   - ArduPilot: `SERIALn_BAUD = 38` or `115`
   - Betaflight: Usually auto-detects

5. **Verify GPS power:**
   - GPS needs 5V (some are 3.3V)
   - Check voltage at GPS VCC pin
   - Should be stable 5V ±0.2V

### Commands/Settings

```bash
# ArduPilot GPS Setup:
SERIAL3_PROTOCOL = 5   # GPS (change 3 to your UART number)
SERIAL3_BAUD = 38      # 38400 baud (typical uBlox)
GPS_TYPE = 1           # Auto-detect
GPS_AUTO_CONFIG = 1    # Auto-configure module

# Betaflight GPS Setup:
# Ports tab:
# UART[n] → Toggle "GPS" ON
# Configuration tab:
# GPS → Enable toggle ON
# GPS Protocol → UBLOX

# Check GPS detection in CLI:
# ArduPilot:
status   # Look for GPS: OK
# Betaflight:
status   # Look for GPS satellite count

# Force GPS type if auto fails:
GPS_TYPE = 2   # uBlox (most common)
```

### 💡 Pro Tip
New GPS modules sometimes ship set to wrong baud rate. If auto-detect fails, connect GPS directly to u-center (uBlox software) via USB-to-serial adapter and verify/set baud rate to 38400.

### ⚠️ Warning
GPS modules are polarity-sensitive! Reversing VCC/GND can instantly damage the module. Double-check polarity before connecting. The module won't smoke but will be dead.

---

## Problem 20: GPS Glitches / Position Jumps

### Symptoms
- GPS position jumps around suddenly
- Drone jerks or twitches in Loiter/Position Hold
- HDOP values spike randomly
- Position error warnings in logs
- Works fine in open sky, bad near buildings

### Common Causes
- **Multipath** — GPS signals bouncing off buildings/trees
- Low satellite count (under 8)
- High HDOP values (poor geometry)
- Interference from onboard electronics
- Weak GPS antenna connection

### Step-by-Step Solution

1. **Check HDOP and satellite count:**
   - Aim for: HDOP < 2.0, Satellites > 8
   - Higher HDOP = lower accuracy
   - Fewer sats = higher error

2. **Improve antenna placement:**
   - GPS on mast, away from frame
   - Clear view of entire sky
   - Away from VTX antenna

3. **Avoid flying near buildings:**
   - Reflected signals cause jumps
   - Fly in open areas for GPS modes
   - Metal buildings are worst

4. **Filter GPS data (ArduPilot):**
   - Increase `GPS_GNSS_MODE` to use more satellite systems
   - Enable `GPS_AUTO_SWITCH` for dual GPS setups
   - Check `EK2_GPS_CHECK` settings

5. **Consider dual GPS:**
   - Two GPS modules = better redundancy
   - ArduPilot blends both for accuracy
   - Catches glitches from single module

### Commands/Settings

```bash
# ArduPilot GPS Quality Settings:
GPS_GNSS_MODE = 67   # GPS + GLONASS + Galileo
GPS_AUTO_SWITCH = 1  # Switch to better GPS
GPS_HDOP_GOOD = 140  # Max HDOP for "good" (1.4)

# EKF GPS Checking:
EK2_GPS_CHECK = 31   # All checks enabled
EK3_GPS_CHECK = 31   # For EKF3

# Increase GPS smoothing:
EK2_GPS_DELAY = 200  # GPS delay in ms
EK3_GPS_DELAY = 200

# Monitor GPS quality:
# Mission Planner → Quick tab
# Watch: hdop, numsats, gps_status
# Also check: "GPS Glitch" in HUD
```

### 💡 Pro Tip
GPS problems often look like flight controller issues. If your drone is drifting or twitching in position hold, check GPS HDOP and sat count FIRST. HDOP > 3.0 means GPS quality is too poor for stable position hold.

### ⚠️ Warning
Flying GPS modes in urban canyons (between tall buildings) is risky. Multipath can cause sudden position jumps of 10+ meters, making the drone dart unexpectedly. Use open areas for GPS-dependent flight modes.

---

## Quick Reference Card

| Symptom | First Check | Likely Fix |
|---------|------------|------------|
| No GPS lock | Outdoors? | Go outside, wait 15 min |
| Compass cal fails | Near metal? | Move away from metal |
| Toilet bowl | Compass orientation | Check COMPASS_ORIENT |
| GPS not detected | TX/RX swapped? | Swap TX↔RX wires |
| Position jumps | HDOP value | Fly in open area |

---

[← Back to ESC & Motor](03-esc-motor-issues.md) | [Next: Companion Computer →](05-companion-computer.md)