# 🚀 Space Rover - Quick Start Guide

## 📦 Installation Checklist

### Raspberry Pi Setup (One Time)

```bash
# 1. Copy files to Pi
scp *.py requirements_rpi.txt setup_rpi.sh pi@raspberrypi.local:~/rover_project/

# 2. SSH into Pi
ssh pi@raspberrypi.local

# 3. Run setup
cd ~/rover_project
bash setup_rpi.sh

# 4. Reboot
sudo reboot
```

### Desktop Setup (For Dashboard)

```bash
# On your computer
pip install -r requirements.txt
```

---

## 🎯 Common Commands

### Testing Components

```bash
# Test all sensors
python3 sensor_manager.py

# Test motors
python3 motor_controller.py

# Test cameras
python3 obstacle_detector.py
```

### Running the Rover

```bash
# Start autonomous operation
python3 rover_main.py

# Stop: Press Ctrl+C
```

### Running Dashboard

```bash
# On any computer
streamlit run streamlit_dashboard.py

# Opens at: http://localhost:8501
```

---

## 🔌 Quick Pin Reference

| Component | Pin | GPIO |
|-----------|-----|------|
| **Motors** |
| Left Motor Enable | ENA | GPIO 13 |
| Left Motor IN1 | IN1 | GPIO 5 |
| Left Motor IN2 | IN2 | GPIO 6 |
| Right Motor Enable | ENB | GPIO 12 |
| Right Motor IN3 | IN3 | GPIO 19 |
| Right Motor IN4 | IN4 | GPIO 26 |
| **IR Sensors** |
| Front IR | OUT | GPIO 17 |
| Back IR | OUT | GPIO 27 |
| Left IR | OUT | GPIO 22 |
| Right IR | OUT | GPIO 23 |
| **DHT11** |
| Temperature/Humidity | DATA | GPIO 4 |
| **pH Sensor (via ADS1115)** |
| I2C Clock | SCL | GPIO 3 |
| I2C Data | SDA | GPIO 2 |

---

## ⚡ Power Connections

```
Motors:
  12V Battery (+) → L298N (12V)
  Battery (-) → L298N (GND) + Pi (GND)  ← IMPORTANT: Common ground!

Raspberry Pi:
  5V Power Bank → Pi 5V pin
  
NEVER connect L298N 5V OUT to Raspberry Pi!
```

---

## 🔧 First Time Setup Steps

### 1. Hardware Assembly
- [ ] Wire all components according to pin reference
- [ ] Double-check power connections
- [ ] Verify common ground between Pi and motor driver
- [ ] Mount cameras in 4 directions

### 2. Software Installation
- [ ] Run `setup_rpi.sh` on Raspberry Pi
- [ ] Reboot Pi
- [ ] Verify I2C: `sudo i2cdetect -y 1`

### 3. Calibration
- [ ] Calibrate pH sensor with standard solutions
- [ ] Adjust IR sensor potentiometers
- [ ] Test motor speeds and update `distance_per_second`

### 4. Testing
- [ ] Test each sensor individually
- [ ] Test motors (forward, backward, turns)
- [ ] Test cameras (check all 4 work)

### 5. Run!
- [ ] Start rover: `python3 rover_main.py`
- [ ] Monitor on dashboard: `streamlit run streamlit_dashboard.py`

---

## 📊 Data Files Generated

| File | Description |
|------|-------------|
| `rover_data_log.json` | Complete JSON log with all data |
| `rover_data.csv` | CSV export for easy analysis |
| `rover_config.json` | Configuration parameters |

---

## 🐛 Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| Motors not moving | Check battery voltage, verify common ground |
| DHT11 timeouts | Add pull-up resistor, try different GPIO pin |
| pH always same | Check I2C connection: `sudo i2cdetect -y 1` |
| Cameras not found | `ls /dev/video*` and add user to video group |
| Permission denied | `sudo usermod -a -G gpio,i2c $USER && sudo reboot` |

---

## 📱 Dashboard Features

Access at `http://localhost:8501`

### Main Views:
1. **Current Status**: Live sensor readings
2. **Path Map**: 2D trajectory with study sites  
3. **Environment**: Temp & humidity graphs
4. **Soil Analysis**: pH measurements over time
5. **Statistics**: Obstacle detection, actions taken
6. **Raw Data**: Table view with CSV export

### Controls:
- **Auto-refresh**: Updates every 2 seconds
- **Manual refresh**: Click "🔄 Refresh Now"
- **Upload data**: Load files from anywhere
- **Download CSV**: Export for further analysis

