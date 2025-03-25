---
title: "IMU DAQ"
data: 2025-02-24
---

STAR:

Project: Record IMU data of the rear suspension of a mountain bike

Task: Build a Data Aquisition(DAQ) system to record from two points on the bicycle frame. Point A = a fixed frame point, Point B = a moving link(wheel).

Setup:
- Raspberry Pi Pico (RP2040)
- 2x IMU (MPU = 6050)
- On/off Light (Blue LED, switch, 2x resistor)
- Power Bank
- MicroPython

Problem 1:
With the first setup, I was seeing data get mixed between IMU sensors
