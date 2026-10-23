# LEDs

## What it is

An LED (light-emitting diode) is a tiny chip that glows when current flows through it. The classic part is a 5 mm round LED with two legs. They use very little power and last for years.

![Labelled drawing of a 5 mm LED showing the long anode leg, the short cathode leg and the flat spot on the rim](/art/starters/displays/leds-1.png)
*Drawing: Inductiveload, Public domain, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:LED,_5mm,_green_(en).svg)*

LEDs only work one way round. The anode (+) is the longer leg, and the cathode (-) is the shorter leg, next to the flat spot on the rim. Wired backwards, an LED stays dark, so double-check which way round each LED goes in your schematic.

An RGB LED packs a red, a green and a blue chip into one 5 mm body, with 4 legs. The longest leg is the shared ("common") one. On a common cathode RGB LED it goes to GND; on a common anode one it goes to + instead.

![Close-up of a clear 5 mm RGB LED with four legs and three tiny coloured chips inside](/art/starters/displays/leds-2.jpg)
*Photo: Mister rf, CC BY-SA 4.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:LED_5mm_RGB.jpg)*

## How it works

### Forward voltage and the resistor

Every LED drops a fixed amount of voltage called the forward voltage (Vf). Blue, white and bright green LEDs drop more than red and yellow ones.

**Always put a resistor in series with an LED.** Without one, the LED pulls far too much current and burns out. The resistor can go on either side of the LED. Pick its value from this table:

| LED colour | Vf (about) | 3.3 V pin, ~5 mA | 5 V via a transistor, ~15 mA |
| --- | --- | --- | --- |
| Red | 2.0 V | 270 Ω | 220 Ω |
| Yellow / orange | 2.1 V | 270 Ω | 220 Ω |
| Green (bright), blue, white | 3.0 V | 68-100 Ω | 150 Ω |

Straight from a 3.3 V pin, blue and white LEDs only get about 0.3 V of headroom, so they light dimly. For full brightness, switch them from 5 V with a transistor.

### PWM dimming

A pin is either on or off. To dim an LED, the pin switches it on and off hundreds of times a second and changes how long it stays on each time. This is PWM (pulse-width modulation), and your eye sees it as a steady, dimmer light.

### Driving lots of LEDs

Pins run out fast, and each pin can only supply a little current. The RP2040 on a Pico or XIAO defaults to 4 mA per pin and 50 mA total across all its GPIO.

There are four standard ways round this:

- **Transistor or MOSFET:** a pin switches a MOSFET, and the MOSFET carries the big current. Use it for high-power LEDs, 5-12 V LED strings, or many LEDs at once.
- **Shift register (74HC595):** 3 pins control 8 outputs, and chained chips give 16, 24 or more.
- **Multiplexing:** arrange LEDs in rows and columns and light one row at a time, fast enough that they all look lit. A 4-digit display then needs 12 pins instead of 32.
- **Charlieplexing:** N pins drive N × (N - 1) LEDs, so 3 pins give 6 and 4 pins give 12.

![Schematic of 3-pin charlieplexing: six LEDs between pins X1, X2 and X3, one resistor per pin](/art/starters/displays/leds-4.png)
*Diagram: Matthiaspaul, CC BY-SA 3.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:3-pin_Charlieplexing_with_common_resistors.svg)*

Current always passes through two resistors, so use half the normal resistor value on each pin.

## Using it in a project

### Parts for your design

These are the parts that go in your schematic and your BOM. Your funding covers them once your design is approved.

