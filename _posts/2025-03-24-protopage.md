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

# Problem 2(filling RAM vs. sampling rate):

My goal is to record high frequency vibrations with an IMU. Human's are capable of detecting very fast frequencies of force. 

To determine the sampling frequency of the DAQ, I need to multiply the largest vibration frequency(Nyquist, in this case) by 2. Typically, roboticist in haptics use 100 Hz as a common vibration frequency. So my goal, should be to measure at 200+Hz. To compare this to experts in the field, I noticed that Cass Labs samples at 400Hz. 

The problem that I ran into is that my configurations of the DAQ and its code are not fast enough to capture the desired vibration frequencies.

It is known, that the biggest ways to slow down a program are to submit data to the console. 

## Method 1 (writing directly to the CSV file

In my first version of the code, I write the IMU data directly to the csv file immediately after the data is recorded. This involves opening up the csv and writing to the file, which is a costly time process.

Sampling rate 17Hz.

## Method 2 (append data to a Python list, and save the data in a CSV once the program was over)

In my second version of the code, I append the IMU data to a list and then wait until the end of the recording session. This was faster, however I kept running into a MemoryError. Without, any delay the program kept crashing. I added a small delay. The fastest that I could sample the data was about 45Hz, without immediately getting a memory error, but after about 10 seconds the program crashed with a MemoryError. Online resources helped me deterine that the RAM was overfilling as the data_list grew larger with data. I tried to implement a garbage collector to clear more cache memory to be used, and I saw minimal improvement, i.e. I gained a little bit more bytes. I am still far from my goal of recording a 2min sampling run.

Fastest sampling rate at 45Hz.

## Further Research
1. Using a programming language(C, as it is precompiled) that is faster than MicroPython. I found other projects online that use MicroPython at a very fast sampling rate(500Hz)
2. Upgrading the microcontroller. I have found more projects that use the RPi Pico at 500kHz, but they say this is only possible when you is C and DMA(direct memory access)

# Resources:
1. i2c explanation: https://learn.adafruit.com/working-with-i2c-devices
2. i2c library used: https://github.com/Lezgend/MPU6050-MicroPython
3. Cass Labs: https://casslabs.xyz/pages/research.html
