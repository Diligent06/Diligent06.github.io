# Hardware & Embedded Devices

## I2C Device Detection

```bash
sudo apt install i2c-tools
ls /dev/i2c*                     # List I2C buses
sudo i2cdetect -y -r 1           # Scan bus 1 (-y: skip confirmation, -r: read mode)
```

## Enable CAN Bus

```bash
sudo modprobe can
sudo modprobe can_raw
sudo modprobe mttcan
sudo ip link set can0 up type can bitrate 1000000
```

## TJA1051T CAN Converter

> ⚠️ **Important:** You must supply **5V** to VCC — do not use 3.3V.

---

## Realsense T265 VIO device usage guidance

Realsense T265 VIO device is supported by Realsense SDK <= 2.50.0, so you need to install SDK earlier than 2.50.0. 

```bash
sudo apt install -y \
    build-essential \
    pkg-config \
    libusb-1.0-0-dev \
    libssl-dev \
    libglfw3-dev \
    libgl1-mesa-dev \
    libglu1-mesa-dev \
    python3-dev
# finish neccessary package installation

mkdir build && cd build

cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DBUILD-PYTHON_BINDINGS=ON \
    -DBUILD_EXAMPLES=OFF \
    -DBUILD_GRAPHICAL_EXAMPLES=OFF \
    -DPython3_EXECUTABLE=$(which python)

make -j2 && sudo make install
# finish realsense SDK installation

export PYTHONPATH=your_librealsense_path/build/wrappers/python:$PYTHONPATH
# export python binding library into your python path and you can use pyrealsense2 build from source by yourself
```


