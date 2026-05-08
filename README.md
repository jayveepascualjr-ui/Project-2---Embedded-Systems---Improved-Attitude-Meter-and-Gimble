# Project 2 - Improved 2-Axis Attitude Meter and Gimbal

//Submitted by: Vergel pascual jr.
//ID: D22124439
//Course: TU839/4
//Subject: Embedded Systems - EENG1030
//Last Edited: 08/05/26

## Overview

This project is an **improved version** of my first project. While the first one only handled one side, this one is a **2-axis gimbal system** that tracks both **Roll and Pitch**. I also added a safety alarm using the **SAI (Serial Audio Interface)** which makes a beeping sound if the gimbal tilts too far.

## Improvements from Project 1

* **Dual-Axis utilization**: The project now uses both X and Y data from the accelerometer to control two servos at the same time.
* **SAI Audio Alarm**: A new feature where the stm32 plays a sine wave tone through a speaker. This uses DMA so the sound doesn't lag the rest of the code.
* **Better Telemetry**: The LCD and Serial monitor now show real degrees for both Roll and Pitch instead of just raw numbers.

## Features and Peripherals used

* **i2c communication**: used to get digital tilt data for Roll and Pitch from the sensor.
* **Timers**: Timer 2 is used for the two servos, and Timer 1 is used for the LED brightness.
* **SAI & DMA**: Used to produce the alarm sound. DMA is great because it handles the audio samples in the background.
* **Interrupts**: A button on PB0 is used to pause the whole system if things get out of hand.
* **LCD & Serial**: Provides a live feed of what the gimbal is doing in degrees.

## Hardware pinout

| peripheral | pin | function |
| --- | --- | --- |
| **Roll Servo** | pa0 | pwm output for left/right tilt |
| **Pitch Servo** | pa3 | pwm output for forward/back tilt |
| **Status LED** | pb1 | brightness shows how far from center |
| **Pause Button** | pb0 | interrupt button to stop/start |
| **Audio Clock** | pa8 | sai clock for the speaker |
| **Audio FS** | pa9 | sai frame sync |
| **Audio Data** | pa10 | sai data out |
| **Debug TX** | pa2 | sends roll/pitch data to pc |

### lcd screen

| lcd pin | pin | function |
| :--- | :--- | :--- |
| **sck** | pa5 | spi clock |
| **mosi** | pa7 | spi data |
| **dc** | pb7 | command select |
| **reset** | pb6 | screen reset |
| **cs** | pa12 | chip select |

## technical implementation

### 1. 2-Axis Program

The system reads raw 16-bit values for both X and Y from the accelerometer. Just like before, I used a deadzone of +/- 1500 because the sensor is very twitchy and surfaces aren't perfectly flat. If the tilt goes past this deadzone, the code adds or subtracts 3.0 to the servo pulse width. This moves the gimbal smoothly to stay level.

### 2. SAI Audio Alarm

This is the main improvement. I used a sine wave table with 180 samples. If the Roll or Pitch angle goes below 30 degrees or above 150 degrees, the SAI block is enabled and the DMA starts pumping the sound to the speaker. I found these angle limits through testing to be the "hazard zone."

### 3. LED and PWM

* **Servos (Timer 2)**: Set to 50Hz (20ms period). I clamped the values between 500us and 2500us so the motors don't grind their gears at the edges.
* **LED (Timer 1)**: This is a visual feedback. I calculated the "difference" from the 1500 center point. The more you tilt, the higher the PWM duty cycle gets, making the LED brighter.

### 4. Interrupt Handling

I moved the button to PB0. It uses a falling edge trigger with an internal pull-up. When pressed, it flips the `is_paused` variable. When paused, the servos stay still and the audio is forced off for safety.

## Debugging and Testing

* **Serial Monitor**: I used `printf` to show Roll and Pitch degrees on the screen. I added a counter to only print every few loops so the text doesn't fly past too fast to read.
* **Hardware Test**: I used the "beeping" code first to make sure my SAI pins (PA8-PA10) were wired right before adding the gimbal code.
* **Oscilloscope**: I used an oscilloscope to check if functions such as LED and Servos are running properly by checking their PWM
* **Servo Tuning**: I used trial and error to find the exact 90-degree center point, which ended up being exactly 1500us.

## how to use

1. Connect the Roll and Pitch servos to PA0 and PA3.
2. Hook up the speaker to the SAI pins.
3. Build and upload using PlatformIO in VS Code.
4. Open the serial monitor to see the angles. If you tilt it too far, it will start beeping until you level it back out.

## references
* **eeng1030 repository (frank duignan)**: base libraries and esample codes [https://github.com/f3dtud/EENG1030](https://github.com/f3dtud/EENG1030)
    * **bmi160_l432**: used as a reference for the i2c protocol implementation and internal register mapping for the bmi160 accelerometer.
    * **analoginpwmout**: used as reference for timer configurations, specifically for controlling LED and servo through PWM
    * **stm32l432_lcd**: provided reference code on how LCD screen worked and files for the display
* **f3dtud/EENG1030**: Used the SAI audio test and the BMI160 library as a base for my code.
* **stm32l432_lcd**: Reference for how to clear lines and print text on the display.
