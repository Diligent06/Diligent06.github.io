# Hardware & Embedded Devices

## AGX (NVIDIA Jetson AGX)

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

## Agilex Robot

```bash
ssh agilex@192.168.1.6
# Password: agx123456789
```

---

## Intel RealSense Camera

```bash
# 1. Install librealsense from official guide:
#    https://dev.realsenseai.com/docs/compiling-librealsense-for-linux-ubuntu-guide

# 2. Install Python wrapper in a conda environment
pip install pyrealsense2

# 3. Run official Python examples:
#    https://github.com/IntelRealSense/librealsense/tree/master/wrappers/python/examples
```

---

## NFS Shared Folders (Xiaojian's Server)

```bash
sudo mkdir -p /mnt/xiaojian/bigai_ml
sudo mount -t nfs 10.1.110.37:/srv/nfs/shared/bigai_ml /mnt/xiaojian/bigai_ml

sudo mkdir -p /mnt/xiaojian/nfs_fillipo
sudo mount -t nfs 10.1.110.37:/srv/nfs/shared/nfs_fillipo /mnt/xiaojian/nfs_fillipo
```
