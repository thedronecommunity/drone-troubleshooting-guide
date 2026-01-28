# Category 7: Telemetry & Ground Station

> Problems 31-35: Mission Planner, SiK Radio, OSD, Parameters, QGroundControl

---

## Problem 31: "No Heartbeat Packets Received"

### Symptoms
- Mission Planner shows "No Heartbeat Packets Received"
- Can't connect to vehicle
- COM port opens but no data
- Worked before, suddenly stopped
- Telemetry radios show solid LEDs but no link

### Common Causes
- **Wrong baud rate** — FC and radio/GCS must match
- TX/RX wires swapped on TELEM port
- UART not configured for MAVLink protocol
- SiK radios not paired (different net IDs)
- Telemetry radio not powered

### Step-by-Step Solution

1. **Verify the entire chain:**
   ```
   FC TELEM TX → Radio RX
   FC TELEM RX → Radio TX
   5V → 5V
   GND → GND
   ```

2. **Check baud rate match:**
   - FC side: `SERIALn_BAUD` (e.g., SERIAL1_BAUD = 57)
   - Radio side: Usually 57600 default
   - GCS: Connection settings → 57600
   - ALL THREE must match!

3. **Configure UART for MAVLink:**
   - ArduPilot: `SERIAL1_PROTOCOL = 2` (MAVLink2)
   - Betaflight: Ports tab → UART → Enable Telemetry

4. **Pair SiK radios (if using):**
   - Both radios need same:
     - Net ID
     - Air Speed
     - Baud Rate
   - Use Mission Planner → Setup → Optional → SiK Radio

5. **Test with direct USB first:**
   - Connect FC directly via USB
   - If USB works but telemetry doesn't = radio issue
   - If USB also fails = FC configuration issue

### Commands/Settings

```bash
# ArduPilot TELEM1 Configuration:
SERIAL1_PROTOCOL = 2    # MAVLink2
SERIAL1_BAUD = 57       # 57600 baud (standard for SiK)

# If using TELEM2:
SERIAL2_PROTOCOL = 2
SERIAL2_BAUD = 57

# SiK Radio Default Settings:
# Baud: 57600
# Air Speed: 64
# Net ID: 25
# TX Power: 20
# Both radios MUST match these settings!

# Mission Planner Connection:
# Port: COMx (your radio's COM port)
# Baud: 57600
# Click "Connect"

# Verify in CLI (connected via USB):
status
# Look for: Heartbeat, MAVLink active

# Test serial port output:
# In MAVProxy:
module load graph
graph SYSTEM_TIME
# Should show data if MAVLink working
```

### 💡 Pro Tip
When debugging, always test USB direct connection first. If USB works, you know the FC is configured correctly, and the problem is the telemetry radio link. Simplify the problem by testing each segment.

### ⚠️ Warning
"No Heartbeat" before every flight is unacceptable for autonomous missions. You NEED that connection to monitor the drone, change parameters, and trigger failsafes. Don't fly autonomous without confirmed telemetry.

---

## Problem 32: Telemetry Range Very Short

### Symptoms
- Signal lost at 100-200 meters
- Works close but drops at distance
- Intermittent connection as drone moves
- Advertised 1km range, getting 200m
- LOS (line of sight) but still drops

### Common Causes
- **Antenna damaged or not connected**
- TX power set too low
- Antenna pointing wrong direction
- Wrong frequency for region
- Interference from other electronics

### Step-by-Step Solution

1. **Check antenna connections:**
   - Both air and ground radio antennas
   - SMA connectors finger-tight
   - No damage to antenna elements
   - Antenna not disconnected inside casing

2. **Verify TX power:**
   - SiK default: 20dBm (100mW)
   - Can increase to max (check your radio spec)
   - Higher power = more range but more current draw

3. **Antenna orientation:**
   - Omnidirectional antennas: vertical orientation
   - Signal is weakest off the tip
   - Ground antenna should be vertical
   - Air antenna perpendicular to ground antenna

4. **Check frequency/region:**
   - 433MHz: Better range, lower data rate
   - 915MHz: US/Australia standard
   - 868MHz: Europe standard
   - Use correct frequency for your region

5. **Reduce interference:**
   - Separate from 2.4GHz video/RC
   - Move away from WiFi sources
   - Shield receiver from motor noise

### Commands/Settings

