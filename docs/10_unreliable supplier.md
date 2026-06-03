# This document contains a list of unreliable suppliers and the reasons why they were identified as such


## 深圳市仁恒影像科技有限公司 (淘宝店铺名称：摄像头生产厂家)
该相机供应商存在以下问题：
1. 相机驱动软件粗糙 （lsusb无法显示相机信息）
2. 相机协议实现有问题，无论何种分辨率，何种相机格式，该相机都只能按照30Hz的频率输出，无法修改
3. 相机带宽申请有问题，无论何种分辨率，何种相机格式，该相机均选择最高的带宽保留等级alt=11 （maxpackload=3060, Ivl=125us）
4. 该供应商无法提供相应软件支持，并且自认为相机驱动没有问题，不承认摆在面前的问题，无视顾客
具体的一些debug information在下面展示：
```bash
(base) root@hzhy:~/workspace/dohc# v4l2-ctl --list-devices
PC Camera: PC Camera (usb-fc800000.usb-1.2):
        /dev/video0
        /dev/video1
        /dev/media0

PC Camera: PC Camera (usb-fc800000.usb-1.4):
        /dev/video2
        /dev/video3
        /dev/media1

(base) root@hzhy:~/workspace/dohc# lsusb -t
/:  Bus 08.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 5000M
/:  Bus 07.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 480M
/:  Bus 06.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 5000M
/:  Bus 05.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 480M
/:  Bus 04.Port 1: Dev 1, Class=root_hub, Driver=ohci-platform/1p, 12M
/:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=ohci-platform/1p, 12M
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ehci-platform/1p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-platform/1p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M
        |__ Port 2: Dev 3, If 1, Class=Video, Driver=uvcvideo, 480M
        |__ Port 2: Dev 3, If 0, Class=Video, Driver=uvcvideo, 480M
        |__ Port 4: Dev 4, If 0, Class=Video, Driver=uvcvideo, 480M
        |__ Port 4: Dev 4, If 1, Class=Video, Driver=uvcvideo, 480M

打开2个终端，分别运行下面两个命令
v4l2-ctl -d /dev/video0 \
--set-fmt-video=width=640,height=480,pixelformat=MJPG \
--stream-mmap --stream-count=10000

v4l2-ctl -d /dev/video2 \
--set-fmt-video=width=640,height=480,pixelformat=MJPG \
--stream-mmap --stream-count=10000

运行第一个可以正常以30hz输出图像，运行第二个会报错 
(base) root@hzhy:~/workspace/dohc# v4l2-ctl -d /dev/video0 \
> --set-fmt-video=width=640,height=480,pixelformat=MJPG \
> --stream-mmap --stream-count=10000
                VIDIOC_STREAMON returned -1 (No space left on device)
同时dmesg中显示
[  199.382845] uvcvideo: Failed to submit URB 0 (-28).

(dohc) root@hzhy:~/workspace/dohc# v4l2-ctl -d /dev/video0 --all
Driver Info:
        Driver name      : uvcvideo
        Card type        : PC Camera: PC Camera
        Bus info         : usb-fc800000.usb-1.2
        Driver version   : 5.10.226
        Capabilities     : 0x84a00001
                Video Capture
                Metadata Capture
                Streaming
                Extended Pix Format
                Device Capabilities
        Device Caps      : 0x04200001
                Video Capture
                Streaming
                Extended Pix Format
Media Driver Info:
        Driver name      : uvcvideo
        Model            : PC Camera: PC Camera
        Serial           : 01.00.00
        Bus info         : usb-fc800000.usb-1.2
        Media version    : 5.10.226
        Hardware revision: 0x00000725 (1829)
        Driver version   : 5.10.226
Interface Info:
        ID               : 0x03000002
        Type             : V4L Video
Entity Info:
        ID               : 0x00000001 (1)
        Name             : PC Camera: PC Camera
        Function         : V4L2 I/O
        Flags         : default
        Pad 0x01000007   : 0: Sink
          Link 0x02000013: from remote pad 0x100000a of entity 'Extension 4': Data, Enabled, Immutable
Priority: 2
Video input : 0 (Input 1: ok)
Format Video Capture:
        Width/Height      : 640/480
        Pixel Format      : 'MJPG' (Motion-JPEG)
        Field             : None
        Bytes per Line    : 0
        Size Image        : 614400
        Colorspace        : sRGB
        Transfer Function : Rec. 709
        YCbCr/HSV Encoding: ITU-R 601
        Quantization      : Default (maps to Full Range)
        Flags             : 
Crop Capability Video Capture:
        Bounds      : Left 0, Top 0, Width 640, Height 480
        Default     : Left 0, Top 0, Width 640, Height 480
        Pixel Aspect: 1/1
Selection Video Capture: crop_default, Left 0, Top 0, Width 640, Height 480, Flags: 
Selection Video Capture: crop_bounds, Left 0, Top 0, Width 640, Height 480, Flags: 
Streaming Parameters Video Capture:
        Capabilities     : timeperframe
        Frames per second: 30.000 (30/1)
        Read buffers     : 0
                     brightness 0x00980900 (int)    : min=-64 max=64 step=1 default=0 value=0
                       contrast 0x00980901 (int)    : min=0 max=95 step=1 default=2 value=2
                     saturation 0x00980902 (int)    : min=0 max=100 step=1 default=64 value=64
                            hue 0x00980903 (int)    : min=-2000 max=2000 step=1 default=0 value=0
 white_balance_temperature_auto 0x0098090c (bool)   : default=1 value=1
                          gamma 0x00980910 (int)    : min=100 max=300 step=1 default=100 value=100
           power_line_frequency 0x00980918 (menu)   : min=0 max=2 default=2 value=2
                                0: Disabled
                                1: 50 Hz
                                2: 60 Hz
      white_balance_temperature 0x0098091a (int)    : min=2800 max=6500 step=1 default=4600 value=4600 flags=inactive
                      sharpness 0x0098091b (int)    : min=1 max=7 step=1 default=1 value=1
         backlight_compensation 0x0098091c (int)    : min=0 max=1 step=1 default=0 value=0
                  exposure_auto 0x009a0901 (menu)   : min=0 max=3 default=3 value=3
                                1: Manual Mode
                                3: Aperture Priority Mode
              exposure_absolute 0x009a0902 (int)    : min=3 max=2047 step=1 default=166 value=166 flags=inactive
                 focus_absolute 0x009a090a (int)    : min=0 max=1023 step=1 default=0 value=280 flags=inactive
                     focus_auto 0x009a090c (bool)   : default=0 value=1
```