---
layout:     post 
title:      "Servo Driver on RasPi [Part 1 of 2]"
subtitle:   "Generating a PWM in User Space"
date:       2026-01-16
author:     "Josh Dean"
url:        "/2026/01/16/servo-driver-part-1/"
categories: ["projects"]
tags:       ["raspberry-pi", "linux", "i2c", "gpio", "pwm", "adc"]
description: Walkthrough for setting up a basic multi-threaded program on a RaspberryPi to control a servomotor via a potentiometer.
image:      "/img/rpi-servo-controller.png"
---

> "Tony Stark was able to build this in a cave! With a box of scraps!" - Obadiah Stane

## Intro
Oh the joys and wonders of having a box of scraps. During my trip home this Christmas, I discovered a particularly dusty [RaspberryPi 4](https://a.co/d/dswCaQ0) and accompanying electronic component [starter kit](https://a.co/d/0P2yWFF). I purchased this lot on a whim about 6 years ago now, before I ever considered pursuing a career in technology, and for awhile my fixation on software kept it housed in the dungeons of my closet. But now, fresh out of a microcontroller course in uni, I looked upon my old prisoner with new eyes. I thought it would be a fun exercise to try and recreate one of our labs where we used a microcontroller generated PWM wave to control a servomotor, only this time using the RaspPi and its GPIO pins. 

> **NOTE:** [Part 2](/2026/01/17/servo-driver-part-2/) actually explains setting up the kernel space device driver!

## Background
I'll just give a brief intro into each of these components, or at least what is minimally relevant to the project.

### - PWMs
PWM, or pulse-width modulation, waves are rectangular waves constantly flipping on/off. As a result, their average "on-time", or duty cycle, results in a percentage which is used to control light intensity, power consumption, etc.

![Duty Cycle Examples](/img/Duty_Cycle_Examples.png)

There are many ways to generate a PWM, but most the most common way is with a timer and a counter. First you fix the timer's frequency, then at the start of each period, turn the signal on and count clock ticks until you have reached the desired on-duration. Finally, turn the signal off for the remainder of the period and repeat.

### - Servomotors
Motors found in a multitude of different applications, these versatile components use a PWM signal within a certain operating frequency range to guide their output angle.

![Servo Timing Diagram](/img/servo-timing-diagram.png)

I used the very common TowerPro SG90 model, which uses a 50 Hz operating range to contol its angle.

### - Potentiometer & ADC
Variable resistors with a twisting knob, to control the flow of power between low and high. Perfect for steering our servomotor since we can scale the current voltage to a percentage of high (5V), and use that to guide our PWM generation. The ADC (analog digital converter) samples the current [analog] voltage and provides us a digital integer value to be used on the CPU.

## Materials
I was restricted to the kit's components, but here's what you'll need if you want to follow along:
- RasPi (any model with GPIO pins and sufficent memory)
- RasPi GPIO extension board
- Jumper wires
- LED & 220 Ω resistor
- 10K Ω Potentiometer (or similar)
- 8-bit ADS7830 (or similar)
- TowerPro SG90 Servomotor (or similar)

## Process

Going forward, all of the code can be found in the [project repository](https://github.com/joshdean814/rpi-servo-driver). The section names loosely correspond with the code files.

### - RasPi Setup
To start, I had to configure the RasPi. My mouse and keyboard always seem to mysteriously vanish, so I just set-up in headless mode [^1]. I just used the standard RasPi 64-bit OS. This took a few tries since I needed it to automatically connect to my home network to enable SSH-ing in from my Mac. I would recommend plugging into a monitor during boot-up, just to verify that it is connecting.

I prefer to work in VSCode when possible, and it is worth noting that by default, VSCode cannot connect to local network devices, so its SSH will fail while the standard terminal succeeds. To allow it, `System Settings > Privacy & Security > Local Network > Visual Studio Code > Toggle On`.

Lastly, just plug in the extension board ribbon and we are ready to roll.

### - Making a Basic PWM
To get a feel for the environment, I wanted to start by using the built-in PWM pin generation in the RasPi to see if if I could get it operational. In the GPIO pinout [^2], we can see the various PWM sites; I chose to use GPIO Pin #18 (PWM0).

![Basic LED Setup](/img/LED_setup.png)

To activate the wave, we must write to the sysfs (userspace interface) for the Linux PWM controller drivers. This can be done in 5 easy steps:

```bash
# Export (this will fail if already done).
echo 0 | sudo tee /sys/class/pwm/pwmchip0/export

# Disable before changing settings.
echo 0 | sudo tee /sys/class/pwm/pwmchip0/pwm0/enable

# Update period, duty cycle.
echo 2000000000 | sudo tee /sys/class/pwm/pwmchip0/pwm0/period
echo 1000000000 | sudo tee /sys/class/pwm/pwmchip0/pwm0/duty_cycle

# Re-enable the wave generation.
echo 1 | sudo tee /sys/class/pwm/pwmchip0/pwm0/enable
```

Within these settings, the speeds are specified in nanoseconds [^3]. So with setting the period to 2000000000 ns (2s), and the duty cycle to 1000000000 ns (1s), we should see the LED blink on for 1s and then off for 1s (50% duty cycle).

If we disconnect the LED and instead connect the servomotor to GPIO 18 and 5V:

![Servo Connection](/img/servo_connected.png)

Then we can change the period to 50 Hz:
```bash
echo 2000000000 | sudo tee /sys/class/pwm/pwmchip0/pwm0/period
```

and then see the motor move:
```bash
# Approx 0 degrees
echo 1000000 | sudo tee /sys/class/pwm/pwmchip0/pwm0/duty_cycle
# Approx 90 degrees
echo 2000000 | sudo tee /sys/class/pwm/pwmchip0/pwm0/duty_cycle
```

## - Adding the Potentiometer/ADC
The wiring here gets a bit messy, but plug the ADC in and connect the `3.3V` (Pin 1) to its `VCC`, `GND` to `GND`, the `SDA` (Pin 3) to the ADC's `SDA`, and the `SCL` (Pin 5) to the ADC's `SCL` pin. Then add the potentiometer with the `3.3V` also in its top pin, and `GND` in its bottom pin. Insert a wire in the middle pin and run it back to the ADC's `A0` pin. Here is a visual:

![ADC/Potent](/img/potent_adc.png)

With this setup, the current voltage reading from the potentiometer gets passed to the ADC's `A0` sample channel. Using I2C serial protocol, we will request the ADC to make readings at certain intervals and report back to us an 8-bit integer [0 - 255].

In the user space of Linux, the general gist of using I2C is first opening a file descriptor to `/dev/i2c-1`, which is the interface with the 1st I2C bus on the device. From here we can use the `ioctl` command with a constant value `I2C_SLAVE` from `#include <linux/i2c-dev.h>` to focus our request on a specific address on the device.

```C
// Direct towards 'TARGET_ADDR'. 
if (ioctl(fd, I2C_SLAVE, TARGET_ADDR) == 0)
{
    // Command the device...
}
```

I was not sure which address the ADC lived on, and a common method to check which devices are actually accessible[^4] is using `sudo i2cdetect -y 1`, which scans the listeners in the range `0x8` to `0x77`. It reported a single listener at `0x4b`, the home of my ADC.

The next steps of I2C are to use a `write`/`read` pair to the device, where we send a single command byte in the `write` and then read back one byte in the `read` (as it is an 8-bit ADC). Optimally, this "read" byte will contain the current voltage reading from the component.

As for the command byte, in page 13 & 14 of the ADS7830 datasheet [^5], we can see the table for configuring the command byte.

![Datasheet](/img/servo-driver/datasheet.png)

We want to get the result in single endian (SD = 1), and  `CH0` as our +IN, which yields `1000` for the first four bits, then we want "*Internal Reference ON and A/D Converter ON*", or 11XX for the second four, for a final result of `0b10001100` (`0x8C`).

At this point I was honestly feeling a little rusty about my C, so I decided to create a simple I2C address scanning program to mimic the function of `i2cdetect`, which can be found in the repo under `potentio-adc/i2c_scan.c`. This is definitely riskier on other devices actively using the I2C bus, but nothing else is running on my RasPi, so I simply walked the `[0x8, 0x77]` address space and sent a dummy byte (`0x0`) to see who was listening.

Now properly warmed up, I set to getting some initial readings from the ADC. By writing our control bit with:
`write(fd, &ctrl, 1)`, we can then wait for the response byte with `read(fd, &curr_meter, 1)`. We can see all of these parts put together in `potentio-adc/ads7830_read.c`, or build the Makefile and run the `ads7830_read` executable to get a reading every second and verify the setup is working.

### - Mapping ADC to Servo
Now that we had the current voltage reading cast to a char [0-255], the only real step left to do is to scale the value into a delay time (in ms). The operating range for the servomotor is [1000, 2000] (microsec), so we need to linearly map the ADC output into this range:

```C
int curr_meter = ads7830_read_ch0(ads_fd);
int updated_ms = 1000 + (curr_meter * 1000) / 255;
```

So with a 50 Hz (20 ms) period, the on-signal will last for 1-2 ms and control our motor.

Because we were no longer using the PWM chip on the RasPi, our wave will now be driven by software rather than the hardware clock. There will of course be some latency for this, but at 50 Hz it should not be too noticeable. To control the GPIO pin directly, we have to access `/dev/gpiochip0` instead of `/sys/class/pwm/pwmchip0/`.

After connecting a file descriptor here, we use a `gpiohandle_request` struct from `#include <linux/gpio.h>`:

```C
// Get the GPIO handler for requested pin.
struct gpiohandle_request req;
memset(&req, 0, sizeof(req));

// Init the handler.
req.lineoffsets[0] = GPIO_PIN_18;
req.flags = GPIOHANDLE_REQUEST_OUTPUT;
req.default_values[0] = 0;
req.lines = 1;

// Send request.
if (ioctl(gpio_chip_fd, GPIO_GET_LINEHANDLE_IOCTL, &req) < 0) { } // Handle error.

close(gpio_chip_fd);
return req.fd; // Communicate with gpio fd going forwards.
```

We must specify the pin be used as an output, as well as a few other default settings. If successful, the new fd should be set in the request struct. We can now pass a `gpiohandle_data` struct with our desired signal (0/1):

```C
    struct gpiohandle_data data;
    memset(&data, 0, sizeof(data));

    data.values[0] = 1; // Turn the signal ON.

    // Try to to write the new value to the pin.
    if (ioctl(gpio_fd, GPIOHANDLE_SET_LINE_VALUES_IOCTL, &data) < 0) { } // Handle error.
```

That seemed far too easy for me, so I decided to add one final wrinkle: placing the servo controller on its own thread loop, and the ADC reading on another. The servo loop would operate at the required 50 Hz frequency, and the ADC would go a little slower at 20 Hz.

I am also fresh out of a real time systems class, so this was a good exercise for relative delay using the built-in `CLOCK_MONOTONIC` high resolution clock. In short, we get the current time, compute the offset into the future for our period, and sleep until its reached. To ensure no data races occured, I wrapped the current `target_us` (microseconds) in an atomic for each load/store. After loading a fresh reading, I clamped it into our range with:

```C
// Clamp the values to [1000, 2000].
if (pulse_us < 1000) { pulse_us = 1000; }
if (pulse_us > 2000) { pulse_us = 2000; }
```

To test it out, run the Makefile in `adc_servo_ctrl/`. If all is wired correctly, the potentiometer should control the servometer clockwise and counterclockwise from 0° -> 90°.

For me at least, there was a little bit of start-up and shutdown jitter. I was able to mitigate these by instead controlling the PWMs at the kernel level, which you can read all about in [part 2](/2026/01/17/servo-driver-part-2/)!

## References
[^1]: https://www.tomshardware.com/reviews/raspberry-pi-headless-setup-how-to,6028.html
[^2]: https://pinout.xyz/
[^3]: https://www.acmesystems.it/pwm
[^4]: https://emlogic.no/2025/06/accessing-i2c-devices-from-userspace-in-linux/
[^5]: http://ti.com/lit/ds/symlink/ads7830.pdf?ts=1768593291843&ref_url=https%253A%252F%252Fwww.ti.com%252Fproduct%252FADS7830