```bash
# SiK Radio Configuration (via Mission Planner):
# Setup → Optional Hardware → SiK Radio

# Key parameters:
TXPOWER = 20       # TX power in dBm (max varies by radio)
AIR_SPEED = 64     # Air data rate (lower = more range)
NETID = 25         # Network ID (both must match)
ECC = 1            # Error correction ON (reduces speed, improves range)

# Higher range settings:
AIR_SPEED = 4      # Lowest speed = max range
ECC = 1            # Error correction on
TXPOWER = [max]    # Maximum allowed for your radio

# Check radio status in Mission Planner:
# Setup → Optional Hardware → SiK Radio
# Shows RSSI, noise, signal quality

# ArduPilot RSSI to OSD:
RSSI_TYPE = 3      # MAVLink telemetry RSSI
# OSD will show telemetry link quality
```

### 💡 Pro Tip
For maximum range: use 433MHz (not 915), lowest air speed (4), ECC on, maximum TX power, and proper antenna orientation. This combo can achieve 5km+ with cheap SiK radios and good line of sight.

### ⚠️ Warning
Don't exceed legal TX power limits for your region! 433MHz and 915MHz have different power limits. Also, high power drains battery faster — your ground radio may need external power.

---

## Problem 33: Parameters Won't Download / Timeout

### Symptoms
- "Getting Parameters" hangs at X%
- Timeout errors when reading parameters
- Some parameters load, then stops
- Very slow parameter download
- Works over USB but not telemetry

### Common Causes
- **Telemetry link too slow** for all parameters
- Packet loss on radio link
- GCS stream rate too high
- Mismatched baud rates
- Radio buffer overflow

### Step-by-Step Solution

1. **Use USB for full parameter download:**
   - Telemetry is slower than USB
   - Initial setup: use USB
   - In-flight: use telemetry for monitoring only

2. **Reduce parameter stream rate:**
   - ArduPilot: Lower `SR1_PARAMS` value
   - Reduces demand on telemetry link
   - Try 5 or 2 Hz instead of 10

3. **Increase timeouts:**
   - Mission Planner → Config → Planner → timeout settings
   - Increase from default values
   - Gives more time for slow links

4. **Check link quality:**
   - Look at RSSI values
   - >50% signal needed for reliable params
   - High noise = packet loss = timeouts

5. **Minimize other streams:**
   - Disable streams you don't need
   - `SR1_POSITION = 0` if not needed
   - Frees bandwidth for parameters

### Commands/Settings

```bash
# ArduPilot Stream Rates (for telemetry):
SR1_PARAMS = 5      # Parameter stream rate (lower = more reliable)
SR1_EXT_STAT = 2    # Extended status
SR1_EXTRA1 = 5      # Attitude
SR1_EXTRA2 = 2      # VFR HUD
SR1_EXTRA3 = 2      # AHRS
SR1_POSITION = 2    # GPS Position
SR1_RC_CHAN = 0     # Disable RC channel stream
SR1_RAW_SENS = 0    # Disable raw sensor stream

# Mission Planner Timeout Settings:
# Config → Planner tab
# Parameter download timeout: 15000ms (increase if needed)
# Command timeout: 10000ms

# For very slow links:
SR1_PARAMS = 2      # Even slower parameter stream
# Or download params via USB, save to file
# Then just monitor via telemetry

# Check link quality:
# Mission Planner → Status tab
# Look for: rxerrors, fixed (packet stats)
```

### 💡 Pro Tip
Save your parameters to a file after configuring via USB. Then you have a backup, and you don't need to download all 800+ parameters over slow telemetry links. Just verify critical ones are set.

### ⚠️ Warning
Don't try to change parameters over a weak telemetry link! Partial writes can corrupt settings. If you need to change parameters, do it via USB or with a strong telemetry connection.

---

## Problem 34: OSD Elements Missing or Wrong

### Symptoms
- Some OSD elements don't appear
- Elements in wrong position
- OSD shows wrong data (speed, altitude)
- OSD works but missing items
- Different from what's configured

### Common Causes
- **OSD not enabled in firmware**
- Elements positioned off-screen
- Font not uploaded to OSD chip
- Data source not available (no GPS = no speed)
- Wrong OSD profile active

### Step-by-Step Solution

1. **Enable OSD in configuration:**
   - Betaflight: Configuration → OSD → Enable
   - ArduPilot: `OSD_TYPE = 1` (or 2/3 depending on hardware)

2. **Check element positions:**
   - Elements at position 0,0 = hidden
   - Max position depends on resolution
   - PAL: 30 rows, NTSC: 13 rows
   - Move elements into visible area

3. **Upload OSD font:**
   - Betaflight: OSD tab → Font Manager
   - Some fonts are better than others
   - Font must match OSD chip type