| Part | What it's for | Typical driver | Notes |
| --- | --- | --- | --- |
| 5 mm LEDs, diffused, mixed colours | Status lights, patterns | GPIO pin + resistor | Diffused LEDs are easier on the eyes than clear ones |
| 5 mm RGB LED, 4-leg | Colour mixing | 3 PWM pins + 3 resistors | Check common anode vs common cathode on the datasheet |
| 0805 or 0603 SMD LEDs | Neat PCB indicators | GPIO pin + resistor | Sit flat, which suits status lights under a diffuser |
| Resistors (100 Ω-10 kΩ) | Current limiting, pull-downs | – | 220 Ω, 330 Ω and 1 kΩ get used most |
| 74HC595 shift register (DIP-16) | 8 more outputs from 3 pins | Itself | Max 70 mA through the whole chip |
| AO3400A N-MOSFET (SOT-23) | Switching big LED loads | GPIO pin to gate | Fully on at 2.5 V gate, so fine for 3.3 V boards |
| 10 kΩ potentiometer, tactile buttons | Inputs | ADC pin / GPIO | See "Adding inputs" |

### On a PCB

Most of you will put your LEDs on a custom KiCad board with a XIAO or a Pico, as in Pathfinder. Your first layout can be messy, since you can re-route it later.

