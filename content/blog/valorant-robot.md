---
title: Making My 3D Printer Play Valorant
date: 2022-07-22 16:43:00
tags: ["robotics", "3d-printer", "gaming"]
series: ["Gaming 3D Printer"]
featured: false
---

Yeah, this article is incomplete. I'll get around to finishing it eventually :)

## Goal

My friends often call me a "bot" at Valorant because I'm dogwater at the game. The joke was that I would make an actual "bot" that plays better than them.

## Initial Design (Bad)

### TLDR

I could tell the 3D printer to move to where it sees the red hue of the enemy by sending it GCODE over USB. This did not work because the default 3D printer firmware is not meant to play fast, reactive games like Valorant.

### Controlling the 3D Printer

Based on the knowledge I initially had on how 3D printers worked, I thought this would be a relatively straightforward project. 3D Printers run gcode files
which resembles the same logic as python turtle program. When you slice a 3D object file using a program like Cura or Fusion 360, it creates a file containing instructions on how the head of the 3D printer should move.

For example, if you wanted to make the 3D printer move in a square, you would run this gcode on it -

```gcode
G91 1 0
G91 0 1
G91 -1 0
G91 0 -1
```

where G91 means "move X, Y from the current position" and the second and third parameter are the change in X and Y the head of the 3D printer will undergo.
This code tells the 3D printer to move one unit up, one unit to the right, one unit down, and finally one unit to the left - making a complete square motion.

These gcode commands can be run either by loading a gcode file on an SD card and running the file on the 3D printer directly, or gcode instructions can be
enqueued by sending them through a USB serial bus connected from an external device to the 3D printer. I could easily make a python program that sends GCODE to my 3D printer over USB.

```python
import serial

ser = serial.Serial('/dev/ttyUSB0', 115200)

ser.write("G91 1 0")

ser.close()
```

### Detection System

Enemies in Valorant are shrouded in a red hue. Simply take a screenshot every frame, look for that red color, and aim at it.
Here is an unsophisticated function I made for single targets.

```python
def find_target_pos(img: np.ndarray) -> (int, int):
    """
    :param img: 2D array of arrays of len 4 - BGRA
    :return: (x, y) of center of target if target is found, None if no target is found
    """
    # The 2nd and 3rd args are two different shades of red. Upper and lower bounds of the red hue of enemies.
    filteredImg = cv2.inRange(img, np.array([0x0, 0x0, 0xD5, 0x00]), np.array([0x53, 0x53, 0xFF, 0xFF]))
    contours, _ = cv2.findContours(filteredImg, 1, 2)

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

### How to Aim

We can use find_target_pos() to get the location of enemies relative to the crosshair. This relative pos will
tell us how we need to aim.

ie: If the relative position is (100, 0), we need to aim up. If the relative position is (-45, 60), we need to aim 45 pixels down and 60 pixels to the right.

### Mapping Character Movement Space to 3D Printer/Linear Movement Space

### Why this didn't work

The CV function and mapping character movement to mouse movement was fine. The real problem was working with the 3D printer to create reactive, fluid movements.

I found out that with the default 3D printer firmware, whenever you send a GCODE command to the 3D printer for execution, it adds it to a job queue. It pops from the queue, executes the command, and repeats this. There is no way to clear this queue or cancel a command currently being executed. If the 3D printer is sent commands faster than it can execute them, it will cause the job queue to build up and the 3D printer becomes slow to respond.

Additionally, how the 3D printer executes those commands was bad. The 3D printer will always decellerate to 0m/s upon completion of a move command. This would cause it to move very roughly because it was frequently stopping and starting.

## New Plan

Clearly, the firmware of the 3D printer was what was holding me back. So let's just get rid of it 🤷.

### How to Write Your Own Firmware for 3D Printers

In order to overcome the limitations of the 3D printer's firmware, I opted to write my own. I own a Monoprice Maker Select V2 which uses a [Melzi 2](https://reprap.org/wiki/Melzi) control board.
![Melzi 2 board](/images/melzi2.jpeg)
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

#### Writing Firmware to control servos

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

#### Writing Firmware to Display Elements on LCD

```C
#include "U8glib.h"

U8GLIB_ST7920_128X64_1X u8g(LCD_SCLCK, LCD_DATA, LCD_CS);