4. **Verify data sources:**
   - No GPS = no GPS-related OSD elements
   - No current sensor = no mAh/Amps
   - Elements show "--" if data unavailable

5. **Check OSD profile:**
   - Multiple profiles available
   - Make sure correct profile is active
   - Profile can be switched via RC channel

### Commands/Settings

```bash
# ArduPilot OSD Configuration:
OSD_TYPE = 1         # 1=integrated, 2=external MAX7456
OSD1_ENABLE = 1      # Enable OSD screen 1

# Element positions (example):
OSD1_ALTITUDE_EN = 1    # Enable altitude
OSD1_ALTITUDE_X = 1     # X position
OSD1_ALTITUDE_Y = 1     # Y position

# Betaflight OSD:
# OSD tab → Enable OSD
# Drag elements to desired positions
# Save

# OSD Elements requiring sensors:
# Altitude → Barometer
# GPS Speed → GPS
# Home Direction → GPS + Compass
# Battery mAh → Current sensor
# RSSI → RSSI_TYPE configured

# Check available data:
status
# Shows what sensors are detected

# Font upload (Betaflight):
# OSD tab → Font Manager → Upload Font
```

### 💡 Pro Tip
The OSD preview in Betaflight doesn't always match reality. After configuring, power up with your goggles/monitor to verify. PAL vs NTSC setting must match your camera output format.

### ⚠️ Warning
Flying without a working OSD means no voltage warning, no RSSI warning, no failsafe indication. Don't fly until your OSD is working, especially battery voltage display.

---

## Problem 35: QGroundControl Won't Connect

### Symptoms
- QGC shows "Waiting for Vehicle"
- Connection times out
- Works in Mission Planner but not QGC
- UDP connection fails
- Serial connection not detected

### Common Causes
- **Auto-connect not finding port**
- Firewall blocking UDP ports
- Wrong connection type selected
- MAVLink 1 vs MAVLink 2 mismatch
- QGC version incompatibility

### Step-by-Step Solution

1. **Try manual connection:**
   - Comm Links → Add
   - Type: Serial
   - Select correct COM port
   - Set baud rate (57600 or 115200)
   - Connect

2. **For UDP connections:**
   - Check firewall allows port 14550
   - Add UDP link: port 14550
   - Used for WiFi telemetry, SITL

3. **Verify MAVLink version:**
   - QGC expects MAVLink 2 by default
   - ArduPilot: `SERIAL1_PROTOCOL = 2` (MAVLink 2)
   - Or try MAVLink 1 for compatibility

4. **Check auto-connect settings:**
   - Application Settings → Comm Links
   - Enable/disable auto-connect options
   - Sometimes auto-detect conflicts

5. **Firewall configuration (Windows):**
   - Allow QGroundControl through firewall
   - Allow UDP 14550 inbound
   - Disable VPN if interfering

### Commands/Settings

```bash
# ArduPilot for QGC (MAVLink 2):
SERIAL1_PROTOCOL = 2   # MAVLink 2 (preferred)
SERIAL1_BAUD = 57      # Match QGC connection

# For UDP (WiFi/Network):
SERIAL_PROTOCOL = 2
# QGC → Comm Links → Add → UDP
# Port: 14550
# Auto-connect

# Manual Serial Connection in QGC:
# 1. Q icon → Application Settings
# 2. Comm Links → Add
# 3. Name: Telemetry
# 4. Type: Serial
# 5. Serial Port: COM[X] or /dev/ttyUSB0
# 6. Baud Rate: 57600
# 7. Connect

# Windows Firewall:
# Control Panel → Firewall → Allow app
# Add QGroundControl → Allow both Private/Public

# Linux USB permissions:
sudo usermod -a -G dialout $USER
# Logout and back in
```

### 💡 Pro Tip
QGC and Mission Planner can both connect simultaneously if you use MAVLink router. Route the connection through mavlink-router, and both GCS apps can connect via UDP to the router.

### ⚠️ Warning
Auto-connect can grab the wrong port (like a GPS or other serial device). If QGC is behaving strangely, disable auto-connect and manually specify your telemetry port.

---

## Quick Reference Card

| Symptom | First Check | Likely Fix |
|---------|------------|------------|
| No heartbeat | Baud rate match? | Set all to 57600 |
| Short range | Antenna connected? | Check SMA tight |
| Param timeout | Using USB? | Use USB for config |
| Missing OSD | OSD enabled? | Enable in firmware |
| QGC won't connect | Auto-connect? | Manual connection |

---

[← Back to Power System](06-power-system.md) | [Next: Sensor Calibration →](08-sensor-calibration.md)