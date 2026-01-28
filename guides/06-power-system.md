# Category 6: Power System Issues

> Problems 26-30: Magic Smoke, Brownout, Connectors, Battery Failsafe, Video Noise

---

## Problem 26: Magic Smoke on First Power-Up

### Symptoms
- Smoke appears immediately when battery connected
- Component gets extremely hot
- Burning smell
- Board/component stops working
- May happen with a pop or spark

### Common Causes
- **Polarity reversed** — positive and negative swapped
- **Solder bridge** — unintended connection between pads
- Over-voltage (wrong battery for component)
- Short circuit in wiring
- Defective component (rare, but happens)

### Step-by-Step Solution

1. **STOP! Disconnect immediately:**
   - Do not reconnect to see if it works
   - Once smoke is released, damage is done
   - Further power will cause more damage

2. **Use a smoke stopper BEFORE first power-up:**
   - Device like VIFLY ShortSaver
   - Limits current, prevents damage
   - LED shows short circuit
   - Should be used on EVERY new build

3. **Check polarity BEFORE connecting:**
   - Red = Positive (+)
   - Black = Negative (–)
   - Verify with multimeter if unsure
   - Look at connector orientation

4. **Inspect for solder bridges:**
   - Look between pads with magnification
   - Check filter capacitor pads especially
   - Remove bridges with solder wick

5. **After smoke — identify damaged component:**
   - Burnt/discolored component
   - Damaged capacitor (bulging)
   - Often FC, ESC, or VTX is victim

### Commands/Settings

```
No software settings can help here — this is hardware damage.

Prevention Checklist (BEFORE power-up):
□ Battery polarity verified
□ No loose wires that could touch
□ Solder joints inspected (no bridges)
□ Smoke stopper connected
□ Multimeter continuity test on power wires

VIFLY ShortSaver Usage:
1. Connect smoke stopper between battery and quad
2. Plug in battery
3. Green LED = safe to connect direct
4. Red LED = SHORT CIRCUIT, do not proceed

Post-Smoke Steps:
1. Visual inspection for damage
2. Replace damaged components
3. Recheck ALL wiring
4. Test with smoke stopper
```

### 💡 Pro Tip
Invest in a smoke stopper (₹500-1000) — it will save you from destroying expensive components. Use it for EVERY first power-up and when debugging. One saved FC pays for 10 smoke stoppers.

### ⚠️ Warning
Never apply power to a build without a smoke stopper until you're confident it's wired correctly. Polarity reversal will instantly destroy ESCs and FCs. This is the #1 cause of "dead on arrival" builds.

---

## Problem 27: Flight Controller Brownout Under Throttle

### Symptoms
- FC reboots when throttle is applied
- OSD flickers or resets during power-up
- Random disconnects under load
- Works fine at low throttle, fails at high
- Capacitor whine or buzzing

### Common Causes
- **Missing or inadequate capacitor** on power input
- Voltage regulator overloaded
- Poor power wiring (thin wires, bad solder joints)
- ESC noise coupling into FC power
- Battery can't deliver current

### Step-by-Step Solution

1. **Add capacitor to ESC power input:**
   - 470µF to 1000µF, 35V or higher
   - Low-ESR type preferred
   - Mount close to ESC power input
   - Short leads for best effectiveness

2. **Check 5V rail stability:**
   - Measure 5V rail with multimeter
   - Under throttle, should stay 4.8-5.2V
   - If drops below 4.5V = regulator problem

3. **Improve power wiring:**
   - Use appropriate gauge wire:
     - Battery to ESC: 12-14 AWG
     - 5V power: 20-22 AWG
   - Ensure solid solder joints
   - Reduce wire length where possible

4. **Separate FC power source:**
   - Some FCs have separate 5V input
   - Use dedicated BEC instead of ESC's BEC
   - Isolates FC from ESC noise

5. **Check battery health:**
   - Old/damaged batteries sag badly
   - Test battery C-rating vs your amp draw
   - Replace batteries that sag heavily

### Commands/Settings

```bash
# Monitor voltage in flight:
# Betaflight OSD:
# Add "Battery Voltage" element
# Add "Main Batt Voltage" in OSD tab

# Betaflight voltage config:
set vbat_max_cell_voltage = 430   # 4.3V per cell
set vbat_warning_cell_voltage = 350  # 3.5V warning
set vbat_min_cell_voltage = 330   # 3.3V minimum

# ArduPilot voltage monitoring:
BATT_MONITOR = 4      # Analog voltage
BATT_VOLT_PIN = [pin] # Check FC pinout
BATT_VOLT_MULT = [calibrated value]
BATT_LOW_VOLT = 3.5   # Per cell
BATT_CRT_VOLT = 3.3   # Critical per cell

# Log voltage for analysis:
# Betaflight: Enable Blackbox
# ArduPilot: Enable LOG_BATT_MASK
```

