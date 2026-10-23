# LED Matrices

## What it is

The classic part is an 8x8 matrix: 64 LEDs in one plastic block, 32 mm square. On its own it has 16 pins and needs a lot of babysitting. So it almost always comes on a small board with a MAX7219 driver chip that handles the hard part.

You'll see two flavours of MAX7219 module. The single module is one 8x8 matrix plus one chip. The "4-in-1" module is four of them already chained into a 32x8 strip, which suits scrolling messages.

![An 8x8 LED matrix block with two of its pins visible underneath](/art/starters/displays/matrices-1.jpg)
*Photo: oomlout, CC BY-SA 2.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:8x8_Bicolour_LED_Matrix.jpg)*

There are other kinds too. HT16K33 backpacks drive an 8x8 over I2C, using two wires. NeoPixel matrices give every dot its own full colour. HUB75 panels are the large full-colour panels used in shop signs. We'll focus on the MAX7219 and cover the rest briefly.

![The back of a HUB75 RGB panel, covered in driver chips, with a ribbon cable and power leads to an adapter board](/art/starters/displays/matrices-3.jpg)
*Photo: SimonWaldherr, CC BY-SA 4.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Hub75_LED_Matrix_with_Raspberry_Pi.jpg)*

## How it works

Inside, the LEDs are wired in a grid. Every LED in a row shares one wire, and every LED in a column shares another. To light the LED at row 3, column 5, you power row 3 and ground column 5.

![Circuit diagram of a 3x3 LED matrix: each LED sits where a row wire crosses a column wire](/art/starters/displays/matrices-2.png)
*Diagram: Jimmykraemer, CC BY-SA 4.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:DotMatrixCircuitDiagram.png)*

But if you power two rows and two columns at once, you light four LEDs instead of two. So drivers use multiplexing. They light one row at a time, then the next, and so on, very fast. Your eye blends it into a steady picture.

That's why you want a driver chip. Doing it by hand needs 16 GPIO pins, 8 resistors and a CPU that never stops refreshing. The MAX7219 does all of that by itself:

- It scans the 8 rows about 800 times a second (datasheet: 500-1300 Hz).
- It sets the LED current with one resistor, called RSET, instead of one per LED.
- It talks SPI over 3 wires: DIN (data), CLK (clock) and CS (chip select, also called LOAD).
- It has 16 brightness levels you can set from code.
- It has a DOUT pin, so you can daisy-chain as many modules as you like on the same 3 wires.

The trade-off is brightness. Each row is only on 1/8 of the time, so the chip pushes higher current in short pulses. The datasheet allows up to 40 mA per LED, and up to 320 mA through one row line.

## Using it in a project

### Parts for your design

These are the parts that go in your schematic and your BOM. Once your design is approved, your funding pays for them.

| Part | What it's for | Driver chip / module | Notes |
| --- | --- | --- | --- |
| Single 8x8 MAX7219 module | Icons, faces, first experiments | MAX7219 (DIP) | 5 V. Has DIN/CS/CLK in and DOUT out |
| 4-in-1 32x8 module ("FC-16") | Scrolling text, clocks | 4x MAX7219 (SMD) | Use `FC16_HW` in MD_Parola. Can draw ~1 A |
| Bare 8x8 matrix + MAX7219 chip | Your own PCB | MAX7219CNG (DIP-24) or MAX7219CWG (SO-24) | Check the matrix datasheet for pinout |
| 8x8 or 16x8 HT16K33 backpack | Only 2 pins free, many displays on one bus | HT16K33 | I2C, addresses 0x70-0x77 |
| NeoPixel matrix (8x8, 16x16, 8x32) | Full colour per pixel | WS2812/SK6812 in each LED | See the NeoPixel tab. 3 bytes of RAM per pixel |
| HUB75 RGB panel (32x16 to 64x32) | Big, bright, full colour | Shift registers on the panel | Needs 12-13 pins, a fast board and 2-4 A at 5 V |

### On a PCB

Most of you will put the matrix on a custom KiCad board with a XIAO or Pico. There are two routes. Pick one.

#### Route A: plug in a ready-made module (easiest)

Your PCB carries a 5-pin socket and the module plugs in.

