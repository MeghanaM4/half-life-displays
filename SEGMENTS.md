# 7-segment displays

7-segment displays are big, bright and readable from across the room. Use them for numbers: clocks, timers, scores and sensor readings.

## What it is

A 7-segment display is seven LED bars arranged in a figure 8, plus a little decimal point (DP). Light the right bars and you get any digit from 0 to 9, and a few rough letters too. The bars are always named **a** to **g**, starting at the top and going clockwise, with **g** in the middle.

![A red single-digit 7-segment display showing the number 4 on a breadboard](/art/starters/displays/segments-1.jpg)
*Photo: RetroEditor, CC BY-SA 4.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:LED_7_Segment_Display.jpg)*

They come as single digits or in packs of 2, 4 or 8. Clock-style 4-digit displays add a colon. The most common digit height is 0.56 in (about 14 mm).

### 14-segment alphanumeric displays

For real letters, a 14-segment display adds diagonal and split bars, so it can spell A-Z. That's a lot of pins, so use a driver chip like the HT16K33. Adafruit's quad alphanumeric backpack is the easy option.

![A 4-digit 14-segment LED display, model HDSP-A42C, with a colon between the middle digits](/art/starters/displays/segments-4.jpg)
*Photo: Hejaled, CC0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:14segment_LED_Display.jpg)*

## How it works

Every segment is an LED. A red LED drops about 2 V (its forward voltage), while blue and white ones drop about 3 V. Like any LED, **always put a resistor in series with every segment**, or you'll cook it (or your microcontroller pin).

![A 7-segment display cut in half, showing the tiny LED chips at the bottom of white plastic light pipes](/art/starters/displays/segments-2.jpg)
*Photo: TubeTimeUS, CC BY-SA 4.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:7-Segment_LED_Display_Cross_Section.jpg)*

In that cross-section, a tiny LED chip sits under each segment, and a white plastic light pipe spreads its glow into a bar.

### Common cathode vs common anode

To save pins, one side of all the LEDs in a digit is joined together:

- **Common cathode (CC):** all the minus ends are joined. Connect the common pin to GND and drive a segment pin HIGH to light it.
- **Common anode (CA):** all the plus ends are joined. Connect the common pin to VCC and drive a segment pin LOW to light it.

Your part's datasheet says which type it is, and so does the part number (CC56-12 is common cathode, CA56-12 is common anode). Pick the matching symbol in your schematic and the matching type in your code. Code set for the wrong type shows "8888" instead of a number.

### Multiplexing

A single digit needs 8 segment pins + 1 common pin. A 4-digit display would need 36 pins if wired separately. Instead, all four digits share the same 8 segment lines, and each digit gets its own common pin, for 12 pins in total.

The microcontroller lights one digit at a time, hundreds of times per second, and your eyes see all four at once. This is multiplexing.

Something has to keep refreshing the whole time. If your code pauses on a `delay(1000)`, only one digit stays lit. Driver chips do the refreshing for you.

### Three ways to drive it

1. **Bare display + direct pins.** Wire all 12 pins to your board. Good for learning how it works, though that's more pins than a XIAO has.
2. **Bare display + shift register.** A 74HC595 turns 3 pins into 8 outputs. You use fewer pins and still do the multiplexing in code.
3. **Driver module.** A chip like the TM1637, MAX7219 or HT16K33 does the multiplexing, brightness and current limiting for you. You just send it numbers over 2-3 wires.

**For your first project, design around a TM1637 module.** It needs 4 wires, costs little and works on 3.3 V and 5 V boards. You can move to a bare display later.

![Three 74HC595 shift register chips soldered on a green circuit board](/art/starters/displays/segments-3.jpg)
*Photo: Towel401 at English Wikipedia, public domain, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:74HC595.jpg)*

## Using it in a project

### Parts for your design

These are the parts that can go in your schematic and BOM. They're bought with your funding once your design is approved.