---

## 🎮 Rover Behavior

### Navigation Priority:
1. **IR Sensors** (highest priority - immediate danger)
2. **Camera Vision** (path planning)
3. **Auto-study** (periodic stops for measurements)

### Actions:
- `forward`: Path clear ahead
- `backward`: Reversing to find alternate route
- `turn_left/turn_right`: Avoiding obstacles
- `spin_left/spin_right`: Rotating to scan area
- `stop`: All paths blocked
- `STUDY_LOCATION`: Taking measurements

### Auto-study Mode:
- Stops every 30 seconds (configurable)
- Takes 3 pH readings and averages
- Records position, temp, humidity
- Logs to JSON and CSV

---

## 📝 Configuration Quick Edit

Edit `rover_config.json`:

```json
{
  "auto_study_enabled": true,   // Enable/disable auto-study
  "study_interval": 30,          // Seconds between studies (default: 30)
  "default_speed": 50,           // Motor speed 0-100 (default: 50)
  "ir_priority": true,           // IR overrides camera (recommended: true)
  "log_to_file": true           // Save data (recommended: true)
}
```

---

## 🔬 pH Sensor Calibration Quick Guide

```python
python3

>>> from sensor_manager import SensorManager
>>> import board
>>> 
>>> config = {
...     'dht_pin': board.D4,
...     'ir_pins': {'front': 17, 'back': 27, 'left': 22, 'right': 23},
...     'adc_channel': 0
... }
>>> 
>>> sensors = SensorManager(config)
>>> 
>>> # Place in pH 4 solution
>>> ph4 = sensors.read_ph_voltage()
>>> print(f"pH 4: {ph4}V")
>>> 
>>> # Rinse, place in pH 7
>>> ph7 = sensors.read_ph_voltage()
>>> print(f"pH 7: {ph7}V")
>>> 
>>> # Rinse, place in pH 10
>>> ph10 = sensors.read_ph_voltage()
>>> print(f"pH 10: {ph10}V")
>>> 
>>> # Save calibration
>>> sensors.calibrate_ph(ph4, ph7, ph10)
```

---

## 📞 Emergency Stops

### Stop Rover:
- Press `Ctrl+C` in terminal
- Automatically stops motors
- Saves all data
- Cleans up GPIO

### If Frozen:
```bash
# Kill Python process
pkill -f rover_main.py

# Reset GPIO
python3 -c "import RPi.GPIO as GPIO; GPIO.cleanup()"
```

---

## 📂 File Upload to Dashboard

Don't have access to Pi files? Upload directly:

1. Open dashboard: `streamlit run streamlit_dashboard.py`
2. Click "📁 Upload Data File" in sidebar
3. Select `rover_data_log.json` or `rover_data.csv`
4. Dashboard loads uploaded data

---

## 🎯 Typical Workflow

```bash
# Morning: Start rover
python3 rover_main.py

# (Rover explores and collects data for hours)

# Evening: Stop rover (Ctrl+C)

# Transfer data to computer
scp pi@raspberrypi.local:~/rover_project/rover_data*.* .

# Analyze on dashboard
streamlit run streamlit_dashboard.py
```

---

## 📊 Expected Data Collection Rate

- **Sensor readings**: Every 0.5-2 seconds
- **Position updates**: Every movement
- **Auto-studies**: Every 30 seconds
- **Typical run**: 100-500 data points per hour

---

## ⚠️ Important Safety Notes

1. **Never connect motor 5V to Pi** - will damage Pi!
2. **Always common ground** - Pi GND = Motor Driver GND
3. **Check battery voltage** - low voltage causes erratic behavior
4. **Store pH probe properly** - in storage solution, not water
5. **Test individually first** - before running complete system

---

## 🎓 Learning Resources

- **GPIO Pinout**: https://pinout.xyz
- **OpenCV Docs**: https://docs.opencv.org
- **Streamlit Docs**: https://docs.streamlit.io
- **Adafruit Guides**: https://learn.adafruit.com

---

## ✅ Pre-flight Checklist

Before running rover:

- [ ] Battery fully charged (>11V)
- [ ] All wires securely connected
- [ ] Common ground established
- [ ] I2C enabled and working
- [ ] All 4 cameras detected
- [ ] pH probe in storage solution
- [ ] Test run completed successfully
- [ ] Data directory exists
- [ ] Config file present

**You're ready to explore! 🚀**

---

*For detailed information, see README.md*
