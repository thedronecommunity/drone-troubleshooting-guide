# Category 5: Companion Computer Integration

> Problems 21-25: Raspberry Pi, MAVLink, Dronekit, Serial Port, Camera Feed

---

## Problem 21: Raspberry Pi Can't Communicate with Flight Controller

### Symptoms
- MAVLink scripts show "No heartbeat received"
- `mavproxy.py` can't connect
- Serial port opens but no data
- Works via USB to laptop but not Pi to FC
- Connection attempt hangs indefinitely

### Common Causes
- **Serial port not enabled on Pi** (disabled by default)
- **Bluetooth using serial port** (conflicts on Pi 3/4)
- TX/RX wires swapped
- Wrong baud rate
- UART not configured for MAVLink on FC
- Wrong serial port (ttyS0 vs ttyAMA0)

### Step-by-Step Solution

1. **Enable serial port on Raspberry Pi:**
   ```bash
   sudo raspi-config
   # Interface Options → Serial Port
   # Login shell over serial: NO
   # Serial port hardware: YES
   sudo reboot
   ```

2. **Disable Bluetooth (frees up ttyAMA0):**
   ```bash
   sudo nano /boot/config.txt
   # Add at end:
   dtoverlay=disable-bt
   # Save and reboot
   ```

3. **Verify wiring:**
   ```
   Pi TX (GPIO14/Pin8) → FC RX
   Pi RX (GPIO15/Pin10) → FC TX
   Pi GND → FC GND
   ```

4. **Configure FC UART for MAVLink:**
   - ArduPilot: 
     - `SERIAL2_PROTOCOL = 2` (MAVLink2)
     - `SERIAL2_BAUD = 921` (921600)

5. **Test connection:**
   ```bash
   # Test serial port exists:
   ls /dev/ttyAMA0
   
   # Simple serial test:
   sudo apt install screen
   screen /dev/ttyAMA0 921600
   # Should see MAVLink data (garbled text = good sign!)
   ```

### Commands/Settings

```bash
# Raspberry Pi Setup:
# Enable serial hardware:
sudo raspi-config
# → Interface Options → Serial Port → No/Yes

# Disable Bluetooth (Pi 3/4/5):
echo "dtoverlay=disable-bt" | sudo tee -a /boot/config.txt
sudo systemctl disable hciuart
sudo reboot

# Check serial port:
ls -la /dev/serial*
# Should show /dev/ttyAMA0 or /dev/ttyS0

# ArduPilot FC Configuration (TELEM2 port):
SERIAL2_PROTOCOL = 2    # MAVLink2
SERIAL2_BAUD = 921      # 921600 baud

# MAVProxy test:
mavproxy.py --master=/dev/ttyAMA0 --baudrate=921600

# Python test:
from pymavlink import mavutil
conn = mavutil.mavlink_connection('/dev/ttyAMA0', baud=921600)
conn.wait_heartbeat()
print("Connected!")
```

### 💡 Pro Tip
Use 921600 baud for companion computer connections — it's much faster than the default 57600 and essential for streaming attitude/position data at high rates. Both FC and Pi must match this baud rate.

### ⚠️ Warning
The Raspberry Pi's GPIO pins are 3.3V! Most flight controllers are 5V tolerant on RX, but verify your FC specs. Sending 5V into the Pi's RX pin will damage it.

---

## Problem 22: Permission Denied on Serial Port (/dev/ttyACM0)

### Symptoms
- `Permission denied: '/dev/ttyACM0'` error
- Script works with `sudo` but not normally
- MAVProxy/Dronekit fails to open serial port
- Serial port visible but can't access

### Common Causes
- **User not in dialout group** (required for serial access)
- ModemManager grabbing the port
- Wrong permissions on device file
- udev rules not configured

### Step-by-Step Solution

1. **Add user to dialout group:**
   ```bash
   sudo usermod -a -G dialout $USER
   # MUST log out and back in for this to take effect!
   ```

