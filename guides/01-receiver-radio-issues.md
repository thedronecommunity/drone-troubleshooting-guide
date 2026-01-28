# Category 1: Receiver & Radio Issues

> Problems 1-5: ELRS, CRSF, Binding, Channel Mapping, SBUS

---

## Problem 1: ELRS/CRSF Receiver Not Detected

### Symptoms
- Channel bars in Betaflight Receiver tab show flat lines at 1500 or are completely grayed out
- No response when moving transmitter sticks
- Receiver LED shows solid green (bound) but FC doesn't see it
- `RXLOSS` appears as arming prevention flag in CLI `status` command

### Common Causes
- **TX and RX wires are swapped** (most common!) - remember: RX pad on receiver connects to TX pad on FC
- Wrong UART selected in Betaflight Ports tab
- Serial RX not enabled on the correct UART
- Wrong receiver protocol selected (should be CRSF for ELRS)
- `serialrx_inverted` or `serialrx_halfduplex` incorrectly set to ON
- F4 flight controller without hardware inverter (SBUS-specific issue)
- Damaged receiver wire or cold solder joint

### Step-by-Step Solution

1. **Verify your wiring. This is the most common cause:**
   ```
   ELRS RX pad → FC TX pad (they cross over!)
   ELRS TX pad → FC RX pad
   5V → 5V
   GND → GND
   ```

2. **In Betaflight Configurator, go to Ports tab:**
   - Find the UART your receiver is connected to
   - Enable "Serial RX" toggle for that UART only
   - Save and Reboot

3. **Go to Configuration tab → Receiver section:**
   - Receiver Mode: `Serial (via UART)`
   - Serial Receiver Provider: `CRSF`
   - Enable Telemetry toggle

4. **In CLI tab, verify these critical settings:**

5. **Power cycle the entire system** (FC + receiver with battery)

6. **Check Receiver tab** - bars should now respond to stick movement

### Commands/Settings

```bash
# In Betaflight CLI, verify these settings:
get serialrx

# Should show:
serialrx_provider = CRSF
serialrx_inverted = OFF
serialrx_halfduplex = OFF

# If wrong, set them:
set serialrx_provider = CRSF
set serialrx_inverted = OFF
set serialrx_halfduplex = OFF
save
```

### 💡 Pro Tip
When in doubt about UART numbers, use Betaflight's Resource Mapping. Type `resource` in CLI to see which UART corresponds to which pads on your specific FC. Look for `SERIAL_RX` assignments.

### ⚠️ Warning
Never rely on the receiver LED alone! A green LED means bound to transmitter, but NOT that the FC is receiving data. Always check the Receiver tab in Betaflight.

---

## Problem 2: ExpressLRS Binding Failure

### Symptoms
- Receiver LED keeps double-blinking (bind mode) but never goes solid
- TX module shows "No connection" or dashes in ELRS Lua script
- Traditional binding (3 power cycles) doesn't work
- Previously working receiver suddenly won't bind after firmware update

### Common Causes
- **Firmware version mismatch** between TX and RX (major version must match: 2.x with 2.x, 3.x with 3.x)
- Different binding phrases on TX and RX
- RX has binding phrase set, blocking traditional bind mode
- Regulatory domain mismatch (915MHz vs 868MHz vs 2.4GHz)
- WiFi mode interfering (receiver stuck in WiFi)

### Step-by-Step Solution

1. **Check firmware versions match:**
   - TX: Open ELRS Lua script → shows version at top
   - RX: Connect via WiFi or check configurator log
   - Major versions MUST match (3.x with 3.x)

2. **If using binding phrase (recommended):**
   - Flash BOTH TX and RX with identical binding phrase
   - Binding phrase is set during flashing in ELRS Configurator
   - After flashing, they auto-bind on power-up

3. **If RX has binding phrase and you want traditional bind:**
   - Must reflash RX WITHOUT a binding phrase
   - Or set same phrase on TX

4. **Enter bind mode properly:**
   - RX: Power cycle 3 times quickly (1 sec each)
   - OR: Hold button during power-up
   - RX should double-blink
   - TX: ELRS Lua → [Bind] option

5. **Nuclear option — reflash both:**
   - Download ELRS Configurator
   - Flash TX module (via WiFi or USB)
   - Flash RX (via Betaflight passthrough or WiFi)
   - Use SAME binding phrase on both