**1. Get your libraries.** KiCad's built-in libraries cover LEDs, resistors, the 74HC595, MOSFETs and the Pico. For a XIAO, add Seeed's [OPL KiCad library](https://github.com/Seeed-Studio/OPL_Kicad_Library) via Preferences → Manage Symbol Libraries and Manage Footprint Libraries.

**2. Draw the schematic.** Press A to add symbols, W to draw wires and L for net labels.

- Single LED: `Device:LED`. Pin 1 is K (cathode), pin 2 is A (anode).
- Resistor: `Device:R` or the tidier `Device:R_Small`.
- RGB LED: pick the `Device:LED_xxxx` symbol whose letters match your LED's pin order. A common 5 mm common cathode part is R, K, G, B, so use `Device:LED_RKGB`. Common anode R, A, G, B is `Device:LED_RAGB`.
- Microcontroller: `MCU_Module:RaspberryPi_Pico`, or the XIAO symbol from Seeed's library.
- Power: `power:+3V3` (pointing up) and `power:GND` (pointing down).

Wire each LED as GPIO net label → resistor → anode, cathode → GND. Use clear labels like `LED_RED`, and press E on each resistor to set its value.

Which pins to connect in your schematic:

| Connection | Raspberry Pi Pico (3.3 V) | XIAO RP2040 (3.3 V) |
| --- | --- | --- |
| Output pin → resistor → anode | GP15 | D6 (GPIO0) |
| Resistor value (red LED) | 270 Ω | 270 Ω |
| Cathode → | GND (pin 38) | GND |
| RGB: R, G, B pins (each via a resistor) | GP13, GP14, GP15 | D6, D7, D8 |
| RGB common leg (common cathode) | GND | GND |

On the Pico and XIAO, any GPIO can do PWM, so fading LEDs can go on any pin.

Check your schematic for these common mistakes:

- **No resistor.** The LED flashes bright once and never again. Every LED needs one.
- **Common anode RGB on a 3.3 V board.** Connect the common leg to 3V3, not 5 V, and remember LOW = on.
- **Too many LEDs on pins.** Eight LEDs at 5 mA each is already 40 mA, close to the RP2040's 50 mA total. Use a 74HC595 or transistors past that.
- **74HC595 on a 3.3 V board.** Power the chip from 3.3 V too, not 5 V.

For a 74HC595, use `74xx:74HC595`. Tie SRCLR (pin 10) to +3V3 and OE (pin 13) to GND, or to a PWM pin to dim all 8 at once. Add a 100 nF capacitor from VCC to GND for every chip.

For a MOSFET switch, use `Transistor_FET:AO3400A`. Wire GPIO → 100 Ω → gate, plus 100 kΩ from gate to GND. The LEDs and their resistor go from your supply to the drain, and the source goes to GND.

**3. Assign footprints.** Click "Assign footprints" in the schematic toolbar and search these names:

| Part | Footprint |
| --- | --- |
| 5 mm LED | `LED_THT:LED_D5.0mm` |
| 5 mm LED, lying flat at the board edge | `LED_THT:LED_D5.0mm_Horizontal_O1.27mm_Z3.0mm` |
| 5 mm RGB LED | `LED_THT:LED_D5.0mm-4_RGB` |
| SMD LED | `LED_SMD:LED_0805_2012Metric_Pad1.15x1.40mm_HandSolder` or `LED_SMD:LED_0603_1608Metric` |
| Through-hole resistor | `Resistor_THT:R_Axial_DIN0207_L6.3mm_D2.5mm_P7.62mm_Horizontal` |
| SMD resistor | `Resistor_SMD:R_0805_2012Metric_Pad1.20x1.40mm_HandSolder` |
| 74HC595 | `Package_DIP:DIP-16_W7.62mm` or `Package_SO:SOIC-16_3.9x9.9mm_P1.27mm` |
| 100 nF capacitor | `Capacitor_SMD:C_0805_2012Metric` or `Capacitor_THT:C_Disc_D5.0mm_W2.5mm_P5.00mm` |
| AO3400A | `Package_TO_SOT_SMD:SOT-23` |
| Off-board LED cable | `Connector_JST:JST_PH_B2B-PH-K_1x02_P2.00mm_Vertical` |
| Pico | `Module:RaspberryPi_Pico_Common_THT` |

Through-hole LEDs can poke through holes in a case. SMD 0805 or 0603 LEDs sit flat, which suits status lights under a diffuser.

**4. Place parts.** Click "Update PCB from Schematic" and drop everything inside an Edge.Cuts outline. Put each resistor next to its LED, and line LED rows up on a 1 mm grid.

The square pad on KiCad's LED footprints is pad 1, the cathode. Its silkscreen circle has a flat side on that pad, matching the flat spot on the LED's rim. Put the 74HC595's 100 nF cap as close to pin 16 as you can.

**5. Route.** Press X to route and V to drop a via. Vias cost nothing extra, so use as many as you need.

- LED signal traces carry 20 mA or less, so 0.25 mm is wide enough.
- Power traces feeding lots of LEDs or a MOSFET should be 0.5 mm, or about 1 mm for 1 A.
- Avoid sharp 90° corners and keep traces short and direct.
- Add a GND fill on the back layer so every cathode has an easy path home.

**6. Check before you submit.** Run Inspect → Design Rules Checker and fix every error. Warnings are usually fine, but unconnected items are not. Then open the 3D viewer (Alt+3) and check that every LED's flat side points the way you expect.

## Adding inputs

Inputs let your LEDs react to the user. This is how to wire the inputs used in Hack Club projects:

- **Tactile or low-profile switch:** one side to a GPIO, other side to GND. Turn on the internal pull-up, so pressed reads 0. Symbol: `Switch:SW_Push`, footprint `Button_Switch_THT:SW_PUSH_6mm`.
- **Lots of buttons:** use a key matrix with a 1N4148 diode per switch. A 6 × 6 grid then needs only 12 pins.
- **EC11 rotary encoder:** A and B to two GPIOs, C to GND. The push switch wires up like another button. Symbol `Device:RotaryEncoder_Switch`, footprint `Rotary_Encoder:RotaryEncoder_Alps_EC11E-Switch_Vertical_H20mm`.
- **Bourns slide pot:** pin 1 to GND, pin 3 to 3V3 and the middle pin to an ADC pin. Footprint `Potentiometer_THT:Potentiometer_Bourns_PTA4543_Single_Slide`.
- **I2C light sensor (BH1750):** SDA and SCL to the XIAO's D4 and D5, plus 3V3 and GND. It reads brightness in lux, which you can use for auto-dimming.

### Example: slider sets brightness, button changes colour

This MicroPython sketch is firmware you write now and flash once your board is built. It's for a XIAO RP2040. A common cathode RGB LED sits on D6, D7 and D8, the slide pot on A0 and the button on D1.

```python
from machine import Pin, PWM, ADC
import time

# XIAO RP2040 pin names -> GPIO numbers: D6=0, D7=1, D8=2, A0=26, D1=27
red = PWM(Pin(0), freq=1000)
green = PWM(Pin(1), freq=1000)
blue = PWM(Pin(2), freq=1000)
slider = ADC(Pin(26))
button = Pin(27, Pin.IN, Pin.PULL_UP)  # pressed = 0

COLOURS = [(1, 0, 0), (0, 1, 0), (0, 0, 1), (1, 1, 0), (0, 1, 1), (1, 0, 1), (1, 1, 1)]
colour = 0
was_pressed = False

while True:
    pressed = button.value() == 0
    if pressed and not was_pressed:  # only act on a fresh press
        colour = (colour + 1) % len(COLOURS)
    was_pressed = pressed

    level = slider.read_u16()  # 0-65535
    r, g, b = COLOURS[colour]
    red.duty_u16(level * r)
    green.duty_u16(level * g)
    blue.duty_u16(level * b)
    time.sleep_ms(20)  # short pause also debounces the button
```

Each press steps through 7 colours, and the slider sets brightness from off to full. For a common anode LED, add `invert=True` to each `PWM(...)` call. To add your own colours, add tuples to `COLOURS`.

The same sketch in Arduino C++, with the same pins:

```cpp
const int redPin = D6;
const int greenPin = D7;
const int bluePin = D8;
const int buttonPin = D1;
const int sliderPin = A0;

const byte colours[7][3] = {
  {1, 0, 0}, {0, 1, 0}, {0, 0, 1}, {1, 1, 0}, {0, 1, 1}, {1, 0, 1}, {1, 1, 1}
};
int colour = 0;
bool wasPressed = false;

void setup() {
  pinMode(redPin, OUTPUT);
  pinMode(greenPin, OUTPUT);
  pinMode(bluePin, OUTPUT);
  pinMode(buttonPin, INPUT_PULLUP);  // pressed reads LOW
}

void loop() {
  bool pressed = digitalRead(buttonPin) == LOW;
  if (pressed && !wasPressed) colour = (colour + 1) % 7;
  wasPressed = pressed;

  int level = analogRead(sliderPin) / 4;  // 0-1023 -> 0-255
  analogWrite(redPin, level * colours[colour][0]);
  analogWrite(greenPin, level * colours[colour][1]);
  analogWrite(bluePin, level * colours[colour][2]);
  delay(20);
}
```

### Rotary encoder: spin to pick an LED (CircuitPython)

CircuitPython's built-in `rotaryio` module reads encoders for you. On RP2040 boards, A and B must be on neighbouring GPIOs, so D6 and D7 (GPIO0 and GPIO1) work.

```python
import board
import digitalio
import rotaryio

encoder = rotaryio.IncrementalEncoder(board.D6, board.D7)
leds = []
for pin in (board.D0, board.D1, board.D2, board.D3):  # 4 LEDs, each with a resistor
    led = digitalio.DigitalInOut(pin)
    led.direction = digitalio.Direction.OUTPUT
    leds.append(led)

while True:
    lit = encoder.position % len(leds)  # wraps around: 0, 1, 2, 3, 0...
    for i, led in enumerate(leds):
        led.value = i == lit
```

Turning the knob walks the light along the row. Swap the `%` for a clamp to stop at the ends instead.

### Light sensor: auto-dim at night (MicroPython)

```python
from machine import Pin, PWM, I2C
import time

i2c = I2C(1, scl=Pin(7), sda=Pin(6))  # XIAO D5 = SCL, D4 = SDA
BH1750 = 0x23                          # address with ADDR pin low or floating
i2c.writeto(BH1750, b"\x10")           # continuous mode, 1 lux resolution
time.sleep_ms(180)                     # first reading takes up to 180 ms

led = PWM(Pin(0), freq=1000)           # D6

while True:
    raw = i2c.readfrom(BH1750, 2)
    lux = ((raw[0] << 8) | raw[1]) / 1.2
    level = min(int(lux / 500 * 65535), 65535)  # 500 lux or more = full brightness
    led.duty_u16(max(level, 2000))              # never fully off
    time.sleep_ms(200)
```

The darker the room, the dimmer the LED, but it never goes fully off. Change `500` to tune how bright the room must be for full power.

## Coding it

Write your firmware now, alongside your schematic, and flash it once your board is built. Plain LEDs don't need a display library:

- **Arduino:** the built-in core functions [`digitalWrite`/`analogWrite`](https://github.com/arduino/reference-en/blob/master/Language/Functions/Analog%20IO/analogWrite.adoc) and [`shiftOut`](https://github.com/arduino/reference-en/blob/master/Language/Functions/Advanced%20IO/shiftOut.adoc). For a XIAO RP2040 or Pico, install the [arduino-pico core](https://arduino-pico.readthedocs.io/en/latest/analog.html).
- **MicroPython:** [`machine.Pin`](https://docs.micropython.org/en/latest/library/machine.Pin.html) and [`machine.PWM`](https://docs.micropython.org/en/latest/library/machine.PWM.html).
- **CircuitPython:** `digitalio` and [`pwmio`](https://docs.circuitpython.org/en/latest/shared-bindings/pwmio/index.html), plus [`rotaryio`](https://docs.circuitpython.org/en/latest/shared-bindings/rotaryio/index.html) for encoders.

### Hello world: Arduino C++

```cpp
const int ledPin = D6;  // XIAO RP2040 (Pico: 15)

void setup() {
  pinMode(ledPin, OUTPUT);
}

void loop() {
  digitalWrite(ledPin, HIGH);  // on
  delay(500);
  digitalWrite(ledPin, LOW);   // off
  delay(500);

  for (int b = 0; b <= 255; b++) {  // fade up with PWM
    analogWrite(ledPin, b);
    delay(4);
  }
  for (int b = 255; b >= 0; b--) {  // and back down
    analogWrite(ledPin, b);
    delay(4);
  }
}
```

The LED blinks once, then breathes up and down. `analogWrite` takes 0-255, where 0 is off and 255 is full on. Change the `delay(4)` to speed up or slow down the fade.

### Hello world: MicroPython

```python
from machine import Pin, PWM
import time

led = Pin(15, Pin.OUT)  # Pico GP15 (on a XIAO RP2040, D6 is Pin(0))
led.on()
time.sleep(0.5)
led.off()

pwm = PWM(led, freq=1000)
while True:
    for b in range(0, 65536, 512):  # fade up
        pwm.duty_u16(b)
        time.sleep_ms(4)
    for b in range(65535, -1, -512):  # fade down
        pwm.duty_u16(b)
        time.sleep_ms(4)
```

Same idea: one blink, then a smooth fade. MicroPython duty runs from 0 to 65535 instead of 0-255.

### Eight LEDs from three pins: 74HC595

```cpp
const int latchPin = D9;  // to RCLK, pin 12
const int clockPin = D8;  // to SRCLK, pin 11
const int dataPin = D10;  // to SER, pin 14

void setup() {
  pinMode(latchPin, OUTPUT);
  pinMode(clockPin, OUTPUT);
  pinMode(dataPin, OUTPUT);
}

void loop() {
  for (int n = 0; n < 256; n++) {
    digitalWrite(latchPin, LOW);
    shiftOut(dataPin, clockPin, MSBFIRST, n);  // send 8 bits
    digitalWrite(latchPin, HIGH);              // all 8 outputs update at once
    delay(200);
  }
}
```

This is a binary counter: the 8 LEDs on QA-QH show 0 to 255. To chain a second chip, wire its SER to the first chip's QH' (pin 9), share clock and latch, and call `shiftOut` twice.

In MicroPython, send the bits yourself:

```python
from machine import Pin

data = Pin(3, Pin.OUT)   # XIAO D10 -> SER (pin 14)
clock = Pin(2, Pin.OUT)  # XIAO D8  -> SRCLK (pin 11)
latch = Pin(4, Pin.OUT)  # XIAO D9  -> RCLK (pin 12)

def show(byte):
    latch.off()
    for i in range(7, -1, -1):  # most significant bit first
        data.value((byte >> i) & 1)
        clock.on()
        clock.off()
    latch.on()

show(0b10101010)  # every other LED on
```

### Six LEDs from three pins: charlieplexing

This matches the charlieplexing diagram above, with X1-X3 on D0-D2.

```cpp
const int pins[3] = {D0, D1, D2};  // X1, X2, X3, each through a resistor
// {anode, cathode} for LED1-LED6 in the diagram
const int leds[6][2] = {{0, 1}, {1, 0}, {1, 2}, {2, 1}, {0, 2}, {2, 0}};

void lightLed(int n) {
  for (int i = 0; i < 3; i++) pinMode(pins[i], INPUT);  // all "disconnected"
  pinMode(pins[leds[n][0]], OUTPUT);
  digitalWrite(pins[leds[n][0]], HIGH);
  pinMode(pins[leds[n][1]], OUTPUT);
  digitalWrite(pins[leds[n][1]], LOW);
}

void setup() {}

void loop() {
  for (int n = 0; n < 6; n++) {
    lightLed(n);
    delay(200);  // drop to 2 ms to make them all look lit at once
  }
}
```

In MicroPython, set a pin to `Pin(x, Pin.IN)` to disconnect it.

## Make it yours

Use these as starting points and change them into your own project.

- **Electronic dice.** Seven LEDs in a die pattern and a button to roll. Teaches random numbers and mapping numbers to LED patterns. See: [Arduino Dice](https://www.instructables.com/Arduino-Dice-1/).
- **Binary clock.** Show hours and minutes in binary on 74HC595-driven LEDs. Teaches shift register chaining and timekeeping. See: [RGB Binary Clock](https://www.instructables.com/RGB-Binary-CLock/).
- **Charlieplexed ring.** 12 LEDs on 4 pins in a circle, as a clock or a knob position display. Teaches charlieplexing and fast refresh. See: [Charlieplexing Made Easy](https://www.instructables.com/Charlieplexing-Made-Easy-and-What-It-Even-Means/).
- **3 × 3 × 3 LED cube.** 27 LEDs in layers, lit one layer at a time. Teaches multiplexing and transistor layer switching. See: [3x3x3 LED Cube](https://www.instructables.com/3x3x3-LED-Cube-3/).
- **Distance stoplight.** Green, yellow and red LEDs that change as something gets closer. Teaches reading a sensor and making decisions. See: SparkFun's [LED animations tutorial](https://learn.sparkfun.com/tutorials/basic-led-animations-for-beginners-arduino/all).
- **Controller pad.** Buttons with matching LEDs on your own PCB that light up when pressed. Teaches the full KiCad-to-firmware flow. See: [Pathfinder](https://pathfinder.hackclub.com/).

Change the layout, the colours, the shape of the PCB and what the lights mean to make the board your own.

## Sources

- [SparkFun: Light-Emitting Diodes (LEDs)](https://learn.sparkfun.com/tutorials/light-emitting-diodes-leds/all)
- [Adafruit: Arduino Lesson 2 (LEDs)](https://learn.adafruit.com/adafruit-arduino-lesson-2-leds?view=all)
- [SparkFun: Basic LED Animations for Beginners (Arduino)](https://learn.sparkfun.com/tutorials/basic-led-animations-for-beginners-arduino/all)
- [Instructables: Charlieplexing Made Easy](https://www.instructables.com/Charlieplexing-Made-Easy-and-What-It-Even-Means/)
- [Hackaday: Another way to look at Charlieplexing](https://hackaday.com/2013/04/08/another-way-to-look-at-charlieplexing/)
- [Instructables: Practical Guide to LEDs 4 (Matrix & Multiplexing)](https://www.instructables.com/Practical-Guide-to-LEDs-4-Matrix-Multiplexing/)
- [Instructables: LED Blink With Arduino](https://www.instructables.com/LED-BLINK-WITH-ARDUINO)
- [Arduino reference: analogWrite](https://github.com/arduino/reference-en/blob/master/Language/Functions/Analog%20IO/analogWrite.adoc)
- [Arduino reference: shiftOut](https://github.com/arduino/reference-en/blob/master/Language/Functions/Advanced%20IO/shiftOut.adoc)
- [arduino-pico: Analog I/O](https://arduino-pico.readthedocs.io/en/latest/analog.html)
- [MicroPython: machine.Pin](https://docs.micropython.org/en/latest/library/machine.Pin.html)
- [MicroPython: machine.PWM](https://docs.micropython.org/en/latest/library/machine.PWM.html)
- [MicroPython: machine.ADC](https://docs.micropython.org/en/latest/library/machine.ADC.html)
- [MicroPython: machine.I2C](https://docs.micropython.org/en/latest/library/machine.I2C.html)
- [MicroPython: RP2 quick reference](https://docs.micropython.org/en/latest/rp2/quickref.html)
- [CircuitPython: pwmio](https://docs.circuitpython.org/en/latest/shared-bindings/pwmio/index.html)
- [CircuitPython: rotaryio](https://docs.circuitpython.org/en/latest/shared-bindings/rotaryio/index.html)
- [CircuitPython: rotaryio RP2040 source (sequential pins)](https://github.com/adafruit/circuitpython/blob/main/ports/raspberrypi/common-hal/rotaryio/IncrementalEncoder.c)
- [CircuitPython: XIAO RP2040 pin names](https://github.com/adafruit/circuitpython/blob/main/ports/raspberrypi/boards/seeeduino_xiao_rp2040/pins.c)
- [Texas Instruments: SN74HC595 datasheet](https://www.ti.com/lit/ds/symlink/sn74hc595.pdf)
- [Raspberry Pi: RP2040 datasheet](https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf)
- [Raspberry Pi: Pico datasheet](https://datasheets.raspberrypi.com/pico/pico-datasheet.pdf)
- [Raspberry Pi: Pico-series documentation](https://www.raspberrypi.com/documentation/microcontrollers/pico-series.html)
- [Alpha & Omega: AO3400A datasheet](https://www.aosmd.com/res/datasheets/AO3400A.pdf)
- [Seeed Studio: XIAO RP2040 wiki](https://wiki.seeedstudio.com/XIAO-RP2040/)
- [Seeed Studio: XIAO RP2040 with Arduino](https://wiki.seeedstudio.com/XIAO-RP2040-with-Arduino/)
- [Seeed Studio: OPL KiCad Library](https://github.com/Seeed-Studio/OPL_Kicad_Library)
- [KiCad symbol libraries](https://gitlab.com/kicad/libraries/kicad-symbols)
- [KiCad footprint libraries](https://gitlab.com/kicad/libraries/kicad-footprints)
- [claws/BH1750 Arduino library](https://github.com/claws/BH1750)
- [Adafruit CircuitPython BH1750 driver](https://github.com/adafruit/Adafruit_CircuitPython_BH1750)
- [Wikipedia: Light-emitting diode physics](https://en.wikipedia.org/wiki/Light-emitting_diode_physics)
- [Kai Pereira: MIDI keypad guide](https://github.com/KaiPereira/midi-keypad)
- [Starbie Week 1 Guide](https://github.com/SharKingStudios/Starbie/blob/main/Week%201%20Guide.md)
- [Blueprint: MIDI starter project](https://blueprint.hackclub.com/starter-projects/midi)
- [Pathfinder](https://pathfinder.hackclub.com/)
- [Hermes](https://hermes.hackclub.com/)
- [Instructables: Arduino Dice](https://www.instructables.com/Arduino-Dice-1/)
- [Instructables: RGB Binary Clock](https://www.instructables.com/RGB-Binary-CLock/)
- [Instructables: 3x3x3 LED Cube](https://www.instructables.com/3x3x3-LED-Cube-3/)
