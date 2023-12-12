# Support ADS7846 on BTT CB1

## Context

I wanted to make [this screen](https://www.waveshare.com/wiki/5inch_HDMI_LCD) work for my 3d-printer.
I'm running the M8P with a CB1.
Unfortunately it is not compatible out of the box, I had to make a few modifications including writing a device tree overlay in order for the touchscreen to work.

The screen I bought uses [a few pins of the 40 pins header](https://www.waveshare.com/w/upload/1/19/5inch-HDMI-LCD-Manual-02-Pi-4B.jpg).
The pins that are used are the following:
* GPIO7
* GPIO9
* GPIO10
* GPIO11
* GPIO25

Most of those are for SPI communication.

Unfortunately the GPIO 25 (Which is equivalent to pin 22 on the 40 pin header) [is not connected on the CB1](https://bigtreetech.github.io/docs/CB1.html#40-pin-gpio), so we need to use a different port.
I used GPIO 24 instead, this is equivalent to the pin 18 or `PC9` from the CB1 perspective.

For the device tree overlay there are a few places I could draw inspiration from.

[This](https://github.com/armbian/sunxi-DT-overlays/blob/master/examples/spi-ads7846.dts) is an example of implementation for the sunxi platform (The CB1 uses Cortex A54).

[Here](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/Documentation/devicetree/bindings/input/touchscreen/ads7846.txt) is the instruction on how to implement the bindings from the linux kernel documentation.

And [Here](https://github.com/raspberrypi/linux/blob/3d9d7e7b2aa312f79f8034a9d42b7e7ccb92c54b/arch/arm/boot/dts/overlays/ads7846-overlay.dts#L4) is the overlay for the raspberrypi.

## How to make this work

Compile the device tree overlay and move it to the right place
```
dtc -@ -O dtb -o sun50i-h616-ads7846.dtbo sun50i-h616-ads7846.dts
cp sun50i-h616-ads7846.dtbo /boot/overlay-user/sun50i-h616-ads7846.dtbo
```

Add the following in /boot/BoardEnv.txt

```
user_overlays=ads7846
param_ads7846_speed=50000
param_ads7846_swapxy=0
param_ads7846_pmax=255
param_ads7846_xohms=150
param_ads7846_xmin=200
param_ads7846_xmax=3900
param_ads7846_ymin=200
param_ads7846_ymax=3900
```

Add the following in /etc/X11/xorg.conf.d/10-monitor.conf
```
Section "Monitor"
	Identifier "HDMI-1"
	HorizSync 28.0-80.0
	VertRefresh 48.0-75.0
	Modeline "800x480_60.00"   29.50  800 824 896 992  480 483 493 500 -hsync +vsync
	Option "PreferredMode" "800x480_60.00"
EndSection
```

Add the following in /etc/X11/xorg.conf.d/99-calibration.conf
```
Section "InputClass"
        Identifier      "calibration"
        MatchProduct    "ADS7846 Touchscreen"
        Option  "Calibration"   "140 3951 261 3998"
        Option  "SwapAxes"      "0"
EndSection
```

Reboot and enjoy :)