### Commands/Settings

```bash
# Check RX version via WiFi:
# 1. Power RX without FC bound (it enters WiFi mode after 60s)
# 2. Connect to "ExpressLRS RX" WiFi (password: expresslrs)
# 3. Navigate to 10.0.0.1 in browser
# 4. Version shown on page

# Betaflight passthrough flashing:
# In Betaflight CLI:
serialpassthrough [UART#] 420000
# Then flash via ELRS Configurator with "Betaflight Passthrough" selected
```

### 💡 Pro Tip
Use a unique binding phrase, such as your pilot name, and flash it to all of your ELRS gear. Binding will then happen automatically when devices power up and you will not need manual binding.

### ⚠️ Warning
ELRS 3.x will not bind with ELRS 2.x. If you update your transmitter module to 3.x, you must update all of your receivers too.

---

## Problem 3: Stick Inputs Reversed/Wrong Channel Order

### Symptoms
- Moving pitch stick makes drone roll
- Throttle stick controls yaw
- Stick movement shows on wrong channel bar in Receiver tab
- Everything seems backwards or mixed up

### Common Causes
- **Channel map mismatch** between radio and Betaflight
- Radio set to wrong channel order (AETR vs TAER)
- Mixes in radio configured incorrectly
- Using Spektrum/JR radio with default TAER order
- Radio in wrong mode (Mode 1 vs Mode 2)

### Step-by-Step Solution

1. **Understand what you're seeing:**
   ```
   Channel Order Meaning:
   A = Aileron (Roll)
   E = Elevator (Pitch)
   T = Throttle
   R = Rudder (Yaw)
   
   Betaflight default: AETR1234
   Spektrum/JR default: TAER1234
   ```

2. **Check your radio's channel order:**
   - EdgeTX/OpenTX: Model Settings → Mixes page
   - Default should be AETR for Betaflight

3. **Fix in Betaflight (easier method):**
   - Go to Receiver tab
   - Change Channel Map dropdown to match your radio
   - Common options: AETR, TAER, AETR1234

4. **Fix in radio (better long-term):**
   - EdgeTX: SYS → Settings → Default Channel Order → AETR
   - Reorder Mixes to: CH1=Ail, CH2=Ele, CH3=Thr, CH4=Rud

5. **Verify in Receiver tab:**
   - Roll stick → Roll bar moves
   - Pitch stick → Pitch bar moves
   - Throttle → Throttle bar moves
   - Yaw → Yaw bar moves

### Commands/Settings

```bash
# Check current channel map in Betaflight CLI:
get map

# Set channel map:
set map = AETR1234   # Standard (FrSky, most radios)
set map = TAER1234   # Spektrum/JR
save

# EdgeTX Default Channel Order:
# SYS → Settings → Default Channel Order → AETR
```

### 💡 Pro Tip
Set your radio's default channel order to AETR once, and every new model you create will automatically match Betaflight's default. No more per-quad configuration!

### ⚠️ Warning
Getting channel mapping wrong causes crashes! Always verify on the bench that each stick moves the correct channel bar BEFORE arming. A reversed pitch/roll will make the drone uncontrollable.

---

## Problem 4: Receiver Loses Connection Mid-Flight

### Symptoms
- Drone suddenly goes into failsafe during flight
- Signal drops at short range (under 500m)
- Intermittent connection loss, then reconnects
- RSSI shows erratic values or sudden drops
- Works fine on bench, fails in air

### Common Causes
- **Antenna not connected properly** (uFL connector not clicked)
- Antenna orientation wrong (pointing at transmitter = worst signal)
- Antenna damaged or wire broken
- Interference from VTX or ESC noise
- Receiver too close to carbon fiber frame
- TX antenna damaged or SMA connector loose

### Step-by-Step Solution

1. **Check receiver antenna connection:**
   - uFL connectors must click firmly into place
   - These are small connectors, so use a fingernail or a plastic tool to press them down
   - If loose, signal is severely degraded

2. **Antenna placement matters:**
   ```
   Good: Antennas at 90° angle, away from frame
   Bad: Antennas flat against carbon fiber
   Worst: Antenna pointing directly at TX
   ```

