# Omega2-Dash
Onion Omega2 Dash IoT Computer - a touchscreen device based on the Omega2

![front](./images/o2-dash-sample-0.jpg)
![back](./images/o2-dash-sample-1.jpg)

# Bring-Up Guide

Test firmware for the Omega2 Dash is available: [`omega2-dash-v0.3.2-b228.bin`](./bin/omega2-dash-v0.3.2-b228.bin)

Steps to flash your Omega2 Dash:

- Download the `omega2-dash-v0.3.2-b228.bin` firmware to the `/tmp` directory on your Omega2 Dash
- Use `sysupgrade` to install the firmware: `sysupgrade -n -v /tmp/omega2-dash-v0.3.2-b228.bin`
- Wait until it reboots, and then reboot again

This firmware has a kernel driver for the ILI9341 TFT driver and enables a framebuffer device, `/dev/fb0`.

## Backlight Control

Backlight is controlled by GPIO21 (PWM Channel 3)

- 100% duty cycle = min brightness
- 0% duty cycle = max brightness


```
omega2-ctrl gpiomux set uart2 pwm23
onion pwm 3 50 120

omega2-ctrl gpiomux set uart2 pwm23
onion pwm 3 0 120
```

## Touch Driver

Using XPT7603 controller, connected to Omega via I2C

- I2C Address: 0x48
- Interrupt pin: GPIO14

Shell commands to test touch input:

```
#read X
i2cget -y 0 0x48 0x88 w

#read Y
i2cget -y 0 0x48 0x98 w
```

![](./images/touch-i2cget-0.png)

## Controlling the Display

By default, the framebuffer is connected to the `/dev/tty1` virtual device.

Try echoing to this tty and you'll see the contents printed on the display!

```
echo "hello from the Omega2 Dash" > /dev/tty1
```

The **blinking cursor** on the display can be disabled:

```
echo 0 > /sys/class/graphics/fbcon/cursor_blink
```

To enable it again, just echo a `1` instead.

The **orientation** of the display can also be changed:

```
echo 1 > /sys/class/graphics/fbcon/rotate
```

Valid options are `0`, `1`, `2`, and `3`


# Demos

## Terminal Demo

Running the Omega's terminal on the display:

![terminal demo gif](./images/demo-terminal-0.gif)

- Run `sh > /dev/tty0` on your Omega2 Dash. Everything you type from then on will show up on the display and not in your terminal window
- Hit `ctrl+c` to exit

## LittleVGL UI Demo

Demo of a touch-enabled GUI made with LittleV Graphics Library:

![littlev demo gif](./images/demo-lvgl-0.gif)

Steps to run on your Omega2 Dash:

- Download [`bin/o2-dash-lvgl-demo` from this repo](./bin/o2-dash-lvgl-demo) to your Omega
- Make it executable: `chmod +x o2-dash-lvgl-demo`
- Run it: `./o2-dash-lvgl-demo`

## Displaying png & jpg Images

Display image files:

![onion logo](./images/demo-fbi-0.jpg)![map](./images/demo-fbi-1.jpg)

Steps to display images on your Omega2 Dash:
- First install the fbi package: `opkg update ; opkg install fbi`
  - This only needs to be done once
- [Transfer](http://docs.onion.io/omega2-docs/transferring-files.html) image files to the Omega. Supported file types are png, jpg
- Display the images by running `fbi -d /dev/fb0 -notty -nocomments --noverbose <PATH TO IMAGE FILE>`
 
For example: `fbi -d /dev/fb0 -notty -nocomments --noverbose /root/map.png`
 
The fbi program is really flexible, try running it with multiple files and see what happens!
