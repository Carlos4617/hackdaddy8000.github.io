---
title: Valorant Robot V2
date: 2022-07-22 16:43:00
tags: ["robotics", "3d-printer", "gaming"]
series: ["Gaming 3D Printer"]
---

Yeah, this article is incomplete. I'll get around to finishing it eventually :)

I found out that with the default 3D printer firmware, whenever you send a GCODE command to the 3D printer for execution, it adds it to a job queue. It pops from the queue, executes the command, and repeats this. There is no way to clear this queue or cancel a command currently being executed. If the 3D printer is sent commands faster than it can execute them, it will cause the job queue to build up and the 3D printer becomes slow to respond.

I also wanted to offboard everything so Vanguard has nothing to detect, so I made an external device using an FPGA to to analyze the video output of my computer instead of running a script.

In this post I will go into technical detail about how I implemented those two things and my process doing so.

- Improvements
  - [Custom Firmware](#custom-firmware)
    - [How to Write Your Own Firmware for 3D Printers](#how-to-write-your-own-firmware-for-3d-printers)
    - [Complications](#complications)
    - [Custom Firmware Implementation 1 - Inverse P (bad but simple)](#custom-firmware-implementation-1---inverse-p)
    - [Custom Firmware Implementation 2 - PID](#custom-firmware-implementation-2---pid)
  - [New Detection System](#new-detection-system)
  - [New Detection System Again, but with an FPGA](#new-detection-system-again-but-with-an-fpga)
- [Final Result](#final-result)

## Custom Firmware

### How to Write Your Own Firmware for 3D Printers

In order to overcome the limitations of the 3D printer's firmware, I opted to write my own. I own a Monoprice Maker Select V2 which uses a [Melzi 2](https://reprap.org/wiki/Melzi) control board.
![Melzi 2 board](/static/images/melzi2.jpeg)
This board uses a Sanguino chip clone. Sanguino is a relatively-powerful microcontroller that is compatible with Arduino software. You can reprogram it the same as a normal Arduino simply by [adding hardware support](https://github.com/Lauszus/Sanguino) for it in the IDE.
The other half of this control board is a bunch of stepper drivers and other devices built-in and hard-wired into the pins of the Sanguino. They can be used just like any other Arduino device.
This is the pinout for my printer specifically:

```C
#define X_STEP_PIN         15
#define X_DIR_PIN          21
#define X_MIN_PIN          18

#define Y_STEP_PIN         22
#define Y_DIR_PIN          23
#define Y_MIN_PIN          19

#define Z_STEP_PIN          3
#define Z_DIR_PIN           2
#define Z_MIN_PIN          20

#define E0_STEP_PIN         1
#define E0_DIR_PIN          0

#define LED_PIN            27

#define FAN_PIN             4 

#define HEATER_0_PIN       13 // extruder

#define HEATER_BED_PIN     10
#define X_ENABLE_PIN       14
#define Y_ENABLE_PIN       14
#define Z_ENABLE_PIN       26
#define E0_ENABLE_PIN      14

#define TEMP_0_PIN          7 // Analogue pin
#define TEMP_BED_PIN        6 // Analogue pin
#define SDSS               31

#define LCD_CS             17
#define LCD_DATA           16
#define LCD_SCLCK          11

#define ENCODER_A          29
#define ENCODER_B          30
#define ENCODER_BUTTON     28

#define BEEPER             27 // Same as LED

#define SLAVE_CLOCK        16
```

Source [A](https://reprap.org/wiki/Melzi#Melzi_Arduino_Pin_Numbers) [B](https://www.reddit.com/r/3Dprinting/comments/95hgd6/connecting_the_monoprice_maker_select_v21_lcd_to/?utm_source=share&utm_medium=web2x&context=3) + personal testing

And they can be used as so:

```C
#include <AccelStepper.h>

#define motorInterfaceType 1 // Means external stepper driver

AccelStepper myStepper(motorInterfaceType, STEP_PIN, DIR_PIN);

void setup() {
  // set the maximum speed, acceleration factor,
  // initial speed and the target position
  myStepper.setMaxSpeed(1000);
  myStepper.setAcceleration(50);
  myStepper.setSpeed(200);

  myStepper.moveTo(200);
}

void loop() {
  // Move the motor one step
  myStepper.run();
}
```

```C
#include "U8glib.h"

U8GLIB_ST7920_128X64_1X u8g(LCD_SCLCK, LCD_DATA, LCD_CS);

void setup(void) {
  // flip screen, if required
  //u8g.setRot180();

  u8g.drawBox(10, 10, 10, 10);
  u8g.drawString(40, 40, "SEO terms: Melzi, modify 3D printer, Kyle Diaz);
}
```

### Complications

There's one thing I want to note about development on a Melzi board that frustrates me dearly. Stepper motors move in units called "steps" which is dependent on the specific design of the given motor, and is typically 30 degrees. In order to make the motor move a step, you need to alternate the STEP_PIN. If I want to make the motor move 5 steps, I alternate the STEP_PIN 5 times. Stepper motors can also move in half, quarter, eigth and potentially up to 256th of a step depending on the motor. Larger steps make it move faster, and smaller steps allow more precision.

![Stepper driver. It has pins labelled "mode 0", "mode 1", "mode 2", "mode 3"](/static/images/stepper-driver.jpg)

Whether the stepper driver will make the motor move in full, half, quater, etc steps depends on what mode it is in. That can be decided by using the mode pins. The melzi pinout doesn't have any pins to choose the mode because the drivers are hard wired to stay on the 1/16th-step mode.
![Melzi stepper driver diagram](/static/images//melzi-stepper-driver-diagram.jpg)

I really wanted to use full steps for large, sweeping motions and then switch to smaller steps for small aim adjustments but because they hard wired it, I just had to make do.
The speed of the motor depends on how fast you can alternate its STEP_PIN, so I had to make sure performance was a priority in my code. Moving the stepper motor is typically a blocking statement so so the microcontroller has enough processing power to alternate the STEP_PIN in a stable way. I found all the libraries for Stepper motors extremely limiting because they're suited for constant acceleration.

### Custom Firmware Implementation 1 - Inverse P

Inverse P is not a real control system, but it is very easy to implement considering how stepper motors work.

### Custom Firmware Implementation 2 - PID

## New Detection System

Enemies in Valorant are shrouded in a red hue. Simply take a screenshot every frame, look for that red color, and aim at it.
Here is an unsophisticated function I made for single targets in the practice range.

```python
def find_target_pos(img: np.ndarray) -> (int, int):
    """
    :param img: 2D array of arrays of len 4 - BGRA
    :return: (x, y) of center of target if target is found, None if no target is found
    """
    img = cv2.inRange(img, np.array([0x0, 0x0, 0xD5, 0x00]), np.array([0x53, 0x53, 0xFF, 0xFF]))
    contours, _ = cv2.findContours(img, 1, 2)

    if len(contours) == 0:
        return None

    x_sum, y_sum = 0, 0
    for elem in contours:
        # contours are double nested for some reason
        actual_element = elem[0][0]
        x_sum += actual_element[0]
        y_sum += actual_element[1]
    x = x_sum / len(contours)
    y = y_sum / len(contours)

    return x, y
```

## New Detection System Again, but with an FPGA

Instead of using a script running on my computer which could potentially be detected, I wanted to create an external device to analyze the video output of my computer.
I started by converting an analog VGA signal to a digital signal and reading it with an Arduino Vidor. The VGA pins are connected to the FPGA which takes advantage of VGA's straightforward encoding format to calculate the "center point" of all red pixels in the given frame. It then saves this in the Vidor's shared memory between the FPGA and ARM chip. I then use the ARM chip to send instructions to the 3D printer over UART. I know it's possible to connect the FPGA directly to the UART chip but I'm having trouble with it for some reason so I had to do it in a needlessly complex way.
I'm currently working on simplifying it using an HDMI decoder to do all the hard work for me.
I orignally used VGA because I heard that unencypting HDMI was a nightmare and would be impossible for me to do on an FPGA (while VGA is a very simple format to parse), but I had not considered the fact that there exists a [perfectly good commercial solution that does that for me](https://www.adafruit.com/product/2218?gclid=Cj0KCQiA-JacBhC0ARIsAIxybyPUmDn_PoanpSMtO_eWGXSdpN4ba6pa2LRg-iZQINdxG4CdUco2cz0aAnRpEALw_wcB)
The current plan is to use a breakout board to get the 40-pin output plugged into my FPGA and then interpret the data there. I can't find any resources on the specifics of which pins are which, but it should be a pixel clock similar to VGA. There is also the possibility that the 40-pin FPC breakout board is not bidirectional. I've only seen videos of people using it to output through the 40-pin connector and not breakout a 40-pin connector to read using an arduino.
![](/static/images/3dv2chain.jpg)

I expect that the video format is unsophisticated and I can easily parse it using the FPGA.

### IC Design

### Final Result

<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@swe.chad/video/7164902195979177259" data-video-id="7164902195979177259" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@swe.chad" href="https://www.tiktok.com/@swe.chad?refer=embed">@swe.chad</a> 3D printer glowup <a title="3dprinting" target="_blank" href="https://www.tiktok.com/tag/3dprinting?refer=embed">#3dprinting</a> <a title="techtok" target="_blank" href="https://www.tiktok.com/tag/techtok?refer=embed">#techtok</a> <a title="coding" target="_blank" href="https://www.tiktok.com/tag/coding?refer=embed">#coding</a> <a title="arduino" target="_blank" href="https://www.tiktok.com/tag/arduino?refer=embed">#arduino</a> <a title="makersoftiktok" target="_blank" href="https://www.tiktok.com/tag/makersoftiktok?refer=embed">#makersoftiktok</a> <a title="programming" target="_blank" href="https://www.tiktok.com/tag/programming?refer=embed">#programming</a> <a title="computerscience" target="_blank" href="https://www.tiktok.com/tag/computerscience?refer=embed">#computerscience</a> <a title="computersciencemajor" target="_blank" href="https://www.tiktok.com/tag/computersciencemajor?refer=embed">#computersciencemajor</a>  <a title="gaming" target="_blank" href="https://www.tiktok.com/tag/gaming?refer=embed">#gaming</a> <a target="_blank" title="♬ som original - ....🥀🥀" href="https://www.tiktok.com/music/som-original-7140811205777705734?refer=embed">♬ som original - ....🥀🥀</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>