### 💡 Pro Tip
The "capacitor fix" is almost universal — if you're having any power-related issues, add a big capacitor. Even if the ESC/FC has one, an extra external cap helps filter motor noise and prevent brownouts.

### ⚠️ Warning
Brownouts cause loss of control! The FC reboots mid-flight, motors stop, drone falls. Fix power issues before flying. If you see OSD flicker on throttle punch, don't fly until fixed.

---

## Problem 28: Battery Connector Getting Hot

### Symptoms
- XT60/XT30 connector hot after flight
- Melting smell from connector
- Connector discolored (yellowing)
- Resistance felt when plugging in
- One side hotter than the other

### Common Causes
- **Cold solder joints** — insufficient heat during soldering
- Wire gauge too thin for current
- Damaged/worn connector pins
- Poor contact between male/female
- Using wrong connector for amp draw

### Step-by-Step Solution

1. **Inspect solder joints:**
   - Should be shiny, not dull/grainy
   - Wire should be fully wetted with solder
   - No gaps between wire and cup
   - Reflow if joints look cold

2. **Use proper wire gauge:**
   | Amps | Wire Gauge |
   |------|------------|
   | <15A | 16 AWG |
   | 15-30A | 14 AWG |
   | 30-60A | 12 AWG |
   | >60A | 10 AWG |

3. **Check connector condition:**
   - Pins should be shiny, not blackened
   - Tight fit when connected
   - Replace if pins are loose or damaged
   - Clean with contact cleaner

4. **Consider upgrading connector:**
   - XT60: Good for up to 60A
   - XT90: For high-power 6S builds
   - Don't use XT30 for >30A continuous

5. **Resoldering technique:**
   - Pre-tin wire and connector cup
   - Use adequate heat (60W iron minimum)
   - Heat connector, flow solder in
   - Hold until fully cooled

### Commands/Settings

```
No software settings — this is hardware issue.

Soldering Best Practices:
1. Pre-tin wire (flux + solder on wire)
2. Pre-tin connector cup (fill 2/3 with solder)
3. Insert wire into hot solder pool
4. Hold still until cooled
5. Inspect: wire fully covered, shiny joint

Heat Calculation:
Power = Current² × Resistance
Hot connector = high resistance = bad joint

Testing:
- Measure resistance across connector
- Should be < 0.01 ohm
- Use multimeter's 200mΩ range
```

### 💡 Pro Tip
Heat shrink doesn't hide bad solder joints — it traps heat and makes the problem worse! Only use heat shrink AFTER verifying your solder joints are good with a tug test and visual inspection.

### ⚠️ Warning
Hot connectors can cause fires! A bad solder joint can reach 200°C+ during flight, melting the connector, wires, and potentially the LiPo. If your connector is hot after flight, fix it immediately before next flight.

---

## Problem 29: Battery Failsafe Triggers at Wrong Voltage

### Symptoms
- Failsafe triggers when battery is still charged
- Shows wrong voltage on OSD
- Voltage reading fluctuates wildly
- Failsafe never triggers (dangerous!)
- Different reading than battery charger shows

### Common Causes
- **Voltage divider/multiplier not calibrated**
- Wrong cell count configured
- Voltage pin not connected
- ADC reference voltage issue
- Loose battery sense wire

### Step-by-Step Solution

1. **Calibrate voltage multiplier:**
   - Measure battery with multimeter
   - Compare to FC reading
   - Adjust BATT_VOLT_MULT until they match

2. **Set correct cell count:**
   - Don't rely on auto-detect for failsafe
   - Set fixed thresholds per cell
   - 4S = 4 cells, 6S = 6 cells

3. **Configure failsafe thresholds:**
   - Warning: 3.5V per cell
   - Critical: 3.3V per cell
   - Example 4S: Warn at 14V, Critical at 13.2V

4. **Check voltage sense wiring:**
   - Dedicated VBAT sense pin on FC
   - Not same as main power input
   - Should be connected to battery positive

5. **Verify with multimeter:**
   - Connect battery
   - Measure at battery terminals
   - Compare to OSD/GCS reading
   - Adjust until <0.2V difference

