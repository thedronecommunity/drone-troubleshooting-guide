# Category 2: Flight Controller & Firmware

> Problems 6-10: Connection, Flashing, Gyro, Settings, Brownouts

---

## Problem 6: Flight Controller Not Detected by Computer

### Symptoms
- No COM port appears when FC is plugged in via USB
- Betaflight Configurator shows "No Port Available"
- Device Manager shows "Unknown Device" or nothing
- FC powers on (LED lights) but computer doesn't see it
- Works on one computer but not another

### Common Causes
- **Using a charge-only USB cable** (no data wires) - this is the most common cause
- Missing or wrong USB drivers (Windows)
- Damaged USB port on FC
- FC stuck in DFU mode
- STM32 Virtual COM Port driver not installed
- USB cable too long or low quality

### Step-by-Step Solution

1. **First, try a different USB cable:**
   - Use cable that came with FC or a known data cable
   - Charge-only cables look identical but won't work
   - If in doubt, try 3-4 different cables

2. **Install proper drivers (Windows):**
   - Download ImpulseRC Driver Fixer
   - Run as Administrator
   - Click "Install" to fix most driver issues
   - Replug FC after installation

3. **Check Device Manager:**
   - Open Device Manager (Win+X → Device Manager)
   - Look under "Ports (COM & LPT)"
   - If showing under "Other devices" with ⚠️ = driver issue

4. **If FC shows in DFU mode:**
   - It will appear as "STM32 BOOTLOADER" in Device Manager
   - Use Zadig to install WinUSB driver
   - Or flash firmware via DFU to restore normal operation

5. **Linux users:**
   ```bash
   # Add user to dialout group
   sudo usermod -a -G dialout $USER
   # Log out and back in
   
   # Check if FC detected
   dmesg | tail -20
   ls /dev/ttyACM*
   ```

### Commands/Settings

```bash
# Windows - ImpulseRC Driver Fixer:
# Download from: https://impulserc.com/pages/downloads
# Run as Admin → Click Install

# Windows - Manual driver (if needed):
# 1. Device Manager → Find device with ⚠️
# 2. Right-click → Update Driver
# 3. Browse → Let me pick → STMicroelectronics Virtual COM Port

# Check if Zadig needed for DFU mode:
# Download Zadig: https://zadig.akeo.ie/
# Options → List All Devices
# Select STM32 BOOTLOADER
# Install WinUSB driver
```

### 💡 Pro Tip
Keep one USB cable that you know is a data capable cable and mark it with tape or a label. Charge only cables are the number one cause of "FC not detected" problems and often waste a lot of time.

### ⚠️ Warning
Avoid installing random drivers. Use ImpulseRC Driver Fixer first because it handles STM32 VCP drivers correctly. Installing the wrong driver, such as old joystick or modem drivers, can prevent the FC from appearing as a serial device.

---

## Problem 7: Cannot Flash Firmware / DFU Mode Issues

### Symptoms
- Betaflight shows "Failed to open port" when flashing
- Flashing starts but fails partway through
- FC won't enter DFU/bootloader mode
- "No DFU device found" error
- Progress bar freezes during flash

### Common Causes
- FC not in DFU/bootloader mode
- Wrong DFU driver installed (Windows)
- Trying to flash wrong firmware target
- USB connection unstable during flash
- Antivirus blocking the flash process

### Step-by-Step Solution

1. **Enter DFU mode correctly:**
   - Method 1: Hold BOOT button, plug in USB, release button
   - Method 2: In CLI type `bl` then press Enter (soft boot to DFU)
   - Method 3: Bridge BOOT pads with tweezers while plugging in

2. **Verify DFU mode:**
   - Betaflight Configurator → should show "DFU" in port dropdown
   - Device Manager → shows "STM32 BOOTLOADER"

3. **Fix DFU drivers (Windows):**
   - Download Zadig
   - Options → List All Devices
   - Find "STM32 BOOTLOADER"
   - Replace driver with WinUSB
   - Unplug/replug FC

4. **Select correct firmware target:**
   - Check FC documentation for correct target name
   - Wrong target = won't work or can brick FC
   - When in doubt, search "YourFCName Betaflight target"