1. In the Schematic Editor, press **A** and add `Connector_Generic:Conn_01x05`. Give it footprint `Connector_PinSocket_2.54mm:PinSocket_1x05_P2.54mm_Vertical`.
2. Add your dev board symbol. Stock KiCad libraries may not include XIAO or Pico symbols. Use the care package from your program, or Seeed's [OPL KiCad library](https://github.com/Seeed-Studio/OPL_Kicad_Library) for the XIAO.
3. Press **L** and add net labels: `MTX_DIN`, `MTX_CLK`, `MTX_CS`. Press **P** for `+5V` and `GND`. Labels save you from long wires.
4. Add a 10 uF capacitor across +5V and GND near the socket (`Device:C`, footprint `Capacitor_SMD:C_0805_2012Metric`).
5. To chain modules, DOUT of one goes to DIN of the next. VCC, GND, CLK and CS are shared.

The socket matches the module's 5-pin input header: VCC, GND, DIN, CS, CLK, on the end labelled DIN (the DOUT end only feeds the next module). In your schematic, connect each pin to these dev board pins:

| Module pin | Raspberry Pi Pico | Seeed XIAO RP2040 | ESP32 DevKit |
| --- | --- | --- | --- |
| VCC | VBUS (pin 40) | 5V | VIN / 5V |
| GND | GND | GND | GND |
| DIN | GP19 | D10 (MOSI) | GPIO23 |
| CLK | GP18 | D8 (SCK) | GPIO18 |
| CS | GP17 | D7 | GPIO5 |

#### Route B: put the chip on your board

1. Add `Driver_LED:MAX7219`. Footprint: `Package_DIP:DIP-24_W7.62mm` for the through-hole MAX7219CNG, or `Package_SO:SOIC-24W_7.5x15.4mm_P1.27mm` for the MAX7219CWG.
2. Connect RSET between V+ (pin 19) and ISET (pin 18). The datasheet table gives 11.8 kOhm for 40 mA with red LEDs. For red LEDs, start with 18 kOhm (just under 30 mA) for less heat. **Never go below 9.53 kOhm.**
3. Add 10 uF and 100 nF between V+ and GND, as close to pin 19 as you can. The datasheet asks for exactly this pair. Connect both GND pins (4 and 9).
4. KiCad has no 8x8 matrix symbol. Make one with 16 pins, or use two `Conn_01x08` symbols. For the footprint, use two `PinSocket_1x08_P2.54mm_Vertical` rows spaced as your matrix datasheet says. Sockets let you swap a dead matrix.
5. Wire DIG0-DIG7 to the matrix's common-cathode lines and SEG A-G plus DP to the anode lines. Matrix pinouts are scrambled, so follow the datasheet pin by pin with net labels.
6. For a 3.3 V board, add a level shifter: `74xx:74AHCT125`, footprint `Package_SO:SOIC-14_3.9x8.7mm_P1.27mm`. Power it from 5 V, tie its OE pins to GND, and give it its own 100 nF cap. Put DIN, CLK and CS through it.
7. To chain a second chip, DOUT (pin 24) goes to the next DIN. CLK and CS are shared.

#### Schematic checks, whichever route you pick

- **Always power the MAX7219 from 5 V, never from 3.3 V.** The chip needs 4.0-5.5 V.
- **Power budget.** Plan for up to 320 mA per fully lit module. ShillehTek measures about 150 mA per module in practice. A USB port gives about 500 mA. Keep brightness at 1-4 on USB, or add a 5 V 2 A supply for a 4-in-1.
- **3.3 V boards and logic levels.** The datasheet wants at least 3.5 V for a logic "high". A Pico or ESP32 only outputs 3.3 V, which is out of spec, so add a level shifter (step 6 above).
- **No resistors needed for a module.** It already has RSET and capacitors on board.
- **Module type.** Pick the right one in code, or the text comes out garbled or mirrored. More on that in Coding it.

#### Placement and routing

- Put the MAX7219 right next to the matrix, or underneath it on the back. The datasheet says to keep these connections short.
- Use 0.25 mm traces for signals. Use 0.5 mm for the DIG lines, since each can sink 320 mA.
- Use 0.8-1.0 mm for +5V and GND to the display. A 4-in-1 can pull about 1 A.
- Don't feed a big chain through your dev board. Add a separate 5 V input, like a USB-C breakout or a 2-pin JST (`Connector_JST:JST_PH_S2B-PH-K_1x02_P2.00mm_Horizontal`).
- Fill the back layer with a GND pour.
- The chip gets warm at full brightness. Leave copper around it and don't bury it under foam.

**Check before you submit:** run DRC and fix every unconnected-net error. Open the 3D viewer and make sure the matrix doesn't sit on top of your XIAO. Measure the gap between your two pin rows against the matrix datasheet so they line up.

## Adding inputs