3. **Get antennas away from interference:**
   - Keep away from VTX antenna
   - Keep away from ESC/motor wires
   - Mount on standoffs or zip-tie away from frame

4. **Check TX side:**
   - Verify TX antenna is tight (SMA finger-tight)
   - Check TX power setting in ELRS Lua
   - Ensure TX antenna matches frequency (2.4GHz vs 900MHz)

5. **For ELRS specifically:**
   - Check Packet Rate vs Range tradeoff
   - 50Hz = maximum range, 500Hz = minimum range
   - Switch Rate Lua setting affects this

### Commands/Settings

```bash
# ELRS Lua Script settings for range:
# Packet Rate: 50Hz (max range) to 500Hz (min latency)
# Telemetry Ratio: 1:2 for racing, 1:128 for long range
# TX Power: 25mW to 1W depending on module

# Check RSSI in Betaflight OSD:
# Add "RSSI Value" element to OSD
# Should show stable 99 on bench
# Below 50 in flight = antenna problem
```

### 💡 Pro Tip
Do a range test before flying! Walk away from quad (armed on bench) while watching RSSI. If it drops below 70 at 50 meters, you have an antenna problem.

### ⚠️ Warning
A loose uFL connector can still show "connected" but with severely reduced range. This is the #1 cause of mid-flight failsafes on new builds. Always click that connector firmly!

---

## Problem 5: SBUS Not Working on F4 Flight Controller

### Symptoms
- SBUS receiver shows bound but no channel response
- Works fine when tested with IBUS/PPM receiver
- FrSky receiver LED is solid but FC shows no input
- Channel bars in Betaflight are flatlined at 1500

### Common Causes
- **F4 processors lack hardware UART inverter** - the SBUS signal is inverted and needs external inversion
- Connected to non-inverted UART pad instead of SBUS pad
- Wrong UART selected for Serial RX
- `serialrx_inverted` setting wrong for your specific FC

### Step-by-Step Solution

1. **Understand the problem:**
   ```
   SBUS = Inverted signal (logic flipped)
   F4 chips = No hardware inverter on most UARTs
   Solution = Use FC's dedicated SBUS pad OR external inverter
   ```

2. **Check if your FC has a dedicated SBUS pad:**
   - Look for pad labeled "SBUS" or "SB"
   - This pad has hardware inverter
   - Connect SBUS signal wire HERE, not regular RX pad

3. **If no SBUS pad, check FC documentation:**
   - Some F4 FCs have specific inverted UART
   - Often UART1 or UART3 is hardware inverted
   - Check FC manual/pinout diagram

4. **Software inversion (some FCs only):**
   - Some F4 support software inversion
   - Try `set serialrx_inverted = ON`
   - Only works on specific UARTs

5. **If nothing works, use external inverter:**
   - Tiny PCB that inverts signal
   - Connect: RX → Inverter → FC RX pad
   - Or switch to CRSF/ELRS (no inversion needed!)

### Commands/Settings

```bash
# Check inversion capability:
get serialrx_inverted

# Try enabling inversion (F4 with software inversion):
set serialrx_inverted = ON
save

# Make sure protocol is correct:
set serialrx_provider = SBUS
save

# Note: If this doesn't work, you NEED hardware SBUS pad
# Check your FC documentation for which pad has inverter
```

### 💡 Pro Tip
Modern ELRS and Crossfire receivers use the CRSF protocol, which is not inverted, so they work on any UART without special pads. If SBUS keeps causing trouble, upgrading to ELRS or CRSF usually simplifies the setup.

### ⚠️ Warning
Don't keep trying random `serialrx_inverted` settings hoping something works. Either your FC has hardware inversion on a specific pad (use that pad), or it doesn't (you need external inverter or different receiver).

---

## Quick Reference Card

| Symptom | First Check | Likely Fix |
|---------|------------|------------|
| No RX channels | TX/RX wire swap | Swap TX↔RX wires |
| Won't bind | Firmware version | Match TX/RX versions |
| Channels scrambled | Channel map | Set correct AETR/TAER |
| Signal drops | Antenna connector | Click uFL firmly |
| SBUS flatline | SBUS pad | Use dedicated SBUS pad |

---

[← Back to Index](../README.md) | [Next: Flight Controller Issues →](02-flight-controller.md)