5. **Flash process:**
   - Enable "Full Chip Erase" for clean install
   - Keep "No Reboot Sequence" checked for DFU
   - Click "Load Firmware [Online]" → Select correct target
   - Click "Flash Firmware"

### Commands/Settings

```bash
# Soft boot to DFU from Betaflight CLI:
bl

# After flash, if FC doesn't connect:
# 1. Unplug USB
# 2. Wait 5 seconds
# 3. Plug back in (should boot normally)

# If FC seems bricked after bad flash:
# 1. Enter DFU mode via BOOT button
# 2. Flash correct firmware
# 3. Full chip erase is essential
```

### 💡 Pro Tip
Before flashing, save your CLI `diff all` output! After flash, you can restore all your settings by pasting the diff back into CLI. Otherwise, you're reconfiguring everything from scratch.

### ⚠️ Warning
Flashing the wrong firmware target can appear to work but will cause strange behavior or brick the FC. Triple-check your target name matches your FC exactly. Look on the FC board itself — target name is often printed there.

---

## Problem 8: "No Gyro Detected" After Flashing

### Symptoms
- Betaflight shows "No Gyro Detected" error
- Setup tab shows gyro values as 0 or no movement
- Just flashed firmware and it was working before
- Model preview doesn't respond to movement

### Common Causes
- **Wrong firmware target flashed** — different FC hardware
- Unified Target settings not applied
- Gyro damaged from impact or ESD
- Board configuration not matching hardware
- Custom mixer/resource overriding gyro settings

### Step-by-Step Solution

1. **Verify you flashed correct target:**
   - Check FC documentation for exact target name
   - Some FCs have multiple variants (V1, V2, etc.)
   - Target printed on FC board itself

2. **For Unified Targets (newer FCs):**
   - After flashing, go to CLI
   - Type `board_name` — if empty, need to set it
   - Apply board configuration:
   ```
   board_name MANUFACTURER_MODELNAME
   save
   ```

3. **Try reapplying board config:**
   - In Configurator → Presets tab
   - Search for your FC name
   - Apply Official Board Configuration preset
   - Save and Reboot

4. **Full reflash with chip erase:**
   - Enter DFU mode
   - Enable "Full Chip Erase" checkbox
   - Flash firmware again
   - Apply board configuration

5. **If gyro truly dead (impact damage):**
   - No software fix possible
   - Consider FC replacement
   - Some FCs have backup gyro (check docs)

### Commands/Settings

```bash
# Check current board name:
board_name

# Set board name for Unified Target:
board_name MATEKF411
save

# List available configurations:
# Betaflight Configurator → Presets → Search FC name

# Check gyro detection in CLI:
status
# Look for "Gyro" line - should show detected gyro model

# Force gyro detection:
set gyro_1_sensor_align = CW0
set gyro_1_align_yaw = 0
save
```

### 💡 Pro Tip
When buying F4/F7 FCs, note the exact target name and save it somewhere. FCs often look identical but use different targets. A photo of the FC label saves headaches later.

### ⚠️ Warning
"No Gyro Detected" after a crash usually means physical damage. Software reflashing won't fix a dead gyro chip. Check for visible damage on the gyro (small square chip near center of FC).

---

## Problem 9: CLI Settings Not Saving

### Symptoms
- Change settings in CLI, they revert after reboot
- `save` command appears to work but settings lost
- Settings save sometimes but not always
- Error messages about EEPROM

### Common Causes
- **Forgetting to type `save`** after CLI commands
- Corrupted EEPROM from power loss during save
- EEPROM wear (very rare on new FCs)
- USB disconnecting during save
- Settings conflict causing fallback to defaults

### Step-by-Step Solution

1. **Always end CLI session with `save`:**
   ```
   set motor_pwm_protocol = DSHOT300
   set deadband = 3
   save      ← REQUIRED!
   ```

2. **Verify settings saved:**
   - After `save`, FC reboots
   - Reconnect and type `get [setting_name]`
   - Confirm it shows new value