Inputs let people interact with the matrix. This is how to wire the usual parts on a XIAO RP2040 board:

- **Buttons:** `Switch:SW_Push`, footprint `Button_Switch_THT:SW_PUSH_6mm` (or low-profile Choc switches). One side to a GPIO, other side to GND. Use `INPUT_PULLUP`, so no resistor needed.
- **Many buttons:** use a key matrix. Rows and columns go to GPIOs, with a 1N4148 diode per key (`Diode_SMD:D_SOD-123`), all facing the same way.
- **EC11 rotary encoder:** `Device:RotaryEncoder_Switch_MP`, footprint `Rotary_Encoder:RotaryEncoder_Alps_EC11E-Switch_Vertical_H20mm`. A and B to GPIOs, C and the mounting pins to GND. The push switch is just another button.
- **Slide pot:** `Device:R_Potentiometer_MountingPin`, footprint `Potentiometer_THT:Potentiometer_Bourns_PTA2043_Single_Slide`. Pin 1 to GND, pin 3 to 3V3 (not 5 V), wiper to an analog pin like A0.
- **I2C light sensor (BH1750):** add a 4-pin Qwiic/STEMMA QT socket (`Connector_JST:JST_SH_SM04B-SRSS-TB_1x04-1MP_P1.00mm_Horizontal`): GND, 3V3, SDA (D4), SCL (D5). Address 0x23.

You write the firmware now and flash it once your board is built. This complete Arduino sketch is for a XIAO RP2040 and a 4-in-1 module. The slide pot sets brightness, the encoder sets scroll speed, and the button swaps the message.

```cpp
#include <MD_Parola.h>
#include <MD_MAX72xx.h>
#include <SPI.h>

#define HARDWARE_TYPE MD_MAX72XX::FC16_HW
#define MAX_DEVICES 4
#define CS_PIN D7          // DIN -> D10, CLK -> D8 (hardware SPI)

const int POT_PIN = A0;    // slide pot wiper
const int ENC_A = D1;      // encoder A
const int ENC_B = D2;      // encoder B
const int BUTTON_PIN = D3; // button to GND

MD_Parola display = MD_Parola(HARDWARE_TYPE, CS_PIN, MAX_DEVICES);

const char *messages[] = { "Hello!", "Make it yours", "8x8 forever" };
int current = 0;
int speed = 50;            // ms per frame, lower = faster
int lastA = HIGH;
int lastButton = HIGH;
int lastLevel = -1;
unsigned long lastPress = 0;

void setup() {
  pinMode(ENC_A, INPUT_PULLUP);
  pinMode(ENC_B, INPUT_PULLUP);
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  display.begin();
  display.displayScroll(messages[current], PA_LEFT, PA_SCROLL_LEFT, speed);
}

void loop() {
  // Slide pot -> brightness 0-15 (only send when it changes)
  int level = map(analogRead(POT_PIN), 0, 1023, 0, 15);
  if (level != lastLevel) {
    display.setIntensity(level);
    lastLevel = level;
  }

  // Encoder -> speed
  int a = digitalRead(ENC_A);
  if (a != lastA && a == LOW) {
    speed += (digitalRead(ENC_B) == HIGH) ? -5 : 5;
    speed = constrain(speed, 10, 200);
    display.setSpeed(speed);
  }
  lastA = a;

  // Button -> next message (200 ms debounce)
  int button = digitalRead(BUTTON_PIN);
  if (button == LOW && lastButton == HIGH && millis() - lastPress > 200) {
    current = (current + 1) % 3;
    display.displayScroll(messages[current], PA_LEFT, PA_SCROLL_LEFT, speed);
    lastPress = millis();
  }
  lastButton = button;

  if (display.displayAnimate()) display.displayReset(); // loop the scroll
}
```

`displayAnimate()` never blocks, so the loop keeps reading inputs while text scrolls. On an ESP32, `analogRead` returns 0-4095, so change 1023 to 4095.

Same idea in MicroPython on a Pico, using the `max7219.py` driver from Coding it. A pot sets brightness and a button swaps the text:

```python
from machine import Pin, SPI, ADC
import time
import max7219

spi = SPI(0, baudrate=10_000_000, polarity=0, phase=0, sck=Pin(18), mosi=Pin(19))
display = max7219.Matrix8x8(spi, Pin(17), 4)
pot = ADC(26)                          # pot wiper on GP26
button = Pin(15, Pin.IN, Pin.PULL_UP)  # button from GP15 to GND

messages = ["HELLO", "NICE", ":)"]
current = 0
x = 32                                 # start off the right edge

while True:
    display.brightness(pot.read_u16() >> 12)  # 0-65535 -> 0-15
    if button.value() == 0:
        current = (current + 1) % len(messages)
        x = 32
        time.sleep_ms(200)                     # simple debounce
    text = messages[current]
    display.fill(0)
    display.text(text, x, 0, 1)
    display.show()
    x -= 1
    if x < -len(text) * 8:                     # fully scrolled off
        x = 32
    time.sleep_ms(40)                          # lower = faster scroll
```

The loop draws the text one pixel further left each pass, which is what makes it scroll. `read_u16() >> 12` squashes the pot's 0-65535 reading into the 0-15 brightness range. Add your own words to `messages`, or change 40 to speed things up.

For auto-brightness, swap the pot for the BH1750 using the [claws/BH1750](https://github.com/claws/BH1750) library:

```cpp
#include <Wire.h>
#include <BH1750.h>
BH1750 lightMeter;

// in setup():
Wire.begin();
lightMeter.begin();

// in loop(), instead of the pot code:
float lux = lightMeter.readLightLevel();
display.setIntensity(lux < 10 ? 0 : (lux < 300 ? 4 : 12)); // dim at night
```

The sensor reports light in lux, so the display dims in a dark room and brightens in a bright one. The 10 and 300 thresholds set where it switches between levels.

For lots of buttons, CircuitPython's built-in `keypad` module scans a key matrix for you (pins here are Pico GP numbers):

```python
import board
import keypad

keys = keypad.KeyMatrix(
    row_pins=(board.GP0, board.GP1, board.GP2),
    column_pins=(board.GP3, board.GP4, board.GP5),
    columns_to_anodes=True,  # True if your diodes are COL2ROW
)

while True:
    event = keys.events.get()
    if event and event.pressed:
        print("Key", event.key_number)  # 0-8: use it to pick a picture
```

A 3x3 grid gives 9 keys on 6 pins. Each press arrives as an event with a key number, so you could map each key to a different face on the matrix.

## Coding it

**Arduino:** use [MD_Parola](https://github.com/MajicDesigns/MD_Parola) for text and effects. It needs [MD_MAX72XX](https://github.com/MajicDesigns/MD_MAX72XX), which handles the hardware. Install both from the Library Manager. For raw pixel control on a single module, [LedControl](https://github.com/wayoda/LedControl) is simpler.

**MicroPython:** use [mcauser/micropython-max7219](https://github.com/mcauser/micropython-max7219). It's one file, `max7219.py`, that goes on your board alongside your code once it's built.

**CircuitPython:** use [Adafruit_CircuitPython_MAX7219](https://docs.circuitpython.org/projects/max7219/en/latest/). For a 4-in-1, create `matrices.CustomMatrix(spi, cs, 32, 8)`. Its `text()` needs the `font5x8.bin` file on your CIRCUITPY drive, and `rotation=` sets which way up the picture is drawn.

Hello world in Arduino C++:

```cpp
#include <MD_Parola.h>
#include <MD_MAX72xx.h>
#include <SPI.h>

#define HARDWARE_TYPE MD_MAX72XX::FC16_HW // must match your module, see below
#define MAX_DEVICES 4                     // 1 for a single module
#define CS_PIN D7                         // XIAO RP2040; DIN and CLK use the hardware SPI pins

MD_Parola display = MD_Parola(HARDWARE_TYPE, CS_PIN, MAX_DEVICES);

void setup() {
  display.begin();
  display.setIntensity(2);                // 0-15, keep it low on USB
  display.displayScroll("Hello, world!", PA_LEFT, PA_SCROLL_LEFT, 50);
}

void loop() {
  if (display.displayAnimate()) display.displayReset(); // restart when done
}
```

This scrolls "Hello, world!" forever. The 50 is milliseconds per frame. Change `PA_SCROLL_LEFT` to `PA_SCROLL_UP` or `PA_SCROLL_DOWN` for other effects.

The modules differ only in how the LEDs are wired to the chip, so `HARDWARE_TYPE` has to match yours, or letters come out mirrored, upside down or scrambled. The FC-16 4-in-1 in the parts table is `FC16_HW`; other modules use `GENERIC_HW`, `PAROLA_HW` or `ICSTATION_HW`.

To draw your own 8x8 picture, use LedControl, which sets one row (8 LEDs) per byte:

```cpp
#include <LedControl.h>

LedControl lc = LedControl(D10, D8, D7, 1); // DIN, CLK, CS, number of modules
byte smile[8] = { 0x3C, 0x42, 0xA5, 0x81, 0xA5, 0x99, 0x42, 0x3C };

void setup() {
  lc.shutdown(0, false);  // module 0 starts asleep, wake it up
  lc.setIntensity(0, 4);  // 0-15
  lc.clearDisplay(0);
  for (int row = 0; row < 8; row++) lc.setRow(0, row, smile[row]);
}

void loop() {}
```

Each hex byte is one row, and each 1 bit is a lit LED. Sketch your face on graph paper and convert each row to binary.

Hello world in MicroPython (Pico):

```python
from machine import Pin, SPI
import max7219

# SPI0: CLK on GP18, DIN on GP19. CS on GP17.
spi = SPI(0, baudrate=10_000_000, polarity=0, phase=0, sck=Pin(18), mosi=Pin(19))
display = max7219.Matrix8x8(spi, Pin(17), 4)  # 4 modules chained

display.brightness(2)        # 0-15
display.fill(0)              # clear
display.text("HI!", 0, 0, 1) # 8x8 font, x=0, y=0, LEDs on
display.show()               # nothing changes until show()
```

The driver uses MicroPython's `framebuf`, so `pixel`, `line`, `rect` and `scroll` work too. Set the last number in `Matrix8x8` to your module count. On a XIAO RP2040, use `sck=Pin(2), mosi=Pin(3)` and CS `Pin(1)`.

**The other matrix types:**

- **HT16K33 backpack:** Arduino uses [Adafruit_LED_Backpack](https://github.com/adafruit/Adafruit_LED_Backpack) with Adafruit GFX (`matrix.begin(0x70)`). CircuitPython uses [adafruit_ht16k33](https://docs.circuitpython.org/projects/ht16k33/en/latest/): `matrix[0, 0] = 1`.
- **NeoPixel matrix:** read the NeoPixel tab first. Then use Adafruit_NeoMatrix and tell it your layout with flags like `NEO_MATRIX_TOP + NEO_MATRIX_LEFT + NEO_MATRIX_ROWS + NEO_MATRIX_ZIGZAG`.
- **HUB75 panel:** these have no built-in brightness control, so your board must refresh them nonstop. Use [Adafruit Protomatter](https://learn.adafruit.com/adafruit-protomatter-rgb-matrix-library) on RP2040 or ESP32 (`rgbmatrix` in CircuitPython). **Always use 5 V**, and budget up to 4 A for a 32x32 at full white.

## Make it yours

Use these as starting points for a design of your own.

- **Name badge.** A 4-in-1 that scrolls your name, with a button to cycle messages. Teaches chaining and power budgets on battery.
- **Robot eyes.** Two single modules that blink and glance around. Teaches drawing bitmaps by hand. The [MD_MAX72XX examples](https://github.com/MajicDesigns/MD_MAX72XX/tree/main/examples) include RobotEyes to learn from.
- **Pong on 8x8.** Two encoders as paddles. Teaches game loops and reading inputs without blocking. There's a SimplePong example in the same folder.
- **Night-aware clock.** Time on a 4-in-1 that dims itself with a BH1750. Teaches I2C sensors and mapping values.
- **Phone message board.** Send text over Bluetooth and scroll it, like [HowToMechatronics' build](https://howtomechatronics.com/tutorials/arduino/8x8-led-matrix-max7219-tutorial-scrolling-text-android-control-via-bluetooth/). Teaches serial input and string buffers.
- **Sound meter.** Bars that bounce with a microphone. Teaches analog sampling and scaling to 8 rows.

If you get stuck, ask for help.

## Sources

- [MAX7219/MAX7221 datasheet (Adafruit mirror)](https://cdn-shop.adafruit.com/datasheets/MAX7219.pdf)
- [MajicDesigns/MD_Parola](https://github.com/MajicDesigns/MD_Parola)
- [MD_Parola documentation](https://majicdesigns.github.io/MD_Parola/)
- [MD_Parola HelloWorld example](https://raw.githubusercontent.com/MajicDesigns/MD_Parola/main/examples/Parola_HelloWorld/Parola_HelloWorld.ino)
- [MD_Parola.h source](https://raw.githubusercontent.com/MajicDesigns/MD_Parola/main/src/MD_Parola.h)
- [MD_Parola.cpp source](https://raw.githubusercontent.com/MajicDesigns/MD_Parola/main/src/MD_Parola.cpp)
- [MajicDesigns/MD_MAX72XX](https://github.com/MajicDesigns/MD_MAX72XX)
- [MD_MAX72XX hardware page](https://majicdesigns.github.io/MD_MAX72XX/page_hardware.html)
- [MD_MAX72xx.h source](https://raw.githubusercontent.com/MajicDesigns/MD_MAX72XX/main/src/MD_MAX72xx.h)
- [MD_MAX72XX examples](https://github.com/MajicDesigns/MD_MAX72XX/tree/main/examples)
- [wayoda/LedControl](https://github.com/wayoda/LedControl)
- [LedControl.h source](https://raw.githubusercontent.com/wayoda/LedControl/master/src/LedControl.h)
- [mcauser/micropython-max7219](https://github.com/mcauser/micropython-max7219)
- [max7219.py source](https://raw.githubusercontent.com/mcauser/micropython-max7219/master/max7219.py)
- [Adafruit CircuitPython MAX7219 docs](https://docs.circuitpython.org/projects/max7219/en/latest/)
- [Adafruit CircuitPython MAX7219 simpletest](https://raw.githubusercontent.com/adafruit/Adafruit_CircuitPython_MAX7219/main/examples/max7219_simpletest.py)
- [Adafruit CircuitPython MAX7219 matrices.py](https://raw.githubusercontent.com/adafruit/Adafruit_CircuitPython_MAX7219/main/adafruit_max7219/matrices.py)
- [HowToMechatronics: 8x8 LED Matrix MAX7219 tutorial](https://howtomechatronics.com/tutorials/arduino/8x8-led-matrix-max7219-tutorial-scrolling-text-android-control-via-bluetooth/)
- [ShillehTek: Arduino MAX7219 scrolling LED marquee](https://shillehtek.com/blogs/news/arduino-max7219-dot-matrix-scrolling-led-marquee)
- [Instructables: Arduino 8x8 LED Matrix](https://www.instructables.com/Arduino-8x8-LED-Matrix/)
- [Instructables: Practical Guide to LEDs 4 (Matrix & Multiplexing)](https://www.instructables.com/Practical-Guide-to-LEDs-4-Matrix-Multiplexing/)
- [Adafruit: 32x16 and 32x32 RGB LED Matrix](https://learn.adafruit.com/32x16-32x32-rgb-led-matrix?view=all)
- [Adafruit: RGB matrix powering](https://learn.adafruit.com/32x16-32x32-rgb-led-matrix/powering)
- [Adafruit: RGB matrix wiring](https://learn.adafruit.com/32x16-32x32-rgb-led-matrix/new-wiring)
- [Adafruit Protomatter RGB Matrix Library](https://learn.adafruit.com/adafruit-protomatter-rgb-matrix-library?view=all)
- [Adafruit Protomatter: Arduino library](https://learn.adafruit.com/adafruit-protomatter-rgb-matrix-library/arduino-library)
- [Adafruit NeoPixel Uberguide: NeoPixel matrices](https://learn.adafruit.com/adafruit-neopixel-uberguide/neopixel-matrices?view=all)
- [Adafruit NeoPixel Uberguide: NeoMatrix library](https://learn.adafruit.com/adafruit-neopixel-uberguide/neomatrix-library)
- [Adafruit LED Backpack: 1.2" 8x8 matrix](https://learn.adafruit.com/adafruit-led-backpack/1-2-8x8-matrix)
- [Adafruit_LED_Backpack matrix8x8 example](https://raw.githubusercontent.com/adafruit/Adafruit_LED_Backpack/master/examples/matrix8x8/matrix8x8.ino)
- [Adafruit CircuitPython HT16K33 docs](https://docs.circuitpython.org/projects/ht16k33/en/latest/)
- [claws/BH1750](https://github.com/claws/BH1750)
- [Seeed XIAO RP2040 wiki](https://wiki.seeedstudio.com/XIAO-RP2040/)
- [Seeed-Studio/OPL_Kicad_Library](https://github.com/Seeed-Studio/OPL_Kicad_Library)
- [KaiPereira/midi-keypad](https://github.com/KaiPereira/midi-keypad)
- [Starbie Week 1 Guide](https://github.com/SharKingStudios/Starbie/blob/main/Week%201%20Guide.md)
- [Hack Club Blueprint: MIDI starter project](https://blueprint.hackclub.com/starter-projects/midi)
- [Hack Club Pathfinder](https://pathfinder.hackclub.com/)
- [Hack Club Hermes](https://hermes.hackclub.com/)