| Part | What it's for | Typical driver chip/module | Notes |
| --- | --- | --- | --- |
| Single-digit 0.56 in display (e.g. 5011BS) | Learning, 1-digit counters | None (direct pins) | 10 pins: 8 segments + 2 joined common pins |
| 4-digit 0.56 in bare display (e.g. Kingbright CC56-12 / CA56-12) | Custom PCBs | Direct pins + transistors, or TM1637 | 12 pins. KiCad has symbols and footprints for these |
| TM1637 4-digit module | Beginner pick: clocks, timers, counters | TM1637 (2-wire) | CLK, DIO, VCC, GND. Usually has a colon |
| MAX7219 8-digit module | Long numbers, 8 digits | MAX7219 (SPI-like, 3 wires) | DIN, CLK, CS. A 5 V chip |
| Adafruit 0.56 in 7-segment backpack | Neat I2C display | HT16K33 (I2C) | Up to 8 on one bus |
| Adafruit quad alphanumeric backpack | Words and letters | HT16K33 (I2C) | 14-segment, 4 characters |
| 74HC595 shift register | Saving pins with bare displays | – | 16 pins. Max 70 mA through the whole chip |

### On a PCB

This section puts the display on your own KiCad board with a XIAO or Pico.

**1. Pick your approach.**

- **Easiest:** put a TM1637 module on your PCB with a 4-pin header. You only design the connector.
- **Middle:** a bare 4-digit display with transistors, driven straight from your microcontroller.
- **Advanced:** a bare display plus your own TM1637 chip. It only uses 2 pins and needs no segment resistors.

**2. Add symbols** (in the Schematic Editor, press **A**):

- Microcontroller: `MCU_Module:RaspberryPi_Pico` is built in. For a XIAO, add Seeed's [OPL KiCad library](https://github.com/Seeed-Studio/OPL_Kicad_Library) through Preferences > Manage Symbol Libraries.
- Display: `Display_Character:CC56-12EWA` (common cathode) or `Display_Character:CA56-12EWA` (common anode). Single 0.39 in digits: `SC39-11EWA` / `SA39-11EWA`.
- Module header: `Connector_Generic:Conn_01x04`.
- Resistors and capacitors: `Device:R` and `Device:C`.
- Transistors: `Transistor_BJT:BC817` (NPN) or `Transistor_BJT:BC807` (PNP). For through-hole, use `2N3904` / `2N3906`.
- Shift register: `74xx:74HC595`. MAX7219: `Driver_LED:MAX7219`.

**3. Wire the schematic.** Press **P** for power symbols (+3V3, +5V, GND) and **L** for net labels, so you don't have to draw long wires everywhere.

For the module (easiest) approach, connect these pins in your schematic:

| Module pin | Seeed XIAO RP2040 | Raspberry Pi Pico |
| --- | --- | --- |
| VCC | 3V3 | 3V3(OUT), pin 36 |
| GND | GND | GND |
| CLK | D6 (GP0) | GP0 |
| DIO | D7 (GP1) | GP1 |

- **On 3.3 V boards, power the module from 3V3, not 5 V.** Many modules have 10 kΩ pull-up resistors from CLK and DIO to VCC. At 5 V, those pull-ups push 5 V into pins that only handle 3.3 V.
- The TM1637 datasheet lists 5 V as typical, but most modules work at 3.3 V (the micropython-tm1637 wiring uses 3V3). The datasheet says blue displays need 5 V, so a blue one may look dim at 3.3 V.
- A 100 µF capacitor across VCC and GND helps avoid glitches during fast updates.

If you're using a different module:

- **MAX7219:** DIN, CLK and CS go to 3 GPIOs. It's a 5 V chip, so with a Pico or XIAO add a level shifter (like a 74AHCT125) to be safe.
- **HT16K33 backpack:** I2C, so SDA/SCL go to D4/D5 on a XIAO or GP4/GP5 on a Pico. Power it from 3V3 or 5V to match your board. Pull-ups are built in.
- **74HC595:** `DS` (pin 14), `SH_CP` (pin 11) and `ST_CP` (pin 12) go to 3 GPIOs. Tie MR (pin 10) to VCC and OE (pin 13) to GND. Q0-Q7 drive segments a-DP through 8 resistors, and [SevSegShift](https://github.com/bridystone/SevSegShift) uses the same commands as SevSeg. Keep segments under about 8 mA each to stay inside the chip's 70 mA total.

For the transistor (middle) approach with a common cathode display:

- Each segment pin (a-g, DP) goes to a GPIO through its own resistor: 220 Ω at 3.3 V, 330 Ω at 5 V. **Never leave out the resistors.** A missing one can burn out a digit for good. At 3.3 V, only red and green displays light well.
- Putting the resistors on the segments instead of the digit pins keeps the brightness even. With resistors on the digit pins, an "8" looks dimmer than a "1", since 7 segments share one resistor.
- Each digit pin goes to the collector of an NPN transistor. Emitter to GND. Base to a GPIO through a 1 kΩ resistor.
- The transistors are there because when a digit shows "8.", all 8 segments flow through that one digit pin. That's up to 8 × 5 mA = 40 mA, which is too much for one GPIO pin.
- In SevSeg, use `N_TRANSISTORS` with `resistorsOnSegments = true`. With a common anode display, use PNP transistors from 3V3 to each digit and `P_TRANSISTORS`.

For the TM1637 (advanced) approach, note that it only drives common anode displays:

- GRID1-GRID4 go to the digit (anode) pins. SEG1-SEG8 go to segments a-DP.
- Add 10 kΩ pull-ups from CLK and DIO to VDD. Bare chips won't talk without them.
- Optional: 100 pF from CLK and DIO to GND reduces noise, like the datasheet circuit.
- Add a 100 nF capacitor right next to VDD, plus a 10 µF bulk capacitor. The datasheet says to put the filter cap "as close to TM1637 chip as possible".
- TM1637 isn't in KiCad's built-in library. It's a 20-pin SOP-20 chip, so make a simple symbol and match a `Package_SO` footprint to the datasheet's package drawing.

Every chip (74HC595, MAX7219, TM1637) gets its own 100 nF decoupling capacitor between VCC and GND.

**4. Assign footprints** (Tools > Assign Footprints):

- Display: `Display_7Segment:CA56-12EWA` (it fits the CC56-12 too).
- Resistors: `Resistor_SMD:R_0805_2012Metric`.
- Transistors: `Package_TO_SOT_SMD:SOT-23`.
- Module header: `Connector_PinHeader_2.54mm:PinHeader_1x04_P2.54mm_Vertical`.
- For a display on a separate board, use a JST PH connector like `Connector_JST:JST_PH_B4B-PH-K_1x04_P2.00mm_Vertical`.

**5. Lay out and route.** Click "Update PCB from Schematic" (F8).

- Put the display on the front and keep the transistors and resistors close behind it. The microcontroller can sit beside it or on the back.
- Draw your board outline on the Edge.Cuts layer.
- Route with **X**. Use 0.25 mm traces for signals and 0.5 mm for 3V3, 5 V and GND. The whole display draws under 100 mA, so you don't need anything wider.
- Add a GND fill on both layers (Place > Add Filled Zone, then press **B** to refill).
- Avoid 90° corners and keep traces short. Rip up and re-route as often as you need to.

**6. Check before you submit.**

- Run Inspect > Design Rules Checker and fix every error (warnings are usually fine).
- **Check pin 1 on the display.** Display pinouts vary by brand, so compare the footprint with your exact part's datasheet.
- Open the 3D viewer (Alt+3) to make sure the display faces the right way and nothing sits under it.
- Add M3 mounting holes if it's going in a case. These displays barely get warm.

## Adding inputs

Inputs let people change the number on the display.

- **Buttons (tactile or low-profile switches):** `Switch:SW_Push`. One side to a GPIO, the other to GND. Use `INPUT_PULLUP` in code, so pressed reads LOW. For lots of buttons, use a key matrix with a 1N4148 diode per key. The TM1637 can also scan up to 16 keys on its K1 and K2 pins.
- **EC11 rotary encoder:** `Device:RotaryEncoder_Switch`, footprint `Rotary_Encoder:RotaryEncoder_Alps_EC11E-Switch_Vertical_H20mm`. A and B go to GPIOs, C goes to GND. The push switch goes to a GPIO and GND.
- **Slide potentiometer (e.g. Bourns PTA series):** one end to 3V3, the other to GND, the middle (wiper) to an analog pin. On the XIAO RP2040, D0-D3 are analog pins.
- **I2C sensor (like the AHT20 temperature sensor):** SDA and SCL go to the same pins as your HT16K33 backpack. I2C devices share the bus, as long as their addresses differ.

You write this firmware now, as part of your design, and flash it once your board is built.

**Full example: encoder counter (Arduino, XIAO RP2040 + TM1637).** Turning the knob counts up and down, and pressing it resets to 0.

```cpp
#include <TM1637Display.h>

const int PIN_CLK = D6;  // TM1637 CLK
const int PIN_DIO = D7;  // TM1637 DIO
const int ENC_A = D8;    // encoder A
const int ENC_B = D9;    // encoder B
const int ENC_SW = D10;  // encoder push switch

TM1637Display display(PIN_CLK, PIN_DIO);
int value = 0;
int lastA = HIGH;

void setup() {
  pinMode(ENC_A, INPUT_PULLUP);
  pinMode(ENC_B, INPUT_PULLUP);
  pinMode(ENC_SW, INPUT_PULLUP);
  display.setBrightness(4);  // 0 (dim) to 7 (bright)
  display.showNumberDec(value);
}

void loop() {
  int a = digitalRead(ENC_A);
  if (a != lastA && a == LOW) {  // A just went LOW: one click
    value += (digitalRead(ENC_B) == HIGH) ? 1 : -1;
    value = constrain(value, 0, 9999);
    display.showNumberDec(value);  // only update when it changes
  }
  lastA = a;

  if (digitalRead(ENC_SW) == LOW) {  // knob pressed: reset
    value = 0;
    display.showNumberDec(value);
    delay(200);  // simple debounce
  }
}
```

The loop watches encoder pin A. Each time it drops LOW, pin B tells us which way the knob turned. For fast spins, an interrupt-based encoder library catches every click.

**Slide pot to percent (MicroPython, XIAO RP2040 + TM1637):**

```python
from machine import Pin, ADC
import time
import tm1637

tm = tm1637.TM1637(clk=Pin(0), dio=Pin(1))  # XIAO D6, D7
pot = ADC(Pin(26))                          # XIAO D0 (A0)

while True:
    percent = pot.read_u16() * 100 // 65535  # 0-65535 -> 0-100
    tm.number(percent)
    time.sleep_ms(50)
```

The display shows the pot's position as 0-100. Try mapping it to a timer length or game speed instead.

**I2C thermometer (CircuitPython, HT16K33 backpack + AHT20):**

```python
import time
import board
import adafruit_ahtx0
from adafruit_ht16k33.segments import Seg7x4

i2c = board.I2C()  # XIAO D4 = SDA, D5 = SCL
display = Seg7x4(i2c)
sensor = adafruit_ahtx0.AHTx0(i2c)

while True:
    # 5 characters like " 23.4" fill all 4 digits (the dot rides along)
    display.print(f"{sensor.temperature:5.1f}")
    time.sleep(1)
```

Both parts share the same two I2C wires. `print()` pushes characters in from the right, so padding to 5 characters overwrites the old reading.

## Coding it

| | Arduino | MicroPython / CircuitPython |
| --- | --- | --- |
| TM1637 module | [avishorp/TM1637](https://github.com/avishorp/TM1637) (or [TM1637TinyDisplay](https://github.com/jasonacox/TM1637TinyDisplay) for scrolling text and animations) | [mcauser/micropython-tm1637](https://github.com/mcauser/micropython-tm1637) |
| Bare display | [SevSeg](https://github.com/DeanIsMe/SevSeg), [SevSegShift](https://github.com/bridystone/SevSegShift) for 74HC595 | Do it by hand, or use a module |
| MAX7219 | [LedControl](https://github.com/wayoda/LedControl) | – |
| HT16K33 backpack | Adafruit LED Backpack library | [adafruit_ht16k33](https://docs.circuitpython.org/projects/ht16k33/en/latest/api.html) (CircuitPython) |

[AceSegment](https://github.com/bxparks/AceSegment) handles every type in one library, but it's aimed at more experienced coders.

**Hello world in Arduino (TM1637).** Install "TM1637" by Avishay Orpaz from the Library Manager.

```cpp
#include <TM1637Display.h>

TM1637Display display(D6, D7);  // CLK, DIO

void setup() {
  display.setBrightness(5);  // 0 (dim) to 7 (bright)
  // 0b01000000 turns the colon on; true keeps leading zeros
  display.showNumberDecEx(1234, 0b01000000, true);  // "12:34"
}

void loop() {}
```

This shows "12:34", and the TM1637 keeps it lit with no help. Change `1234` to any number up to 9999, or pass `0` instead of `0b01000000` to drop the colon.

**Hello world in MicroPython (TM1637).** Once your board is built, install the library with `mpremote mip install github:mcauser/micropython-tm1637`, or copy `tm1637.py` onto it.

```python
from machine import Pin
import tm1637

tm = tm1637.TM1637(clk=Pin(0), dio=Pin(1))  # XIAO D6/D7 or Pico GP0/GP1
tm.brightness(5)        # 0-7
tm.numbers(12, 34)      # "12:34"
tm.scroll("hello")      # scrolls text across
```

`numbers()` shows two numbers around the colon, which suits clocks. Also try `tm.show("help")`, `tm.temperature(24)` and `tm.hex(0xdead)`.

**Bare display with SevSeg (Pico).** These are Pico GP numbers, so change them to match your schematic.

```cpp
#include "SevSeg.h"
SevSeg sevseg;

void setup() {
  byte digitPins[] = {2, 3, 4, 5};
  byte segmentPins[] = {6, 7, 8, 9, 10, 11, 12, 13};  // a-g, then DP
  bool resistorsOnSegments = false;  // our resistors are on the digits
  sevseg.begin(COMMON_CATHODE, 4, digitPins, segmentPins, resistorsOnSegments);
  sevseg.setBrightness(90);
}

void loop() {
  sevseg.setNumber(3141, 3);  // shows 3.141
  sevseg.refreshDisplay();    // must run constantly
}
```

`refreshDisplay()` does the multiplexing, so call it every loop. **Never use `delay()` here.** Use `millis()` for timing instead. For a common anode display, use `COMMON_ANODE`. If your resistors are on the segments instead of the digits, set `resistorsOnSegments = true`.

**CircuitPython with an HT16K33 backpack** is the simplest option: `Seg7x4(i2c)` for 7-segment, `Seg14x4(i2c)` for alphanumeric. Then use `display.print(1234)`, or `display.marquee("hello world ", 0.2)` on the 14-segment one.

## Make it yours

Use these as starting points and design your own thing. Mix inputs, change the case or add a buzzer.

- **Countdown timer.** Set minutes with an encoder, press to start, flash at zero. *Teaches:* `millis()` timing without blocking. Inspiration: [two-digit countdown](https://www.instructables.com/Seven-Segment-Display-Tutorial/).
- **Desk thermometer.** Show the temperature, and press a button to switch to humidity. *Teaches:* I2C sensors and sharing a bus. Inspiration: [LM35 seven-segment thermometer](https://www.instructables.com/Simple-Seven-Segment-Temperature-Display-Using-Ard/).
- **Reaction-time game.** Light up at a random time and show how many ms you took to hit the button. *Teaches:* random numbers and measuring time.
- **Scoreboard.** Two pairs of digits with buttons for each player, maybe on huge displays. *Teaches:* multiplexing and transistor drivers at scale. Inspiration: [big multiplexed displays with shift registers](https://www.instructables.com/Multiplexing-7-Segment-displays-with-Arduino-and-S/).
- **Scrolling name badge.** A 14-segment backpack that scrolls your name and a message picked with a button. *Teaches:* text on segments and battery power.
- **Bare-display challenge.** Skip the module and drive a 4-digit display with your own transistors on a PCB. *Teaches:* how multiplexing really works.

## Sources

- [Instructables: Seven Segment Display Tutorial](https://www.instructables.com/Seven-Segment-Display-Tutorial/)
- [Instructables: Arduino 7 Segment (5011BS, Common Anode or Cathode) Tutorial](https://www.instructables.com/Arduino-7-Segment-5011BS-Common-Anode-or-Cathode-T/)
- [Circuit Basics: How to Set Up a 7 Segment Display on the Arduino](https://www.circuitbasics.com/arduino-7-segment-display-tutorial/)
- [Instructables: Using a 4 Digit & 7 Segment Display, With Arduino](https://www.instructables.com/Using-a-4-digit-7-segment-display-with-arduino/)
- [Instructables: Multiplexing 7 Segment Displays With Arduino and Shift Registers](https://www.instructables.com/Multiplexing-7-Segment-displays-with-Arduino-and-S/)
- [Instructables: Arduino MAX7219 7-Segment Display Tutorial](https://www.instructables.com/MAX7219-7-Segment-Using-Arduino/)
- [Instructables: Simple Seven Segment Temperature Display Using Arduino](https://www.instructables.com/Simple-Seven-Segment-Temperature-Display-Using-Ard/)
- [DeanIsMe/SevSeg](https://github.com/DeanIsMe/SevSeg) and [README](https://raw.githubusercontent.com/DeanIsMe/SevSeg/master/README.md)
- [bridystone/SevSegShift](https://github.com/bridystone/SevSegShift)
- [avishorp/TM1637](https://github.com/avishorp/TM1637), [TM1637Display.h](https://raw.githubusercontent.com/avishorp/TM1637/master/TM1637Display.h) and [TM1637Test example](https://raw.githubusercontent.com/avishorp/TM1637/master/examples/TM1637Test/TM1637Test.ino)
- [jasonacox/TM1637TinyDisplay](https://github.com/jasonacox/TM1637TinyDisplay)
- [mcauser/micropython-tm1637](https://github.com/mcauser/micropython-tm1637) and [tm1637.py](https://raw.githubusercontent.com/mcauser/micropython-tm1637/master/tm1637.py)
- [bxparks/AceSegment](https://github.com/bxparks/AceSegment)
- [wayoda/LedControl](https://github.com/wayoda/LedControl)
- [Titan Micro TM1637 datasheet V2.4](https://akizukidenshi.com/goodsaffix/TM1637_V2.4_EN.PDF)
- [Texas Instruments SN74HC595 datasheet](https://www.ti.com/lit/ds/symlink/sn74hc595.pdf)
- [Adafruit LED Backpacks guide](https://learn.adafruit.com/adafruit-led-backpack?view=all), [0.54" Alphanumeric](https://learn.adafruit.com/adafruit-led-backpack/0-54-alphanumeric), [Pinouts](https://learn.adafruit.com/adafruit-led-backpack/pinouts) and [7-segment Arduino wiring](https://learn.adafruit.com/adafruit-led-backpack/0-dot-56-seven-segment-backpack-arduino-wiring-and-setup)
- [Adafruit: CircuitPython LED backpacks](https://learn.adafruit.com/micropython-hardware-led-backpacks-and-featherwings/circuitpython)
- [adafruit_ht16k33 API docs](https://docs.circuitpython.org/projects/ht16k33/en/latest/api.html), [segments.py](https://raw.githubusercontent.com/adafruit/Adafruit_CircuitPython_HT16K33/main/adafruit_ht16k33/segments.py) and [segments example](https://raw.githubusercontent.com/adafruit/Adafruit_CircuitPython_HT16K33/main/examples/ht16k33_segments_simpletest.py)
- [Adafruit_CircuitPython_AHTx0](https://github.com/adafruit/Adafruit_CircuitPython_AHTx0)
- [Seeed Studio XIAO RP2040 wiki](https://wiki.seeedstudio.com/XIAO-RP2040/)
- [Seeed OPL KiCad Library](https://github.com/Seeed-Studio/OPL_Kicad_Library)
- [Raspberry Pi Pico-series documentation](https://www.raspberrypi.com/documentation/microcontrollers/pico-series.html)
- [KiCad symbol library](https://gitlab.com/kicad/libraries/kicad-symbols) and [KiCad footprint library](https://gitlab.com/kicad/libraries/kicad-footprints)
- [KaiPereira/midi-keypad](https://github.com/KaiPereira/midi-keypad), [Starbie Week 1 Guide](https://github.com/SharKingStudios/Starbie/blob/main/Week%201%20Guide.md), [Blueprint MIDI starter project](https://blueprint.hackclub.com/starter-projects/midi), [Pathfinder](https://pathfinder.hackclub.com/) and [Hermes](https://hermes.hackclub.com/)
- Wikimedia Commons file pages: [LED 7 Segment Display](https://commons.wikimedia.org/wiki/File:LED_7_Segment_Display.jpg), [7-Segment LED Display Cross Section](https://commons.wikimedia.org/wiki/File:7-Segment_LED_Display_Cross_Section.jpg), [74HC595](https://commons.wikimedia.org/wiki/File:74HC595.jpg), [14segment LED Display](https://commons.wikimedia.org/wiki/File:14segment_LED_Display.jpg)