2. **Verify group membership:**
   ```bash
   # After logging back in:
   groups
   # Should include "dialout"
   ```

3. **Disable ModemManager (it interferes):**
   ```bash
   sudo systemctl disable ModemManager
   sudo systemctl stop ModemManager
   ```

4. **Create udev rule for consistent access:**
   ```bash
   sudo nano /etc/udev/rules.d/99-pixhawk.rules
   # Add:
   SUBSYSTEM=="tty", ATTRS{idVendor}=="26ac", MODE="0666"
   # Save and reload:
   sudo udevadm control --reload-rules
   # Replug device
   ```

5. **Quick fix (temporary):**
   ```bash
   sudo chmod 666 /dev/ttyACM0
   # Resets after reboot
   ```

### Commands/Settings

```bash
# Add to dialout group (permanent fix):
sudo usermod -a -G dialout $USER
# LOG OUT AND BACK IN!

# Check groups:
groups
# Must show: dialout

# Stop ModemManager from grabbing port:
sudo systemctl stop ModemManager
sudo systemctl disable ModemManager

# Persistent udev rule for Pixhawk:
echo 'SUBSYSTEM=="tty", ATTRS{idVendor}=="26ac", MODE="0666"' | \
  sudo tee /etc/udev/rules.d/99-pixhawk.rules
sudo udevadm control --reload-rules

# Verify permissions:
ls -la /dev/ttyACM0
# Should show: crw-rw-rw- ... (666 permissions)
```

### 💡 Pro Tip
After adding yourself to the dialout group, you MUST log out and log back in (or reboot). Running `groups` in the same terminal won't show the new group until you do this.

### ⚠️ Warning
Don't run your drone scripts as `sudo` in production! It works but creates security risks and file permission issues. Fix the underlying permission problem instead.

---

## Problem 23: MAVLink Connection Unstable / High Packet Loss

### Symptoms
- Connection drops frequently
- "Packet loss" warnings in MAVProxy
- Commands delayed or lost
- Telemetry data gaps
- Works at low data rates, fails at high rates

### Common Causes
- **Baud rate mismatch** between FC and companion
- Serial cable too long or poor quality
- Stream rates set too high for connection
- Buffer overflows on companion
- CPU load too high on companion

### Step-by-Step Solution

1. **Match baud rates exactly:**
   - FC side: `SERIAL2_BAUD = 921` (921600)
   - Pi side: `--baudrate=921600` in MAVProxy
   - Both MUST match!

2. **Reduce stream rates if needed:**
   - ArduPilot: Lower `SR2_*` parameters
   - Start with 10 Hz for all streams
   - Increase as needed

3. **Use mavlink-router for stability:**
   ```bash
   # Better than direct connection for multiple apps
   sudo apt install mavlink-router
   # Routes MAVLink to multiple endpoints
   ```

4. **Monitor packet loss:**
   ```bash
   mavproxy.py --master=/dev/ttyAMA0 --baudrate=921600
   # In MAVProxy:
   link   # Shows link statistics
   ```

5. **Check for serial buffer issues:**
   ```bash
   # Increase buffer if needed:
   stty -F /dev/ttyAMA0 speed 921600 raw -echo
   ```

### Commands/Settings

```bash
# ArduPilot Stream Rates (TELEM2 = Serial2):
SR2_POSITION = 10     # Position at 10 Hz
SR2_EXTRA1 = 10       # Attitude at 10 Hz
SR2_EXTRA2 = 10       # VFR HUD at 10 Hz
SR2_EXTRA3 = 2        # AHRS/System at 2 Hz
SR2_PARAMS = 10       # Parameters
SR2_RAW_SENS = 0      # Raw sensors (disable if not needed)
SR2_RC_CHAN = 0       # RC channels (disable if not needed)

# MAVProxy with optimal settings:
mavproxy.py --master=/dev/ttyAMA0 --baudrate=921600 \
  --source-system=255 --source-component=0

# Python MAVLink with timeout:
from pymavlink import mavutil
conn = mavutil.mavlink_connection('/dev/ttyAMA0', baud=921600)
conn.wait_heartbeat(timeout=10)

# Check connection health:
# In MAVProxy:
link          # Shows packet stats
status        # Shows connection status
```

