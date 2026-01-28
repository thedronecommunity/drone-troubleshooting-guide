# Category 3: ESC & Motor Issues

> Problems 11-15: Motor Spinning, Direction, BLHeli, Desync, Grinding

---

## Problem 11: Motors Not Spinning in Betaflight

### Symptoms
- Motor sliders in Motors tab show values but motors don't spin
- Motors beep once on power-up but won't respond to slider
- Works with transmitter sticks but not Motor tab
- All motors same issue, not just one

### Common Causes
- **No battery connected** — ESCs need battery power, USB isn't enough
- Motor Protocol mismatch (DSHOT vs PWM)
- ESC signal wires not connected to FC motor outputs
- "Motor Stop" enabled but slider below arm threshold
- I²C mode enabled on ESC (rare, some BLHeli32)
- "Props on" safety feature blocking motor test

### Step-by-Step Solution

1. **Connect a battery!**
   - ESCs require battery power to spin motors
   - USB only powers FC, not ESCs
   - This is the #1 cause of "motors won't spin"

2. **Verify Motor Protocol matches ESC:**
   - Most modern ESCs: DSHOT300 or DSHOT600
   - Older ESCs: ONESHOT125 or PWM
   - Check ESC documentation

3. **Check signal wire connections:**
   - ESC signal wire → FC motor pad (M1, M2, M3, M4)
   - Ground wire may or may not be needed (check FC docs)
   - Verify no loose/broken wires

4. **In Betaflight Motors tab:**
   - Check "I understand the risks" checkbox
   - Remove all propellers first!
   - Slide motor slider above ~1050 value

5. **Check motor output in CLI:**
   ```
   resource
   # Look for MOTOR 1-4 assignments
   # Verify they match your wiring
   ```

### Commands/Settings

```bash
# Check motor protocol:
get motor_pwm_protocol

# Set motor protocol (match your ESC):
set motor_pwm_protocol = DSHOT300   # Most common
# Options: PWM, ONESHOT125, ONESHOT42, MULTISHOT, DSHOT150, DSHOT300, DSHOT600

save

# Verify motor output pins:
resource
# Shows: MOTOR 1 A03, MOTOR 2 A02, etc.

# Check if motors enabled:
get motor
# motor_pwm_protocol should match ESC capability
```

### 💡 Pro Tip
Do the "ESC beep test" — when you plug in battery (without USB), you should hear the ESC startup melody (typically 3 beeps then 2 beeps for DSHOT). No beeps = power or ESC issue, not FC issue.

### ⚠️ Warning
ALWAYS remove propellers before testing motors! The motor test ignores all safety checks. A prop can cause serious injury in an instant. Take props off, put them on the other side of the room.

---

## Problem 12: Wrong Motor Direction/Order

### Symptoms
- Drone flips immediately on takeoff
- Individual motor spins wrong direction
- Motor numbers don't match Betaflight diagram
- Drone yaws uncontrollably or spins
- "Tip test" fails (push front down, front motors should speed up)

### Common Causes
- **Motor wires soldered to ESC in wrong order** (reverses direction)
- Motor mapping doesn't match physical position
- "Props In" vs "Props Out" setting wrong
- ESC motor direction not configured in BLHeli
- FC orientation not set correctly

### Step-by-Step Solution

1. **Understand the Betaflight motor layout:**
   ```
         FRONT
     4 ↺     2 ↻
           X
     3 ↻     1 ↺
         BACK
   
   ↺ = Counter-clockwise (CCW)
   ↻ = Clockwise (CW)
   ```

2. **Verify motor ORDER (which motor is which):**
   - In Motors tab, raise Motor 1 slider
   - Note which physical motor spins
   - Should be rear-right (per diagram above)
   - Repeat for motors 2, 3, 4

3. **Verify motor DIRECTION:**
   - Motor 1 (rear-right): CCW
   - Motor 2 (front-right): CW
   - Motor 3 (rear-left): CW
   - Motor 4 (front-left): CCW

4. **Fix motor direction in BLHeli:**
   - Open BLHeli Configurator
   - Click "Read Setup"
   - Change "Motor Direction" for wrong motors: Normal ↔ Reversed
   - Write settings

5. **If motor ORDER is wrong:**
   - Either re-solder signal wires to correct pads
   - Or remap in Betaflight CLI:
   ```
   resource MOTOR 1 [new_pin]
   ```