3. **If settings keep reverting:**
   - Back up working parts: `diff all` → save to file
   - Full reset: `defaults` then `save`
   - Restore essentials from saved diff

4. **Corrupted EEPROM fix:**
   - Enter DFU mode
   - Flash firmware with "Full Chip Erase"
   - This wipes EEPROM completely
   - Reconfigure from scratch or restore diff

5. **Check for conflicting settings:**
   - Some settings conflict and cause resets
   - Watch for errors in CLI output
   - Configure one feature at a time, save, verify

### Commands/Settings

```bash
# Proper CLI workflow:
set acc_calibration = 1,2,3,4,5,6
save                 # ← DON'T FORGET!

# View all changed settings:
diff all

# Back up configuration:
diff all
# Copy entire output to text file

# Restore from backup:
# Paste diff output into CLI
# Then: save

# Factory reset if corrupted:
defaults
save

# Full EEPROM wipe (via DFU flash):
# 1. Enter DFU mode
# 2. Enable "Full Chip Erase" 
# 3. Flash firmware
```

### 💡 Pro Tip
After any major configuration session, run `diff all` and save the output to a text file named with date and quad name. This is your restore point if anything goes wrong.

### ⚠️ Warning
Never unplug USB while `save` is processing! The EEPROM write takes a moment. Interrupting it can corrupt settings. Wait for the FC to reboot automatically.

---

## Problem 10: Random Reboots/Freezes

### Symptoms
- FC reboots randomly during flight
- Drone falls out of sky momentarily then recovers
- FC freezes and stops responding to inputs
- OSD shows brief blackout then returns
- Happens more with high throttle

### Common Causes
- **Brownout** — voltage drops below FC operating threshold
- Power supply can't handle current spikes
- Poor solder joints on power connections
- Missing capacitor on battery input
- Damaged voltage regulator on FC
- Short circuits causing voltage drops

### Step-by-Step Solution

1. **Check power wiring:**
   - Inspect battery → ESC → FC power path
   - Look for cold solder joints
   - Reflow any suspicious connections

2. **Add/check low-ESR capacitor:**
   - 470µF to 1000µF, 35V or higher rating
   - Install as close to ESC power input as possible
   - This filters voltage spikes from motors

3. **Check battery voltage under load:**
   - Connect battery
   - Punch throttle briefly (no props!)
   - Watch OSD voltage — shouldn't drop more than 0.5V

4. **Verify FC power supply:**
   - FC 5V rail should be stable 4.9-5.1V
   - Measure at FC 5V pad with multimeter
   - Check during motor spin-up

5. **Look for short circuits:**
   - Visual inspection for solder bridges
   - Check resistance between battery + and -
   - Should be high resistance (not near zero)

### Commands/Settings

```bash
# Check voltage in Betaflight:
# Setup tab → shows battery voltage
# OSD → enable Voltage element

# Log brownout events:
# Enable blackbox logging
set blackbox_device = SDCARD  # or SPIFLASH
save

# After flight, analyze logs for voltage drops
# Betaflight Blackbox Explorer shows voltage graph

# Set conservative voltage warning:
set vbat_warning_cell_voltage = 350  # 3.5V per cell
save
```

### 💡 Pro Tip
The "punch test" reveals power issues: with props OFF, do quick throttle punches while watching voltage. If voltage sags badly (>1V drop), you have weak batteries or poor wiring. Fix this before flying!

### ⚠️ Warning
Random reboots during flight are DANGEROUS — the drone becomes uncontrollable briefly. Always diagnose and fix before flying. This is usually a power/wiring issue, not software.

---

## Quick Reference Card

| Symptom | First Check | Likely Fix |
|---------|------------|------------|
| No COM port | USB cable | Try different cable |
| Won't enter DFU | BOOT button | Hold boot, then plug in |
| No Gyro | Firmware target | Flash correct target |
| Settings don't save | Typed `save`? | Add `save` command |
| Random reboots | Power wiring | Add capacitor, check solder |

---

[← Back to Receiver Issues](01-receiver-radio-issues.md) | [Next: ESC & Motor Issues →](03-esc-motor-issues.md)