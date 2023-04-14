# GC9A01-for-RPi4
Some notes on how to get a GC9A01 round display working on Raspberry Pi 4B

Using the waveshare example libraries in python and c, we got the SPI / spidev stuff to work following these instructions: https://www.waveshare.com/1.28inch-LCD-Module.htm 
https://www.waveshare.com/wiki/1.28inch_LCD_Module

However, since we have a multiscreen setup build with QtQuick, we wanted the screen to show our Qt prototypes straight up. They couldn't easily write to the screen. A lot of trial and error followed, where we tried screenshotting the protoype and dumping it into the framebuffer using NumPy. Some of the things we tried involved fbtft, fbcp, mmap. So to save someone else that headache, here is what worked in the end for us. 

1. We used the built in GC9A01 overlay (on top of / instead of the spi), which creates a /dev/fb1 framebuffer for the screen.
2. We had our chatbot of choice create an xrandr conf file that set /dev/fb1 up as a screen
3. That's it.

Now, I initially wanted a multihead boot (HDMI and the SPI screens both working on boot), but couldn't get it going. I tried following this:
https://forums.raspberrypi.com/viewtopic.php?t=306624

Accidentally during that work, the small screen came alive showing the desktop (the corner of it, anyway). The HDMI was not working. Since that is actually good for our purposes I didn't pursue a multihead xrandr config further, but should be doable.

Attached is our /boot/config.txt and the xrandr config file. 

*1:*
Add this to the end of the config file (I don't think the spi part is necessary, as it is overwritten by the latter gc9a01 overlay). Also, I'm not sure the HDMI stuff is ever applied. 

[all]
dtoverlay=spi-bcm2835-overlay
enable_uart=1
dtoverlay=gc9a01, width=240,height=240,fps=50

hdmi_force_hotplug=1 
hdmi_cvt=300 300 60 1 0 0 0 
hdmi_group=2 
hdmi_mode=1 
hdmi_mode=87 
display_rotate = 1 

## Framebuffer color depth must match for X11 to run without error
framebuffer_depth=16
gpu_mem=256

*2.*
Copy the 99-fbdev.conf file into /etc/X11/xorg.conf.d/ Reboot and it will start with desktop on the little one (only). Have ssh setup before you do this so you can reach in and delete it without having to work the terminal through a round 240X240 screen - that sucks. You can do it on the tiny screen, but yeah, it's tiny so make sure to know which folders you need to go to beforehand.

*3.* We couldn't start our Qt/PySide2 prototype over ssh (it complains there is no screen), but it works locally or by creating it as a service on the RPi and start it through that.