### Commands/Settings

```bash
# Check motor resource mapping:
resource | grep MOTOR

# Check if "Props Out" mode (reversed):
get yaw_motors_reversed
# OFF = Props In (default), ON = Props Out

# Motor direction in Betaflight (if ESC supports):
# Configuration tab → ESC/Motor Features → Motor Direction

# BLHeli Configurator:
# 1. Connect FC with battery
# 2. Close Betaflight (important!)
# 3. Open BLHeli Configurator
# 4. Read Setup
# 5. Change "Motor Direction" for each ESC: Normal or Reversed
# 6. Write Setup
```

### 💡 Pro Tip
Do the "Tip Test" before every first flight: arm the quad, tilt it forward — front motors should speed up to compensate. Tilt left — right motors speed up. If response is opposite, motor direction is wrong!

### ⚠️ Warning
Wrong motor direction = instant flip on takeoff. This can damage the drone and is dangerous. ALWAYS verify with tip test before first flight, props off!

---

## Problem 13: BLHeli Won't Connect to ESCs

### Symptoms
- "Reading Setup" shows no ESCs detected
- BLHeli Configurator connects to FC but can't see ESCs
- Timeout errors when trying to read ESC settings
- Individual ESCs missing (shows 3 of 4)

### Common Causes
- **Betaflight Configurator still open** — must close it first!
- No battery connected (ESCs need power)
- Wrong BLHeli version for your ESCs
- Disabled "ESC Passthrough" on UART
- FC doesn't support passthrough on that port

### Step-by-Step Solution

1. **Close Betaflight Configurator completely!**
   - BLHeli needs exclusive serial port access
   - Having Betaflight open blocks this
   - Fully close, not just disconnect

2. **Connect battery to power ESCs:**
   - USB only powers FC
   - ESCs need battery power to communicate
   - BLHeli talks through FC → ESC signal wire

3. **Use correct BLHeli version:**
   - BLHeli_S ESCs → BLHeli Suite or Configurator
   - BLHeli_32 ESCs → BLHeli_32 Suite
   - Wrong version = ESCs not recognized

4. **Disable peripherals temporarily:**
   - VTX SmartAudio/Tramp can interfere
   - Go to Betaflight Ports tab
   - Disable all peripherals except Serial RX
   - Save, disconnect, try BLHeli again

5. **If some ESCs missing:**
   - Signal wire issue for that motor
   - ESC might be damaged
   - Try reseating connections

### Commands/Settings

```bash
# Before using BLHeli:
# 1. Close Betaflight Configurator
# 2. Connect battery
# 3. Open BLHeli Configurator/Suite

# If using BLHeli Suite (Windows):
# Connect via USB, select correct COM port
# Click "Read Setup"

# Disable peripherals via CLI (temporary):
serial 0 0 115200 57600 0 115200
serial 1 0 115200 57600 0 115200
# (This disables all UART functions temporarily)
# Re-enable after BLHeli configuration

# Check MOTOR protocol supports passthrough:
get motor_pwm_protocol
# DSHOT300/600 support bidirectional passthrough
```

### 💡 Pro Tip
BLHeli_S vs BLHeli_32 are completely different software. Check your ESC documentation to know which you have. Using wrong software = nothing detected. BLHeli_32 ESCs often say "32" somewhere on the ESC.

### ⚠️ Warning
Never update ESC firmware unless you know exactly which firmware is correct. Wrong ESC firmware can permanently brick the ESC. If in doubt, just change settings, don't update firmware.

---

## Problem 14: ESC Desync / Motor Stuttering in Flight

### Symptoms
- Motors stutter or hesitate during aggressive maneuvers
- Sudden power loss then recovery
- One motor sounds different/rough during flight
- "Motor desync" or "flyaway" in flight
- Happens more with low throttle + high demand

### Common Causes
- **Motor timing too high** for motor/prop combo
- **Rampup power too aggressive**
- Missing capacitor causing voltage noise
- Cheap ESCs overwhelmed by demand
- Motor/prop mismatch (too much prop for motor)
- PWM frequency not optimal for motor

### Step-by-Step Solution

1. **Add capacitor if missing:**
   - 470-1000µF low-ESR capacitor
   - Close to ESC power input
   - Filters voltage noise that causes desync

