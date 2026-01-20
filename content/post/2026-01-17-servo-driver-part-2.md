---
layout:     post 
draft:      true
title:      "Servo Driver on RasPi [Part 2 of 2]"
subtitle:   "Generating a PWM in User Space"
date:       2026-01-17
author:     "Josh Dean"
url:        "/categories/projects/servo-driver-part-2/"
categories: ["projects"]
tags:       ["raspberry-pi", "linux", "i2c", "gpio", "pwm", "adc"]
description: Walkthrough for setting up a basic multi-threaded program on a RaspberryPi to control a servomotor via a potentiometer.
image:      "/img/servo-driver/kernel_module_ai.png"
---


## Background

### - Kernel Modules
Loadable kernel modules (LKMs) are bits of code that can be inserted and removed into the Linux kernel, without requiring a rebuild of the entire OS. For something like a device driver, these are extremely helpful as they can allow hardware control and a custom interface with user-space.