---
title: "Exploring the Raspberry Pi Pico"
date: 2021-03-31T23:11:06+01:00
url: "/raspberry-pi-pico/01"
tags: ["raspberrypi", "raspberrypi-pico", "hardware", "code"]
image: "posts/exploring-the-pico/raspberry-pi-pico.png"
---

The [RaspberryPi Pico was released earlier this year](https://www.raspberrypi.org/blog/raspberry-pi-silicon-pico-now-on-sale/) and I haven't seen this much hype since the Arduino or the original RaspberryPi. It looked good, and it is very cheap ([you can get one for less than 4£](https://thepihut.com/products/raspberry-pi-pico)) so i got one to play around.

![](https://www.raspberrypi.org/documentation/rp2040/getting-started/static/64b50c4316a7aefef66290dcdecda8be/Pico-R3-SDK11-Pinout.svg)

### Why am i writing this ?

There are plenty of good tutorials, examples, and books for the RaspberryPi Pico, so anything i write about here is most likely available somewhere else. But for my own benefit I decided to explore the different key features of the Pico and document them in a series of posts.

### Getting started

#### Install and Connect

1. [Install the MicroPython UF2 file](https://www.raspberrypi.org/documentation/rp2040/getting-started/#getting-started-with-micropython)
This was very simple to do, download the pre-compiled file from the website and drop it to the mounted RPI-RP2 USB device
![Setup Animation](https://www.raspberrypi.org/documentation/rp2040/getting-started/static/4e43e3d335a3769aa85da9519b67af89/MicroPython-FINAL.gif)

*Image credit: [raspberry-pi website](https://www.raspberrypi.org/documentation/rp2040/getting-started/#getting-started-with-micropython)*

2. Connecting to the REPL

```bash
brew install minicom # needed for serial communications with the device
ls /dev/tty.usbmodem0000000000001 # confirm the device is available
minicom -b 115200 -o -D /dev/tty.usbmodem0000000000001
```

#### Gotchas

1. To mount the Pico in mass device mode, press and keep pressing the button for a bit after you connect the usb cable.
2. To get the REPL, do not press the button when connecting the device, it should appear as `/dev/tty.usbmodem0000000000001`

#### Hello World

```python
>>> help()

Welcome to MicroPython!

For online help please visit https://micropython.org/help/.

For access to the hardware use the 'machine' module.  RP2 specific commands
are in the 'rp2' module.
...

>>> print("Hello World")

Hello World
```

#### Blink

```python
from machine import Pin
import time

led = Pin(25, Pin.OUT) # Pin 25 is attached to the on device LED

for i in range(10):
    led.toggle()
    time.sleep(0.5)
```

#### Blink with Timer

This uses the [built in timer](https://datasheets.raspberrypi.org/rp2040/rp2040-datasheet.pdf) to control the blinking frequency

> The system timer peripheral on RP2040 provides a global microsecond timebase for the system, and generates
> interrupts based on this timebase. It supports the following features:
> A single 64-bit counter, incrementing once per microsecond.

```python
from machine import Pin, Timer

led = Pin(25, Pin.OUT) # Pin 25 is attached to the on device LED

def blink(timer):
    global led
    led.toggle()

timer = Timer()
timer.init(freq=0.5, mode=Timer.PERIODIC, callback=blink)
```

### Writing code in an Editor

The [official tutorial](https://projects.raspberrypi.org/en/projects/getting-started-with-the-pico/2) recommends using [Thonny](https://thonny.org/) but i prefer using vscode. To do that you need the [Pico-Go extension](https://marketplace.visualstudio.com/items?itemName=ChrisWood.pico-go)

### How do python files get uploaded ?

The official docs do not mention this explicitly, but looking around I found [Adafruit MicroPython tool](https://github.com/scientifichackers/ampy) which is used for uploading code to Adafruit MicroPython boards. Looking at the code gives a clue to how simple yet annoying it is.

```python
# https://github.com/scientifichackers/ampy/blob/master/ampy/files.py

# ...
# remotely execute open file
self._pyboard.exec_("f = open('{0}', 'wb')".format(filename))

# convert files to chunks then write them remotely
self._pyboard.exec_("f.write({0})".format(chunk))
```

This is not necessarily how it exactly works in Pico-Go or on the RaspberryPi Pico, but it gives me a mental model of how its working under the hood

### Conclusion

* Setting up the Pico with MicroPython is fairly simple.
* The timer on the Pico can be used to schedule fairly accurate ticks.
* REPL helps to play around quickly, but for anything practical we should use an Editor like [vscode + pico-go](https://marketplace.visualstudio.com/items?itemName=ChrisWood.pico-go) or [Thonny](https://thonny.org/)
* [The official python documentation is pretty good](https://datasheets.raspberrypi.org/pico/raspberry-pi-pico-python-sdk.pdf)

### What to explore next ?

* Play with Temperature sensor
* USB device emulation (Keyboard & Mouse)
* Check floating point acceleration
* Explore the low power modes
* Explore dual core (Threads)

[![discuss on twitter](https://img.shields.io/badge/-discuss-blue?style=flat-square&logo=Twitter&logoColor=white)](https://twitter.com/intent/tweet?text=@abdallahhodieb%20https://www.abdallahhodieb.com/raspberry-pi-pico/01)