2. **Lower motor timing in BLHeli:**
   - Open BLHeli Configurator/Suite
   - Reduce "Motor Timing" from High to Medium or Low
   - Start conservative, increase if needed

3. **Reduce rampup power:**
   - BLHeli_32: Reduce "Rampup Power"
   - Prevents ESC from demanding too much too fast
   - Default 100% → try 75% or 50%

4. **Check demag compensation:**
   - BLHeli → "Demag Compensation"
   - High = aggressive, can cause desync
   - Try Medium or Low

5. **Motor/prop combo check:**
   - Heavy props on small motors = desync prone
   - Use prop/motor combos recommended by manufacturer
   - Lower pitch prop = less demanding on ESC

### Commands/Settings

```bash
# BLHeli_S Settings for desync prevention:
Motor Timing: Medium (try Low if still issues)
PWM Frequency: 24kHz (default)
Demag Compensation: Medium (or Low)

# BLHeli_32 Settings:
Motor Timing: 16-22° (default usually 22°)
Rampup Power: 50-75% (instead of 100%)
PWM Frequency: 24kHz

# Betaflight motor settings:
set motor_pwm_protocol = DSHOT300
# Try DSHOT300 instead of DSHOT600 if issues persist
save
```

### 💡 Pro Tip
Desync almost always happens during "blip" scenarios — quick throttle chops followed by punch-outs. If this describes your flying, focus on rampup power and timing settings. Racing pilots are most affected.

### ⚠️ Warning
Desync can cause crashes! If you're experiencing desync, lower your rampup and timing before your next flight. A conservative ESC tune is safer than an aggressive one.

---

## Problem 15: Motor Grinding/Stuttering on Startup

### Symptoms
- Motor makes grinding noise when starting
- Motor stutters at low RPM but fine at high
- Motor hesitates before spinning up
- One motor sounds different from others
- Motor gets hot quickly

### Common Causes
- **Motor screws too long** — hitting stator windings
- Debris inside motor (dirt, grass, sand)
- Damaged motor bearings
- Bent motor shaft
- Motor winding damaged (short circuit)
- ESC timing settings wrong

### Step-by-Step Solution

1. **Check motor screws first:**
   - Remove motor from frame
   - If screws too long, they hit stator
   - Measure screw length vs motor depth
   - Use shorter screws or add washers

2. **Inspect for debris:**
   - Look inside motor bell (magnetic part)
   - Remove any grass, dirt, sand
   - Clean with compressed air
   - Careful not to damage windings

3. **Test motor freely:**
   - Remove motor from frame
   - Spin by hand — should be smooth
   - Gritty feeling = bearing damage
   - Catching = debris or winding damage

4. **Check bearings:**
   - Spin motor bell by hand
   - Should spin freely 5+ rotations
   - Grinding/roughness = bad bearings
   - Bearings are replaceable on some motors

5. **If windings damaged:**
   - Look for discolored (burnt) winding wire
   - Check for visible damage to copper wires
   - Damaged windings = replace motor

### Commands/Settings

```bash
# No software fix for mechanical issues
# But check ESC settings aren't making it worse:

# BLHeli:
Motor Timing: Medium (High can stress weak motors)
Startup Power: Medium (not Low)

# If motor runs rough but is mechanically OK:
# Try Complementary PWM: ON (BLHeli_32)
# Can help with startup smoothness
```

### 💡 Pro Tip
The #1 cause of motor grinding on new builds is screws too long! Standard motor mounting screws are often too long for the motor. Always test-fit the screw depth before mounting — it should NOT touch the stator inside.

### ⚠️ Warning
A motor that's grinding will get hot fast and can burn out. Don't keep flying hoping it gets better. Diagnose and fix before the motor fails mid-flight or causes a fire.

---

## Quick Reference Card

| Symptom | First Check | Likely Fix |
|---------|------------|------------|
| Motors won't spin | Battery connected? | Plug in battery |
| Wrong direction | BLHeli direction | Reverse in BLHeli |
| BLHeli won't connect | Betaflight closed? | Close Betaflight |
| Desync in flight | Capacitor installed? | Add capacitor, lower timing |
| Grinding noise | Screw length | Use shorter screws |

---

[← Back to Flight Controller](02-flight-controller.md) | [Next: GPS & Compass →](04-gps-compass.md)