# TFT screens

Small SPI colour screens take a project from "it blinks" to "it has a real interface". Use them for menus, graphs, game screens and small dashboards. The cheap ones cost a few dollars.

## What it is

A TFT is a tiny colour LCD, like a phone screen from 2008. Hobby modules come on a small board with pins along one edge, and you talk to them over SPI using 5-6 wires.

You'll see three driver chips over and over:

- **ST7735:** 1.8" screens, 128x160 pixels. Cheap and everywhere.
- **ST7789:** 1.3-2.4" screens, 240x240 or 240x320, often IPS (colours look right from any angle).
- **ILI9341:** 2.4-2.8" screens, 240x320, often with a resistive touch layer on top.

![A 2.2 inch ILI9340C TFT breakout on a breadboard showing a clock and temperature](/art/starters/displays/tft-1.jpg)
*Photo: AaltoFablab, CC BY 2.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Arduino_2,2%22_TFT_display_(20679950896).jpg)*

Look at the pin labels on the left of that board: BL, SCK, MISO, MOSI, CS, RST, D/C, VIN, GND. That's the whole interface.

## How it works

Every pixel is three tiny liquid-crystal shutters: red, green and blue. A thin-film transistor (the "TFT") sits behind each one and sets how much light gets through.

![Macro photo of an LCD showing rows of red, green and blue subpixels](/art/starters/displays/tft-2.jpg)
*Photo: Politelyinsulting, CC BY-SA 3.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:LCD_subpixels.jpg)*

The crystals don't make light. A white LED backlight shines through from behind. On small TFTs it's usually 1-4 LEDs along one edge, spread out by a plastic light guide.

![An LCD TV with its pixel layer removed, showing LEDs glowing along the top and bottom edges](/art/starters/displays/tft-3.jpg)
*Photo: RobertKuhlmann, CC BY-SA 3.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:LCD-Hintergrundbeleuchtung_mit_Edge-LEDs.JPG)*

That's a TV, but your 2" screen is built the same way. A 2.8" ILI9341 panel has 4 white LEDs at about 3.2 V that share up to 60 mA. **Never power the backlight straight from a GPIO pin unless the module has a transistor for it.**

The driver chip has its own memory holding every pixel. You send a pixel once and it stays on screen until you change it. That's why even a small microcontroller can drive one.

### The SPI pins

| Pin | Also labelled | What it does |
|---|---|---|
| SCK | SCL, CLK | Clock, one pulse per bit |
| MOSI | SDA, DIN, SDI | Data from your board to the screen |
| CS | TCS | Chip select. Low means "I'm talking to you" |
| DC | D/C, A0, RS | Low = this byte is a command, high = pixel data |
| RST | RES, RESET | Pull low to restart the driver chip |
| BL | LED, LITE | Backlight on/off, or brightness with PWM |
| MISO | SDO | Data back from the screen (usually unused) |

On a TFT, "SDA" means SPI data, not I2C.

### Colours, speed and RAM

Colours are 16-bit "RGB565": 5 bits red, 6 bits green, 5 bits blue. So pure red is `0xF800` and white is `0xFFFF`.

A full 240x320 screen is 153,600 bytes. At a 40 MHz SPI clock that takes about 31 ms, so 30 full redraws per second at best. Redraw only what changed.

Your board's RAM decides what you can do:

- **Raspberry Pi Pico:** 264 KB. A full-screen buffer fits, which gives smooth animation.
- **ESP32:** over 200 KB free, so a full-screen buffer fits too. Wi-Fi also lets you show live data.

### 3.3 V logic

These chips run on 3.3 V. Many modules have a regulator so VCC can take 5 V, but the **data pins are often still 3.3 V only**. Adafruit's breakouts include a level shifter; most cheap red boards don't.

## Using it in a project

### Parts for your design

These are the parts that go in your schematic and BOM. They're bought with your funding once your design is approved.