### 💡 Pro Tip
Use `mavlink-router` if you need multiple applications connecting to the FC simultaneously. It handles message routing and prevents conflicts between QGC, custom scripts, and other tools.

### ⚠️ Warning
High packet loss during flight can cause autonomous missions to fail. Always test your MAVLink connection stability on the ground before flying autonomous missions.

---

## Problem 24: Dronekit Commands Ignored by Flight Controller

### Symptoms
- Dronekit connects but commands do nothing
- `vehicle.mode = VehicleMode("GUIDED")` doesn't work
- Arm command hangs or fails
- Commands accepted but vehicle doesn't respond
- Works in SITL but not real hardware

### Common Causes
- **Pre-arm checks failing** (GPS, compass, etc.)
- Not in correct mode for command (need GUIDED)
- GCS_PID params blocking control
- Vehicle not armable (`vehicle.is_armable` = False)
- Safety switch not pressed

### Step-by-Step Solution

1. **Check vehicle is armable:**
   ```python
   print("Armable:", vehicle.is_armable)
   print("Mode:", vehicle.mode.name)
   print("System Status:", vehicle.system_status.state)
   ```

2. **Wait for GPS and initialization:**
   ```python
   while not vehicle.is_armable:
       print(" Waiting for vehicle to initialize...")
       print(f" GPS: {vehicle.gps_0}")
       time.sleep(1)
   ```

3. **Set GUIDED mode before commanding:**
   ```python
   vehicle.mode = VehicleMode("GUIDED")
   while vehicle.mode.name != 'GUIDED':
       time.sleep(0.5)
   ```

4. **Check and clear pre-arm issues:**
   - GPS lock required (8+ sats)
   - Compass calibrated
   - No EKF errors
   - Battery OK

5. **Press safety switch if equipped:**
   - Hardware switch must be pressed before arming
   - LED goes solid when ready
   - Or disable: `BRD_SAFETY_DEFLT = 0`

### Commands/Settings

```python
# Dronekit Connection and Arming:
from dronekit import connect, VehicleMode
import time

# Connect
vehicle = connect('/dev/ttyAMA0', baud=921600, wait_ready=True)

# Wait for armable
print("Waiting for initialization...")
while not vehicle.is_armable:
    print(f"  GPS: {vehicle.gps_0}")
    print(f"  Battery: {vehicle.battery}")
    print(f"  EKF OK: {vehicle.ekf_ok}")
    time.sleep(1)

# Set GUIDED mode
print("Setting GUIDED mode...")
vehicle.mode = VehicleMode("GUIDED")
while vehicle.mode.name != 'GUIDED':
    time.sleep(0.5)

# Arm
print("Arming...")
vehicle.armed = True
while not vehicle.armed:
    print("  Waiting for arm...")
    time.sleep(1)

print("Armed and ready!")

# ArduPilot params for external control:
# SYSID_MYGCS = 255   # Allow control from any GCS
# Not typically needed, but check if commands blocked
```

### 💡 Pro Tip
Always check `vehicle.is_armable` before trying to arm. It's a compound check that includes GPS, compass, battery, and other pre-arm requirements. If False, check individual components to find what's failing.

### ⚠️ Warning
Never test autonomous commands with props on until you've verified failsafes work! Start with props off, verify arm/disarm, mode changes, then test with props in a safe area.

---

## Problem 25: Camera Feed Not Working/Lagging

