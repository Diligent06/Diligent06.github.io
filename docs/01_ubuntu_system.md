# Ubuntu System Administration

## User Management

```bash
# Add a new user (run as root)
useradd your_user_name
usermod -aG sudo your_user_name   # Grant sudo privileges

# Recover lost sudo permission via liveUSB
sudo lsblk -f
sudo mount /dev/root /mnt
sudo chroot /mnt /bin/bash
passwd <username>   # Reset password
```

## Kernel Management

```bash
# List installed kernel images
dpkg --list | grep linux-image

# List bootable kernels
grep -A100 "menuentry " /boot/grub/grub.cfg | grep menuentry

# Switch default kernel
sudo nano /etc/default/grub
# Change: GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 6.8.1-1026-realtime"
sudo update-grub
sudo reboot
```

## Real-Time Kernel (Ubuntu Pro)

```bash
# Register Ubuntu Pro
sudo pro attach C129jKUHfzZGJ1Dk1DHuW9K8Stg7vf
pro status --all

# Reference: https://ubuntu.com/blog/enable-real-time-ubuntu
```

## Process Management

```bash
# Kill Python processes
kill -9 $(pgrep python)                         # Kill all python processes
kill -9 $(pgrep python | sed -n '2p')           # Kill second python process
pkill -9 python
ps aux | grep python | awk '{print $2}' | xargs kill -9

# Check background jobs and kill by job number
jobs
kill %1

# Run script in background (survives terminal close)
nohup python3 -u my_script.py > output.log 2>&1 &   # -u: unbuffered output
```

## File System & Storage

```bash
# Extend disk space in VirtualBox VM
VBoxManage modifyhd "vdi_path" --resize 50000   # 50000 MB

# Inside the VM
lsblk                          # Find root device (e.g. /dev/sda2)
sudo growpart /dev/sda 2
sudo resize2fs /dev/sda2
df -Th /                       # Verify new size
```

## SSH & Remote File System

```bash
# Mount remote directory via SSHFS
sshfs -p 6666 -o uid=$(id -u),gid=$(id -g),allow_other diligent@10.1.119.63:/mnt/fillipo /mnt/fillipo
sshfs -o uid=$(id -u),gid=$(id -g),allow_other yue@10.1.110.33:/home/yue/anaconda3/envs/bigai-eai /home/diligent/Desktop/yue/bigai_eai_yue
```

## GitHub SSH with Multiple Keys

```bash
# ~/.ssh/config
Host yangw9505
  HostName github.com
  User git
  IdentityFile ~/.ssh/wangw9505_github

# Set remote URL using the Host alias
git remote set-url origin yangw9505:user/repo.git
```

## Qt5.15 on Ubuntu 20.04

```bash
# Install from PPA: https://launchpad.net/~beineri/+archive/ubuntu/opt-qt-5.15.0-focal

# Fix OpenGL / software rendering issues
export LIBGL_DRIVERS_PATH=/usr/lib/x86_64-linux-gnu/dri
export LIBGL_ALWAYS_SOFTWARE=1
```

## GNU Library Override

```bash
export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6
export LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH
```

## I2C Device Detection

```bash
sudo apt install i2c-tools
ls /dev/i2c*                     # List I2C buses
sudo i2cdetect -y -r 1           # Scan bus 1 (-y: skip confirmation, -r: read mode)
```