void setup(void) {
  // flip screen, if required
  //u8g.setRot180();

  u8g.drawBox(10, 10, 10, 10);
  u8g.drawString(40, 40, "SEO terms: Melzi, modify 3D printer, hackdaddy8000);
}
```

#### Complications

There's one thing I want to note about development on a Melzi board that frustrates me dearly. Stepper motors move in units called "steps" which is dependent on the specific design of the given motor, and is typically 30 degrees. In order to make the motor move a step, you need to alternate the STEP_PIN. If I want to make the motor move 5 steps, I alternate the STEP_PIN 5 times. Stepper motors can also move in half, quarter, eigth and potentially up to 256th of a step depending on the motor. Larger steps make it move faster, and smaller steps allow more precision.

![Stepper driver. It has pins labelled "mode 0", "mode 1", "mode 2", "mode 3"](/images/stepper-driver.jpg)

Whether the stepper driver will make the motor move in full, half, quater, etc steps depends on what mode it is in. That can be decided by using the mode pins. The melzi pinout doesn't have any pins to choose the mode because the drivers are hard wired to stay on the 1/16th-step mode.
![Melzi stepper driver diagram](/images//melzi-stepper-driver-diagram.jpg)

I really wanted to use full steps for large, sweeping motions and then switch to smaller steps for small aim adjustments but because they hard wired it, I was eternally stuck with 1/16th steps.
The speed of the motor depends on how fast you can alternate its STEP_PIN, so I had to make sure performance was a priority in my code. Moving the stepper motor is typically a blocking statement so so the microcontroller has enough processing power to alternate the STEP_PIN in a stable way. I found all the libraries for Stepper motors extremely limiting because they're suited for constant acceleration, and were tailored towards 3D-printer/CNC operations.

I just opted to manually control the servos by manually toggling the pinouts.

#### Custom Firmware Implementation 1 - Inverse P

Inverse P is not a real control system, but it is very easy to implement considering how stepper motors work. As discussed before, the speed of the servo is dependent on how frequently you can flip its pin.

This would theoretically be the fastest you could move the stepper motor assuming the motor could keep up:

```C++
while (true) {
  digitalWrite(X_STEP_PIN, HIGH);
  digitalWrite(X_STEP_PIN, LOW);
}
```

You could slow down the speed by throwing a few sleep functions in there, but in my implementation, that loop handles more logic than handling the servos. It has to handle receiving commands from my computer, too. That loop should be running as fast as possible so it can reliably get that data.

Instead of slowing down the loop, we can simply decrease the frequency at which the pins are flipped.

```C++
int freq = 2;
int i = 0;
while (true) {
  otherLogic();
  if (i % freq == 0) {
    flipPin(X_STEP_PIN);
  }
  i++;
}
```

This worked fine because in reality, the servos could not handle trying to flip the servos as fast as possible on my processor. In practice, freq would be 500 or more.

#### Custom Firmware Implementation 2 - PID

## New FPGA Design - Fricking Poggers Giga Awesome

Instead of using a script running on my computer which could potentially be detected, I wanted to create an external device to analyze the video output of my computer.

I started by converting an analog VGA signal to a digital signal and reading it with an Arduino Vidor. The VGA pins are connected to the FPGA which takes advantage of VGA's straightforward encoding format to calculate the "center point" of all red pixels in the given frame. It then saves this in the Vidor's shared memory between the FPGA and ARM chip. I then use the ARM chip to send instructions to the 3D printer over UART. I know it's possible to connect the FPGA directly to the UART chip but I'm having trouble with it for some reason so I had to do it in a needlessly complex way.

I'm currently working on simplifying it using an HDMI decoder to do all the hard work for me.

I orignally used VGA because I heard that unencypting HDMI was a nightmare and would be impossible for me to do on an FPGA (while VGA is a very simple format to parse), but I had not considered the fact that there exists a [perfectly good commercial solution that does that for me](https://www.adafruit.com/product/2218?gclid=Cj0KCQiA-JacBhC0ARIsAIxybyPUmDn_PoanpSMtO_eWGXSdpN4ba6pa2LRg-iZQINdxG4CdUco2cz0aAnRpEALw_wcB)

The current plan is to use a breakout board to get the 40-pin output plugged into my FPGA and then interpret the data there. I can't find any resources on the specifics of which pins are which, but it should be a pixel clock similar to VGA. There is also the possibility that the 40-pin FPC breakout board is not bidirectional. I've only seen videos of people using it to output through the 40-pin connector and not breakout a 40-pin connector to read using an arduino.

![](/images/3dv2chain.jpg)

I expect that the video format is unsophisticated and I can easily parse it using the FPGA.

### IC Design

## Final Result

<blockquote class="tiktok-embed" cite="https://www.tiktok.com/@hackdaddy8000/video/7164902195979177259" data-video-id="7164902195979177259" style="max-width: 605px;min-width: 325px;" > <section> <a target="_blank" title="@hackdaddy8000" href="https://www.tiktok.com/@hackdaddy8000?refer=embed">@hackdaddy8000</a> 3D printer glowup <a title="3dprinting" target="_blank" href="https://www.tiktok.com/tag/3dprinting?refer=embed">#3dprinting</a> <a title="techtok" target="_blank" href="https://www.tiktok.com/tag/techtok?refer=embed">#techtok</a> <a title="coding" target="_blank" href="https://www.tiktok.com/tag/coding?refer=embed">#coding</a> <a title="arduino" target="_blank" href="https://www.tiktok.com/tag/arduino?refer=embed">#arduino</a> <a title="makersoftiktok" target="_blank" href="https://www.tiktok.com/tag/makersoftiktok?refer=embed">#makersoftiktok</a> <a title="programming" target="_blank" href="https://www.tiktok.com/tag/programming?refer=embed">#programming</a> <a title="computerscience" target="_blank" href="https://www.tiktok.com/tag/computerscience?refer=embed">#computerscience</a> <a title="computersciencemajor" target="_blank" href="https://www.tiktok.com/tag/computersciencemajor?refer=embed">#computersciencemajor</a>  <a title="gaming" target="_blank" href="https://www.tiktok.com/tag/gaming?refer=embed">#gaming</a> <a target="_blank" title="♬ som original - ....🥀🥀" href="https://www.tiktok.com/music/som-original-7140811205777705734?refer=embed">♬ som original - ....🥀🥀</a> </section> </blockquote> <script async src="https://www.tiktok.com/embed.js"></script>
