---
title: RustDesk Setup on Jetson Orin Nano
date: 2025-04-22
draft: false
tags: 
description:
---
Created: 2025-04-22
## 1. Install RustDesk
1. Install dependencies: `sudo apt install -y libxdo3`
2. Download RustDesk: `wget https://github.com/rustdesk/rustdesk/releases/download/1.3.9/rustdesk-1.3.9-aarch64.deb -O /tmp/rustdesk.deb`
3. Install rustdesk from the deb file: `sudo dpkg -i /tmp/rustdesk.deb`
## 2. Setup a dummy monitor for headless display
* Run administrative privvyledge interactively: `sudo -s`
```bash
cd /etc/X11 && cp xorg.conf xorg.conf.orig && \
echo 'Section "Device"
            Identifier "DummyDevice"
            Driver "dummy"
            VideoRam 256000
        EndSection
         
        Section "Screen"
            Identifier "DummyScreen"
            Device "DummyDevice"
            Monitor "DummyMonitor"
            DefaultDepth 24
            SubSection "Display"
                Depth 24
                Modes "1920x1080_60.0"
            EndSubSection
        EndSection
         
        Section "Monitor"
            Identifier "DummyMonitor"
            HorizSync 30-70
            VertRefresh 50-75
            ModeLine "1920x1080" 148.50 1920 2448 2492 2640 1080 1084 1089 1125 +Hsync +Vsync
EndSection' >> /etc/X11/xorg.conf.d/10-dummy.conf
```
### Notes:
* Change the resolution and framerate as desired, below sets up a 1920x1080p display at 60Hz
* Replace '/etc/X11/xorg.conf.d/10-dummy.conf' with '/etc/X11/xorg.conf' if only a dummy monitor will be used.
## 2. Allow RustDesk with headless support
1. Install Prerequisites: `sudo apt install ubuntu-desktop xserver-xorg-video-dummy lightdm -y` -> Choose 'LightDM' when prompted. 
2. `sudo rustdesk --option allow-linux-headless Y`
## 3. Enable RustDesk Startup on boot
1. `sudo systemctl enable rustdesk`
## Setup Adaptive Display
1. In the rustdesk window, Click the monitor icon -> Toggle "Scale Adaptive"

## Tips
* Get rustdesk ID on the command line: `rustdesk --get-id`
* Setting a password on the command line: `sudo rustdesk --password <password>`
* Allow unattended/full access from the command line: `sudo rustdesk --option access-mode=full`