### Symptoms
- No video from Pi Camera to ground station
- Video has 5+ second latency
- Feed freezes or stutters
- Works locally but not streaming
- High CPU usage on companion

### Common Causes
- **Not using hardware encoding** (Pi's GPU must encode)
- Wrong GStreamer pipeline
- Network configuration issues
- Camera not enabled in raspi-config
- USB camera compatibility issues

### Step-by-Step Solution

1. **Enable camera in raspi-config:**
   ```bash
   sudo raspi-config
   # Interface Options → Legacy Camera → Enable
   sudo reboot
   ```

2. **Use hardware-accelerated encoding:**
   ```bash
   # For Pi Camera (CSI), use h264 hardware encoder:
   rpicam-vid -t 0 --width 1280 --height 720 --framerate 30 \
     -o - | gst-launch-1.0 fdsrc ! h264parse ! \
     rtph264pay config-interval=1 pt=96 ! \
     udpsink host=GROUND_IP port=5600
   ```

3. **Install GStreamer:**
   ```bash
   sudo apt update
   sudo apt install gstreamer1.0-tools \
     gstreamer1.0-plugins-good \
     gstreamer1.0-plugins-bad
   ```

4. **Test camera locally first:**
   ```bash
   # Test Pi Camera:
   rpicam-still -o test.jpg
   
   # USB camera:
   v4l2-ctl --list-devices
   ffmpeg -f v4l2 -i /dev/video0 -frames 1 test.jpg
   ```

5. **Receiving on ground station:**
   ```bash
   # On ground computer (Linux):
   gst-launch-1.0 udpsrc port=5600 ! \
     application/x-rtp,encoding-name=H264 ! \
     rtph264depay ! h264parse ! avdec_h264 ! \
     autovideosink
   
   # Or use QGroundControl's video settings
   ```

### Commands/Settings

```bash
# Pi Camera 2 with hardware encoding (lowest latency):
rpicam-vid -t 0 --width 1280 --height 720 --framerate 30 \
  --inline --listen -o tcp://0.0.0.0:5600

# Alternative for older Pis:
raspivid -t 0 -w 1280 -h 720 -fps 30 -b 2000000 \
  -o - | gst-launch-1.0 fdsrc ! h264parse ! \
  rtph264pay config-interval=1 pt=96 ! \
  udpsink host=192.168.1.100 port=5600

# USB Camera with v4l2:
gst-launch-1.0 v4l2src device=/dev/video0 ! \
  video/x-raw,width=640,height=480,framerate=30/1 ! \
  videoconvert ! x264enc tune=zerolatency bitrate=1000 ! \
  rtph264pay ! udpsink host=192.168.1.100 port=5600

# Network setup:
# Ground station IP: 192.168.1.100
# Pi IP: 192.168.1.101
# Port: 5600 (UDP)

# QGroundControl Video Settings:
# Video Source: UDP h.264 Video Stream
# Port: 5600
```

### 💡 Pro Tip
The Pi's hardware encoder is essential for low-latency video. Software encoding (libx264) causes high CPU load and 500ms+ latency. Always use `rpicam-vid` or `raspivid` which use the GPU.

### ⚠️ Warning
Streaming high-resolution video over WiFi can interfere with your RC link if both use 2.4GHz. Use 5GHz WiFi for video, or dedicated video TX like DJI/Walksnail. Never lose RC control due to video bandwidth!

---

## Quick Reference Card

| Symptom | First Check | Likely Fix |
|---------|------------|------------|
| No heartbeat | Serial enabled? | `raspi-config` serial |
| Permission denied | In dialout group? | `usermod -a -G dialout` |
| High packet loss | Baud rate match? | Set both to 921600 |
| Commands ignored | `is_armable` true? | Fix pre-arm issues |
| Video lag | Using hardware enc? | Use `rpicam-vid` |

---

[← Back to GPS & Compass](04-gps-compass.md) | [Next: Power System →](06-power-system.md)