| Part | What it's for | Driver chip / module | Notes |
|---|---|---|---|
| 1.8" 128x160 TFT | First screen, small gadgets | ST7735 / ST7735R | Cheapest. Some have a microSD slot. Firmware needs the matching "black/green/red tab" setting |
| 1.3" or 1.54" 240x240 IPS | Watches, badges, sharp graphics | ST7789 | Some have no CS pin. IPS panels need colour inversion turned on in your firmware |
| 2.0" 240x320 IPS | Bigger readable UI | ST7789 | Good middle ground |
| 2.4-2.8" 240x320 touch | Touch menus, control panels | ILI9341 + XPT2046 touch chip | Draws about 55-60 mA in total. 3.3 V logic only |
| Adafruit breakouts (1.8", 1.54", 2.8") | Easy 5 V use | ST7735R / ST7789 / ILI9341 | Built-in level shifter, backlight PWM pin, 18-pin EYESPI flex connector |
| Bare panel with flex cable | Thin custom PCBs | Any of the above | Advanced. Needs an FPC connector and backlight circuit |

### On a PCB

Most of you will put the screen on a custom board with a Pico or XIAO. The easiest route is to **give your PCB a header socket that the module plugs into**. These are the steps in KiCad.

**1. Schematic.** Open the Schematic Editor and press `A` to add symbols.

Connect these pins in your schematic. SCK and MOSI go to your board's hardware SPI pins, which are much faster than other pins. CS, DC and RST can go to any spare GPIO pins, and BL to a PWM pin (or to 3.3 V for always-on).

| Display pin | Raspberry Pi Pico | XIAO RP2040 | ESP32 DevKit |
|---|---|---|---|
| VCC | 3V3 | 3V3 | 3V3 |
| GND | GND | GND | GND |
| SCK | GP18 | D8 | GPIO18 |
| MOSI | GP19 | D10 | GPIO23 |
| CS | GP17 | D7 | GPIO5 |
| DC | GP20 | D6 | GPIO16 |
| RST | GP21 | D3 | GPIO17 |
| BL | GP22 | D2 | GPIO4 |

A 2.8" module draws about 55-60 mA, mostly backlight, so one screen is fine from a Pico or XIAO 3V3 pin.

For touch, some resistive touch layers give you 4 raw pads (X+, X-, Y+, Y-) to read with analog pins. Others add an XPT2046 chip on the same SPI bus, with its own chip-select pin (`T_CS`). TFT_eSPI supports it via `TOUCH_CS`.

- Microcontroller: `MCU_Module:RaspberryPi_Pico` is built into KiCad. XIAO symbols aren't, so use the one from your program's care package (Starbie's has the XIAO ESP32-C3).
- Screen: `Connector_Generic:Conn_01x08` for an 8-pin module. Label each pin with the text tool, **in the exact order shown on your module's listing or datasheet**. Pin orders differ between sellers.
- Press `L` and add net labels (`TFT_SCK`, `TFT_MOSI`, `TFT_CS`, `TFT_DC`, `TFT_RST`, `TFT_BL`) on both the connector and the microcontroller. Same name means connected.
- Press `P` for power: `+3V3` on VCC, `GND` on GND. Power points up, ground points down.
- Add a 10 µF and a 100 nF capacitor (`Device:C_Small`) between +3V3 and GND right next to the connector. They soak up current spikes when the screen redraws.
- Backlight: if your module's BL pin has a transistor (Adafruit's do), connect it straight to `TFT_BL`. If not, add a `Transistor_FET:AO3401A` P-channel MOSFET. Source goes to +3V3, drain to the BL pin, and gate to `TFT_BL` through a 100 Ω resistor. Add 10 kΩ from gate to +3V3 so the light stays off during boot. This flips the logic, so LOW turns it on.

**2. Footprints.** Open the footprint assignment tool and pick:

| Part | Footprint |
|---|---|
| Module socket | `Connector_PinSocket_2.54mm:PinSocket_1x08_P2.54mm_Vertical` |
| Pico | `Module:RaspberryPi_Pico_Common_THT` |
| Capacitors, resistors | `Capacitor_SMD:C_0805_2012Metric`, `Resistor_SMD:R_0805_2012Metric` |
| MOSFET | `Package_TO_SOT_SMD:SOT-23` |
| Mounting holes | `MountingHole:MountingHole_2.7mm_M2.5` or `MountingHole_3.2mm_M3` (measure your module's holes) |

**3. Layout.** Click "Update PCB from Schematic" and draw your outline on Edge.Cuts.

- Find the module's drawing and put the socket and mounting holes where the module will actually sit. Mounting holes for standoffs mean it can't wobble.
- Keep the area under the screen clear of tall parts.
- Place the Pico close to the socket. Keep SPI traces short, ideally a few centimetres.
- Use 0.25 mm traces for signals and 0.5 mm for 3V3, GND and backlight.
- Pour a GND fill on the back layer. A solid ground under the SPI lines keeps the picture clean.
- Avoid sharp 90° corners.

**4. Check before you submit.** Run DRC (Inspect → Design Rules Checker) and fix every error; most warnings are fine to ignore. Open the 3D viewer to check the module lines up.

**Bare panels (advanced).** A bare panel's flex cable slides into an FPC connector. Adafruit's EYESPI standard uses 18 pins at 0.5 mm pitch, and KiCad has a matching footprint: `Connector_FFC-FPC:Hirose_FH12-18S-0.5SH_1x18-1MP_P0.50mm_Horizontal`. Use the panel's datasheet, not the driver chip's, and triple-check pin count and pitch.

![A white flex-cable connector on a green circuit board](/art/starters/displays/tft-4.jpg)
*Photo: Evan-Amos, Public domain, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Nintendo-Game-Boy-Color-Motherboard-screen_conector_flex_(cropped).jpg)*

Bare panels also expose the backlight LEDs directly (LEDA anode, LEDK cathodes). Feed LEDA from 5 V through a resistor and switch LEDK to ground with a `Transistor_FET:AO3400A` N-channel MOSFET. For about 40 mA: (5 V - 3.2 V) / 0.04 A = 45 Ω, so use 47 Ω.

## Adding inputs

Inputs turn your screen into an interface. This table shows how the usual parts connect to a Pico. On a XIAO you only have 5 pins left after the screen, so pick a few.

| Input | KiCad symbol / footprint | Connections | Pico pins |
|---|---|---|---|
| Button (tactile or low-profile) | `Switch:SW_Push` / `Button_Switch_THT:SW_PUSH_6mm` | One side to GPIO, other to GND. Use the internal pull-up | GP12 |
| EC11 rotary encoder | `Device:RotaryEncoder_Switch` / `Rotary_Encoder:RotaryEncoder_Alps_EC11E-Switch_Vertical_H20mm` | A and B to GPIOs, C to GND. Push switch to GPIO and GND | A GP14, B GP15, switch GP13 |
| Slide pot | `Device:R_Potentiometer_MountingPin` / `Potentiometer_THT:Potentiometer_Bourns_PTA4543_Single_Slide` | Ends to GND and 3V3, wiper to an ADC pin. Ground the mounting pins | GP26 |
| I2C sensor breakout (e.g. AHT20 temperature) | `Connector_Generic:Conn_01x04` | VCC, GND, SDA, SCL (match the breakout's order) | SDA GP4, SCL GP5 |

For more than 4-5 buttons, use a key matrix with a 1N4148 diode per key, like the Blueprint MIDI keypad. Most sensor breakouts already have I2C pull-up resistors; a bare sensor chip needs 4.7 kΩ pull-ups on SDA and SCL.

This complete Arduino sketch for a Pico is ready to flash once your board is built. The encoder changes a number, the button changes its colour, and the slide pot sets the backlight brightness.

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_ST7789.h>
#include <SPI.h>

#define TFT_CS  17
#define TFT_DC  20
#define TFT_RST 21
#define TFT_BL  22
#define ENC_A   14
#define ENC_B   15
#define BUTTON  12
#define SLIDER  26

Adafruit_ST7789 tft = Adafruit_ST7789(TFT_CS, TFT_DC, TFT_RST);

volatile int count = 0;            // changed inside the interrupt
const uint16_t colours[] = {ST77XX_WHITE, ST77XX_RED, ST77XX_GREEN, ST77XX_CYAN};
int colourIndex = 0;
int shownCount = -9999;            // forces the first draw
bool lastButton = HIGH;

// Runs every time encoder pin A changes
void onTurn() {
  if (digitalRead(ENC_A) == digitalRead(ENC_B)) count--;
  else count++;
}

void setup() {
  pinMode(ENC_A, INPUT_PULLUP);
  pinMode(ENC_B, INPUT_PULLUP);
  pinMode(BUTTON, INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(ENC_A), onTurn, CHANGE);

  tft.init(240, 240);              // your panel's width and height
  tft.fillScreen(ST77XX_BLACK);
  tft.setTextSize(4);
}

void loop() {
  // Slide pot: 0-1023 in, 0-255 brightness out
  analogWrite(TFT_BL, analogRead(SLIDER) / 4);

  // Button: on each new press, pick the next colour
  bool button = digitalRead(BUTTON);
  if (button == LOW && lastButton == HIGH) {
    colourIndex = (colourIndex + 1) % 4;
    shownCount = -9999;            // redraw in the new colour
  }
  lastButton = button;

  // Encoder: only redraw when the number changes
  int now = count;
  if (now != shownCount) {
    tft.setCursor(30, 100);
    tft.setTextColor(colours[colourIndex], ST77XX_BLACK);  // black background wipes old digits
    tft.print(now);
    tft.print("    ");
    shownCount = now;
  }
  delay(10);                       // also a cheap debounce
}
```

It only redraws when something changes, so there's no flicker.

Blueprint's firmware uses CircuitPython, where the built-in `rotaryio` module handles the encoder for you. This snippet reads an AHT20 over I2C (it uses `adafruit_ahtx0` from the library bundle):

```python
import board, busio, rotaryio
import adafruit_ahtx0

encoder = rotaryio.IncrementalEncoder(board.GP14, board.GP15)
i2c = busio.I2C(board.GP5, board.GP4)       # SCL, SDA
sensor = adafruit_ahtx0.AHTx0(i2c)

print(encoder.position)                     # goes up and down as you turn
print(sensor.temperature)                   # degrees C
```

To show it on screen, set `text.text = f"{sensor.temperature:.1f} C"` in your loop, using the label from the CircuitPython example below.

## Coding it

Write your firmware now, and flash it once your board is built.

| Language | Library | Best for |
|---|---|---|
| Arduino | [Adafruit_GFX](https://learn.adafruit.com/adafruit-gfx-graphics-library/overview) + [Adafruit ST7735/ST7789](https://github.com/adafruit/Adafruit-ST7735-Library) or [Adafruit_ILI9341](https://github.com/adafruit/Adafruit_ILI9341) | Beginners, any board |
| Arduino | [TFT_eSPI](https://github.com/Bodmer/TFT_eSPI) | Speed on ESP32 and RP2040 |
| MicroPython | [st7789py_mpy](https://github.com/russhughes/st7789py_mpy) (ST7789) or [micropython-ili9341](https://github.com/rdagger/micropython-ili9341) | Pico and ESP32 |
| CircuitPython | [adafruit_st7789](https://github.com/adafruit/Adafruit_CircuitPython_ST7789) + displayio | Easiest Python route |

### Arduino: Adafruit_GFX

Install "Adafruit ST7735 and ST7789 Library" from the Library Manager; it pulls in Adafruit GFX too.

```cpp
#include <Adafruit_GFX.h>    // shapes and text
#include <Adafruit_ST7789.h> // the ST7789 driver
#include <SPI.h>

#define TFT_CS  17
#define TFT_DC  20
#define TFT_RST 21
#define TFT_BL  22

Adafruit_ST7789 tft = Adafruit_ST7789(TFT_CS, TFT_DC, TFT_RST);

void setup() {
  pinMode(TFT_BL, OUTPUT);
  digitalWrite(TFT_BL, HIGH);           // backlight on
  tft.init(240, 240);                   // or (240, 320), (135, 240)...
  tft.fillScreen(ST77XX_BLACK);
  tft.setCursor(10, 10);                // (0, 0) is the top-left corner
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(3);
  tft.println("Hello, world!");
  tft.fillCircle(120, 150, 40, ST77XX_MAGENTA);
}

void loop() {}
```

This uses hardware SPI, so you only list CS, DC and RST. On an ILI9341, swap in `Adafruit_ILI9341.h`, `Adafruit_ILI9341 tft(...)` and `tft.begin()`. On an ST7735, use `Adafruit_ST7735` with `tft.initR(INITR_BLACKTAB)`. A 240x240 board with no CS pin needs `tft.init(240, 240, SPI_MODE3)` and `-1` for CS.

### Arduino: TFT_eSPI

TFT_eSPI is much faster, but pins are not set in your sketch. They live in a setup file inside the library folder (`Documents/Arduino/libraries/TFT_eSPI`):

- `User_Setup.h`: the default setup file. Edit it directly, or ignore it.
- `User_Setup_Select.h`: a list of `#include` lines. Exactly one must be uncommented, and that file is the setup used. Out of the box it's `#include <User_Setup.h>`.

The cleanest approach is to copy a close match from `User_Setups/` (like `Setup24_ST7789.h`), edit the pins, and point `User_Setup_Select.h` at it. **Updating the library wipes your changes**, so keep a copy in `Arduino/libraries/TFT_eSPI_Setups/` and include it as `#include <../TFT_eSPI_Setups/my_setup.h>`.

A setup file for the Pico pins above:

```cpp
#define ST7789_DRIVER
#define TFT_WIDTH  240
#define TFT_HEIGHT 240
#define TFT_MISO 16          // not wired; the library's RP2040 setups define one
#define TFT_MOSI 19
#define TFT_SCLK 18
#define TFT_CS   17
#define TFT_DC   20
#define TFT_RST  21
#define TFT_BL   22
#define TFT_BACKLIGHT_ON HIGH
#define LOAD_GLCD            // small built-in font
#define LOAD_FONT4           // medium font
#define SPI_FREQUENCY 40000000
```

And the sketch:

```cpp
#include <TFT_eSPI.h>
TFT_eSPI tft = TFT_eSPI();     // settings come from your setup file

void setup() {
  tft.init();
  tft.fillScreen(TFT_BLACK);
  tft.setTextColor(TFT_WHITE, TFT_BLACK);
  tft.drawString("Hello, world!", 10, 10, 4);  // text, x, y, font number
}

void loop() {}
```

If your panel needs a different colour order or inversion, uncomment one `TFT_RGB_ORDER` or `TFT_INVERSION_ON/OFF` line in your setup file. ST7735 panels also need the matching `ST7735_..TAB` option.

### MicroPython: st7789py

Your firmware needs `lib/st7789py.py` and a font such as `romfonts/vga1_bold_16x32.py` from the repo. Once your board is built, copy them onto the Pico with Thonny.

```python
from machine import Pin, SPI
import st7789py as st7789
import vga1_bold_16x32 as font

spi = SPI(0, baudrate=40_000_000, sck=Pin(18), mosi=Pin(19))
tft = st7789.ST7789(
    spi, 240, 240,                 # width, height
    reset=Pin(21, Pin.OUT),
    cs=Pin(17, Pin.OUT),
    dc=Pin(20, Pin.OUT),
    backlight=Pin(22, Pin.OUT),    # driver turns it on for you
    rotation=0,
)

tft.fill(st7789.BLACK)
tft.text(font, "Hello!", 10, 10, st7789.WHITE, st7789.BLACK)
tft.fill_rect(20, 80, 200, 60, st7789.color565(255, 120, 0))  # orange box
```

The driver supports 240x320, 240x240, 135x240 and 128x128 panels. If your panel needs a different colour order or inversion, set `color_order=st7789.RGB` or `tft.inversion_mode(False)`. For PWM brightness, leave out `backlight=` and use `machine.PWM(Pin(22))` instead. No-CS boards: set `cs=None` and add `polarity=1, phase=1` to `SPI(...)`.

### CircuitPython: displayio

Your firmware needs `adafruit_st7789` and `adafruit_display_text` from the library bundle. Once your board is built, copy them to `CIRCUITPY/lib`.

```python
import board, busio, displayio, terminalio
from fourwire import FourWire
from adafruit_display_text import label
from adafruit_st7789 import ST7789

displayio.release_displays()                 # free the pins from last run
spi = busio.SPI(board.GP18, board.GP19)      # SCK, MOSI
bus = FourWire(spi, command=board.GP20, chip_select=board.GP17, reset=board.GP21)
display = ST7789(bus, width=240, height=240, rowstart=80, backlight_pin=board.GP22)

screen = displayio.Group()
display.root_group = screen
text = label.Label(terminalio.FONT, text="Hello, world!", color=0xFFFF00, scale=2, x=20, y=120)
screen.append(text)

while True:
    pass
```

displayio redraws for you whenever you change something, like `text.text = "New!"`. `rowstart=80` comes from Adafruit's 240x240 example. For an ST7735R, use `adafruit_st7735r` with `width=128, height=160`.

## Make it yours

Use these as starting points and design your own thing. A project copied from a guide isn't yours to submit.

1. **Weather station:** show temperature and humidity from an I2C sensor as a live graph. Teaches redrawing only what changed.
2. **Encoder menu:** scroll a menu with an EC11 and click to choose. Teaches state machines and UI layout.
3. **Mini game:** Pong or Snake with buttons. Teaches game loops and frame timing (try a TFT_eSPI sprite to stop flicker).
4. **MIDI controller with a screen:** show which note or knob you're changing, like the [Blueprint MIDI starter](https://blueprint.hackclub.com/starter-projects/midi). Teaches combining many inputs.
5. **Desk dashboard:** an ESP32 pulls the time, weather or your GitHub stats over Wi-Fi. Teaches APIs and text layout.
6. **Touch soundboard:** an ILI9341 touchscreen with big buttons. Teaches touch calibration and hit-testing.

If you get stuck, post about it in #half-life.

## Sources

- [A-Z Guide to Interfacing TFT LCD Displays w/ Arduino (Instructables)](https://www.instructables.com/Absolute-Beginners-Guide-to-TFT-LCD-Displays-by-Ar/)
- [Beginner Arduino: Using a 1.8 Inch TFT Display (Instructables)](https://www.instructables.com/Beginner-Arduino-Using-a-18-Inch-TFT-Display/)
- [Select Color Display for ESP32 (Instructables)](https://www.instructables.com/Select-Color-Display-for-ESP32/)
- [Adafruit GFX Graphics Library: Overview](https://learn.adafruit.com/adafruit-gfx-graphics-library/overview)
- [Adafruit GFX: Coordinate System and Units](https://learn.adafruit.com/adafruit-gfx-graphics-library/coordinate-system-and-units)
- [Adafruit 1.8" TFT Display: Overview](https://learn.adafruit.com/1-8-tft-display/overview)
- [Adafruit 1.8" TFT Display: Breakout Pinouts](https://learn.adafruit.com/1-8-tft-display/breakout-pinouts)
- [Adafruit 1.3" and 1.54" 240x240 TFT: Pinouts](https://learn.adafruit.com/adafruit-1-3-and-1-54-240-x-240-wide-angle-tft-lcd-displays/pinouts)
- [Adafruit 2.8" and 3.2" TFT Touchscreen Breakout: Pinouts](https://learn.adafruit.com/adafruit-2-8-and-3-2-color-tft-touchscreen-breakout-v2/pinouts)
- [Adafruit ST7735 and ST7789 Library (GitHub)](https://github.com/adafruit/Adafruit-ST7735-Library)
- [Adafruit_ILI9341 (GitHub)](https://github.com/adafruit/Adafruit_ILI9341)
- [TFT_eSPI (GitHub)](https://github.com/Bodmer/TFT_eSPI)
- [TFT_eSPI User_Setup_Select.h](https://github.com/Bodmer/TFT_eSPI/blob/master/User_Setup_Select.h)
- [TFT_eSPI User_Setup.h](https://github.com/Bodmer/TFT_eSPI/blob/master/User_Setup.h)
- [TFT_eSPI Setup24_ST7789.h](https://github.com/Bodmer/TFT_eSPI/blob/master/User_Setups/Setup24_ST7789.h)
- [TFT_eSPI Setup60_RP2040_ILI9341.h](https://github.com/Bodmer/TFT_eSPI/blob/master/User_Setups/Setup60_RP2040_ILI9341.h)
- [st7789py_mpy (GitHub)](https://github.com/russhughes/st7789py_mpy)
- [micropython-ili9341 (GitHub)](https://github.com/rdagger/micropython-ili9341)
- [Adafruit_CircuitPython_ST7789 (GitHub)](https://github.com/adafruit/Adafruit_CircuitPython_ST7789)
- [Mischianti: TFT ILI9341 articles](https://mischianti.org/category/electronic/display-en/tft-ili9341/)
- [Mischianti: Complete Guide: ILI9341 with TFT_eSPI](https://mischianti.org/complete-guide-using-an-ili9341-display-with-the-tft_espi-library/)
- [Hackaday: Displays We Love Hacking: SPI and I2C](https://hackaday.com/2023/12/21/displays-we-love-hacking-spi-and-i2c/)
- [Prototype Supplies: 2.8" 240x320 SPI TFT with touchscreen](https://protosupplies.com/product/tft-lcd-2-8-240x320-rgb-spi-display-with-touchscreen/)
- [4D Systems: 4DLCD-28QA datasheet](https://resources.4dsystems.com.au/datasheets/4dlcd/4DLCD-28QA/)
- [Seeed Studio: XIAO RP2040 wiki](https://wiki.seeedstudio.com/XIAO-RP2040/)
- [Seeed Studio: XIAO ESP32-C3 wiki](https://wiki.seeedstudio.com/XIAO_ESP32C3_Getting_Started/)
- [KiCad symbol library (GitLab)](https://gitlab.com/kicad/libraries/kicad-symbols)
- [KiCad footprint library (GitLab)](https://gitlab.com/kicad/libraries/kicad-footprints)
- [Blueprint: MIDI starter project (Hack Club)](https://blueprint.hackclub.com/starter-projects/midi)
- [Kai Pereira: midi-keypad KiCad walkthrough](https://github.com/KaiPereira/midi-keypad)
- [Kai Pereira: Hack Club KiCad Libraries](https://github.com/KaiPereira/Hackclub-KiCad-Libraries)
- [Starbie: Week 1 Guide](https://github.com/SharKingStudios/Starbie/blob/main/Week%201%20Guide.md)
- [Pathfinder (Hack Club)](https://pathfinder.hackclub.com/)
- [Hermes (Hack Club)](https://hermes.hackclub.com/)
