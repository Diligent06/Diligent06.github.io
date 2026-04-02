# Hardware & Embedded Devices

## I2C Device Detection

```bash
sudo apt install i2c-tools
ls /dev/i2c*                     # List I2C buses
sudo i2cdetect -y -r 1           # Scan bus 1 (-y: skip confirmation, -r: read mode)
```

### Enable CAN Bus

```bash
sudo modprobe can
sudo modprobe can_raw
sudo modprobe mttcan
sudo ip link set can0 up type can bitrate 1000000
```

### TJA1051T CAN Converter

> ⚠️ **Important:** You must supply **5V** to VCC — do not use 3.3V.

---


