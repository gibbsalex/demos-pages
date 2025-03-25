---
title: "Buiding a DAQ: sampling multiple IMU's"
data: 2025-02-24
---
# Project: 
Record IMU data of the rear suspension of a mountain bike

# Task: 
Build a Data Aquisition(DAQ) system to record from two points on the bicycle frame. Point A = a fixed frame point, Point B = a moving link(wheel).

# Setup:
- Raspberry Pi Pico (RP2040)
- 2x IMU (MPU = 6050)
- On/off Light (Blue LED, switch, 2x resistor)
- Power Bank
- MicroPython

# i2c speed run

Structure of a transaction:
Start | Sensor Address + R/W | Acknowledge(ACK) | DATA | ACK | ... (repeat) ... | Stop 

The microcontroller must know which sensor to communicate with(address), and the action it needs to take(R/W), and then the data that it is sending.

SDA: Serial Data Signal
SCL: Serial Clock Signal
^ sensors that share these pins are on the same "i2c bus"

The Start is identified at the instant the SDA pin transitions from HIGH to LOW, when the SCL is HIGH.
The Stop is identified at the instant the SDA pin transitions from LOW to HIGH, when the SCL is HIGH.

### The "Gimmick"
Things we need to know:
1. We need to identify all of the address for the sensors that we would like to talk to
2. Wire all SDA and SCL pins to the same bus

Things that the i2c library takes care of:
1. Low-level structure of the transaction
2. Choosing which registers to communicate with on each board

# Problem 1(multiple sensors on multiple channels):
With the first setup, I was seeing data get mixed between IMU sensors. As an example, Sensor 2 would read data from Sensor 3.

The first problem was that I was trying to use 2 i2c channels at the same time. I found multiple sources online that suggested that I stick to one channel at a time. I decided to rewired the project to reflect the simplification.

The second problem with my setup was that I was using multiple sensors with the same i2c ID. When I bought the sensors, I assumed they would work together because my first prototype worked. I didn't consider that multiple sensors of the same type would have the same i2c ID. I decided not to use an i2c multiplexer, and to stick with 2 sensors that had different i2c ID's.

# Problem 2(filling memory vs. sampling rate):

My goal is to record high frequency vibrations with an IMU. 

# Resources:
1. i2c explanation: https://learn.adafruit.com/working-with-i2c-devices
2. i2c library used: https://github.com/Lezgend/MPU6050-MicroPython