### Commands/Settings

```bash
# Betaflight Voltage Calibration:
# Configuration tab → Battery Voltage
# Enter Scale value until OSD matches multimeter

# CLI method:
set vbat_scale = 110      # Default, adjust as needed
set vbat_max_cell_voltage = 430
set vbat_min_cell_voltage = 330
set vbat_warning_cell_voltage = 350
save

# ArduPilot Calibration:
# Measure battery voltage with multimeter
# Set measured voltage in:
# Setup → Optional Hardware → Battery Monitor
# Click "Measured Battery Voltage" and enter value

# ArduPilot Parameters:
BATT_VOLT_MULT = 10.1    # Adjust to match multimeter
BATT_AMP_PERVLT = 17     # If using current sensor
BATT_LOW_VOLT = 14.0     # 4S warning (3.5V × 4)
BATT_CRT_VOLT = 13.2     # 4S critical (3.3V × 4)
BATT_FS_LOW_ACT = 1      # RTL on low battery
BATT_FS_CRT_ACT = 1      # RTL on critical

# Verify:
# Connect battery → Check voltage in Status
# Should match multimeter within 0.2V
```

### 💡 Pro Tip
Don't set failsafe too aggressive — voltage sag under load can trigger false failsafes. Test at max throttle: if OSD shows 3.5V/cell but charger puts in only 50%, your calibration is wrong.

### ⚠️ Warning
No failsafe is DANGEROUS — you'll fly until the battery is damaged or the drone falls from the sky. Failing to trigger is worse than triggering too early. Always verify failsafe actually works before relying on it!

---

## Problem 30: Video Noise/Lines When Motors Spin

### Symptoms
- Clean video with motors off, noise when on
- Horizontal lines scrolling through video
- Video static increases with throttle
- OSD text flickering or distorted
- Noise worse with higher throttle

### Common Causes
- **ESC noise coupling into video ground**
- Missing filter capacitor
- VTX and camera sharing ground with ESC
- Poor power filtering to camera/VTX
- Unshielded video cable running near ESC wires

### Step-by-Step Solution

1. **Add capacitor to ESC:**
   - 470-1000µF across battery input
   - Filters motor noise at source
   - Essential for clean video

2. **Use LC filter for video equipment:**
   - Install between ESC 5V and camera/VTX
   - Filters high-frequency noise
   - Cheap insurance (~₹100-200)

3. **Separate video ground:**
   - Don't use ESC ground for video
   - Dedicated ground from FC 5V
   - Avoid ground loops

4. **Route cables properly:**
   - Keep video cable away from ESC/motor wires
   - Cross at 90° if they must cross
   - Use shielded video cable if needed

5. **Add ferrite rings:**
   - Around camera power leads
   - Around video signal wire
   - Absorbs high-frequency noise

### Commands/Settings

```
No software fix for video noise — this is electrical filtering issue.

Filtering Strategy:
1. ESC Side: 1000µF low-ESR capacitor on battery input
2. VTX Side: LC filter between ESC 5V and VTX
3. Camera Side: LC filter or clean 5V from FC

LC Filter Connection:
Battery/ESC 5V → LC Filter IN → LC Filter OUT → Camera/VTX

Ground Wiring Best Practice:
- Star grounding: All grounds meet at one point
- Separate high-current (ESC) and low-current (video) grounds
- Connect video ground directly to FC, not through ESC

Cable Routing:
- Motor/ESC wires: Away from everything else
- Video cable: Separate path from power
- If must cross, cross at 90°
```

### 💡 Pro Tip
The "capacitor test" is diagnostic: if adding a big cap (1000µF+) directly across battery terminals reduces noise significantly, the problem is definitely ESC noise. Make the cap permanent.

### ⚠️ Warning
Severe video noise can make flying dangerous — you can't see obstacles or orient the drone. If video is unusable at high throttle, don't fly until fixed. Adding filtering is cheap; crashes are expensive.

---

## Quick Reference Card

| Symptom | First Check | Likely Fix |
|---------|------------|------------|
| Magic smoke | Polarity? | Check + and – |
| Brownout | Capacitor? | Add 1000µF cap |
| Hot connector | Solder joints | Reflow connections |
| Wrong voltage | Calibrated? | Adjust VOLT_MULT |
| Video noise | Cap on ESC? | Add cap + LC filter |

---

[← Back to Companion Computer](05-companion-computer.md) | [Next: Telemetry & Ground Station →](07-telemetry-ground-station.md)