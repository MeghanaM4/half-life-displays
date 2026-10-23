# NeoPixels

NeoPixels are full-colour LEDs that you set one by one, all from a single data pin. Use them for glowing rings, light strips, status lights and colour effects.

## What it is

A NeoPixel is a tiny RGB LED with its own driver chip built in. "NeoPixel" is Adafruit's name for them. The chips inside are usually the WS2812B or SK6812. They come as single pixels, rings, sticks, matrices and long strips.

Each pixel can show any of about 16.7 million colours (256 levels each of red, green and blue). Some versions (RGBW) add a fourth, white LED. You can chain hundreds together and still only use one pin on your microcontroller.

![An addressable strip showing a rainbow (top) next to a normal strip that can only show one colour at a time (bottom)](/art/starters/displays/neopixels-1.jpg)
*Photo: DoyleTheCat, CC BY-SA 4.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Addressable_LEDs.jpg)*

## How it works

Every pixel has four pins: 5V (VDD), GND (VSS), data in (DIN) and data out (DOUT). You send a long stream of bits into the first pixel's DIN at 800 kbit/s. It keeps the first 24 bits (8 each for green, red and blue) and passes the rest out of DOUT to the next pixel.

So the colours ripple down the chain like a bucket brigade. When the data line goes quiet for a moment (50-280 µs, depending on the chip), every pixel switches to its new colour at once. That's why your code always ends with a "show" or "write" call.

The timing is very strict, so you never toggle the pin yourself. A library does it for you. Strips have an arrow printed on them: it points from DIN to DOUT, the way the data flows.

![One pixel cut from a WS2812B strip, with +5V, Din and GND pads on the left, an arrow showing data direction, and a small capacitor beside the LED](/art/starters/displays/neopixels-2.jpg)
*Photo: John McMaster, CC BY 4.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Pack1_-_BTF-WS2812B_-_John_McMaster.jpg)*

Power needs the most planning. Each pixel has three LEDs that draw up to about 20 mA each, so that's up to **60 mA per pixel at full white**.

Normal animations average closer to 20 mA per pixel, but plan for the worst case.

| Pixels | Typical (20 mA each) | Full white (60 mA each) |
| --- | --- | --- |
| 8 | 0.16 A | 0.48 A |
| 16 | 0.32 A | 0.96 A |
| 30 | 0.6 A | 1.8 A |
| 60 | 1.2 A | 3.6 A |

The formula is pixels × mA ÷ 1000 = amps. Always pick a supply with headroom. As Adafruit puts it: extra amps are fine, extra volts are not. **Never feed 5 V pixels more than 5 V.**

## Using it in a project

### Parts for your design

These are the parts that go in your schematic and BOM. They're bought with your funding once your design is approved.

![A 16-pixel NeoPixel ring, with 5050-size WS2812 LEDs around a circular board](/art/starters/displays/neopixels-3.jpg)
*Photo: SparkFun Electronics, CC BY 2.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:12664-01a.jpg)*

| Part | What it's for | Driver chip | Notes |
| --- | --- | --- | --- |
| Ring (12, 16 or 24 pixels) | Clocks, dials, knob indicators | WS2812B (5050) | A good first part, with only 3 connections |
| Stick (8 pixels) | Level meters, status bars | WS2812B (5050) | 8 pixels in a straight row |
| Strip (30-144 pixels/m) | Room lighting, long effects | WS2812B or SK6812 | Needs its own 5 V supply past ~8 pixels |
| 8x8 matrix | Tiny pixel art, games | WS2812B | 64 pixels = up to 3.8 A at full white |
| WS2812B-5050 bare LED | Your own PCB | Built in | 5 x 5 mm |
| WS2812B-2020 bare LED | Small, neat PCBs | Built in | 2 x 2 mm, 12 mA per colour |
| SK6812MINI-E | Under-key lighting | Built in | Reverse-mount: shines through a hole in the PCB |
| 74AHCT125 / 74AHCT1G125 | Level shifter for 3.3 V boards | – | Turns 3.3 V data into clean 5 V data |
| 1000 µF, 6.3 V+ capacitor | Bulk cap across strip power | – | Soaks up current spikes |
| 330 Ω resistor | Series resistor on data | – | Anything 300-500 Ω works |

### On a PCB

Most of you will put pixels on a custom KiCad board with a XIAO or a Pico, just like the Hack Club starter projects. These are the steps.

**1. Schematic**

1. Add your microcontroller. For a XIAO, add Seeed's official KiCad symbols and footprints (linked from the XIAO wiki in the sources). Place the symbol.
2. Press A and search `WS2812B-2020` (or `WS2812B` for the 5050 size). The symbols live in KiCad's `LED` library. For under-key lighting, use `SK6812MINI-E`.
3. Wire VDD to a `+5V` power symbol and VSS to `GND`. Keep voltage labels pointing up and ground pointing down so the sheet stays readable.
4. Chain the data: the first pixel's DOUT goes to the second pixel's DIN, and so on. Short net labels (press L) like `LED1`, `LED2` keep long chains tidy.
5. Add a 100 nF capacitor (`Device:C`) from +5V to GND for every pixel. Both the WS2812B and WS2812B-2020 datasheets show one per pixel in their example circuits.
6. Add one bulk capacitor where 5 V enters the LED section. Use 10-22 µF on a USB-powered board, or 500-1000 µF if a big external supply feeds a strip.
7. Add the level shifter: `74AHCT1G125` from the `74xGxx` library (one buffer, SOT-23-5). Wire pin 1 (OE) and pin 3 to GND, and pin 2 (A) to your data GPIO. Pin 5 (VCC) goes to +5V with its own 100 nF cap.
8. Pin 4 (Y) goes through a 330 Ω resistor to the first pixel's DIN.
9. Route the last pixel's DOUT to a pad, so future you can chain more pixels on.
10. Driving an off-board strip? Add a 3-pin connector (`Conn_01x03`) for 5V, data and GND.

**Why the level shifter?** WS2812B pixels at 5 V want a data "high" of at least 0.7 × 5 V = 3.5 V. A 3.3 V pin sits just below that, so it can flicker or fail randomly. The shifter turns 3.3 V data into clean 5 V data. No room for one? Hackaday's trick powers just the first pixel through a diode, so its supply drops to about 4.3 V. Its threshold drops to about 3.0 V, and it passes clean 4.3 V data to the rest.

A note on the 2020 size: the WS2812B-2020-V6 datasheet lists its "high" threshold as 0.55 × VDD, about 2.75 V at 5 V. So 3.3 V data works directly on that version. Older versions and the 5050 need 0.7 × VDD, so keep the shifter unless you're sure.

Connect these pins in your schematic:

| Pixel pin | Pico | XIAO RP2040 / ESP32-C3 |
| --- | --- | --- |
| 5V | VBUS (pin 40) | 5V |
| GND | GND | GND |
| DIN | GP0 → level shifter → 330 Ω → DIN | D6 → level shifter → 330 Ω → DIN |

The board's 5 V USB pin is fine for small pixel counts. A 60-pixel strip on USB makes the board brown out and reset, so bigger strips need their own 5 V supply. If you add one, wire its positive terminal to the pixels' 5V and its negative terminal to the pixels' GND, then **join the supply's GND to your board's GND**. Without a shared ground, the data has nothing to be measured against.

Before moving on, check that your data runs into the first pixel's DIN, not its DOUT. A chain fed from the wrong end stays dark.

**2. Footprints**

Open Assign Footprints. The LED symbols already fill in KiCad's matching footprints:

| Part | KiCad footprint |
| --- | --- |
| WS2812B (5050) | `LED_SMD:LED_WS2812B_PLCC4_5.0x5.0mm_P3.2mm` |
| WS2812B-2020 | `LED_SMD:LED_WS2812B-2020_PLCC4_2.0x2.0mm` |
| SK6812MINI-E | `LED_SMD:LED_SK6812MINI-E_3.2x2.8mm_P1.5mm_ReverseMount` |
| 74AHCT1G125 | `Package_TO_SOT_SMD:SOT-23-5` |
| 100 nF cap, 330 Ω resistor | `Capacitor_SMD:C_0603_1608Metric`, `Resistor_SMD:R_0603_1608Metric` |
| Strip connector | `Connector_JST:JST_XH_B3B-XH-A_1x03_P2.50mm_Vertical` |

**Never swap a 5050 footprint onto a 2020 symbol.** Their pin numbers are different (5050: 1 = VDD, 4 = DIN; 2020: 1 = DOUT, 3 = DIN). Matching the symbol and footprint keeps them right.

**3. Placement and routing**

![Close-up of 5050 WS2812 pixels on a NeoPixel stick, each with a tiny capacitor placed right beside it](/art/starters/displays/neopixels-4.jpg)
*Photo: Daniele Napolitano, CC BY-SA 4.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:WS2812Closeup1.jpg)*

- Put each 100 nF cap right next to its pixel's VDD pin, like the stick above.
- Put the shifter and 330 Ω resistor near the first pixel, with a short data trace.
- Keep the chain in order. Each DOUT → DIN hop should be short and direct.
- For SK6812MINI-E, press F to flip it to the back. Cut a hole under it so it shines through. KiCad's DRC may warn about edge clearance there, which is expected.

Make the power traces wide. All the pixel current flows through the +5V and GND traces. Using the IPC-2221 formula for 1 oz outer copper and a 10 °C rise:

| Current | Minimum trace width |
| --- | --- |
| 0.5 A | 0.15 mm |
| 1 A | 0.3 mm |
| 2 A | 0.8 mm |
| 3 A | 1.4 mm |

In Board Setup → Net Classes, make a `Power` class with 0.8 mm tracks and assign +5V and GND to it. Keep data at the default 0.2-0.25 mm. Then fill the back layer with a GND pour so every pixel has a short path to ground.

If you're powering a long strip through your board, check the connector too. JST-XH contacts are rated 3 A each, so use a separate screw terminal for more than that.

**4. Check before you submit**

1. Run DRC and fix every error. Unconnected-pad errors are never OK to ignore.
2. Open the 3D Viewer (Alt+3) and check each LED's orientation. The polarity corner should match the footprint.
3. Add up your current: 12 × WS2812B-2020 at full white is 12 × 36 mA = 432 mA. That's close to USB's 500 mA, so cap brightness in code.
4. If JLCPCB is assembling your board, the WS2812B-2020 is part C965555 (an "extended" part).

Pixels at full white get warm. Leave them some airflow in your case design, and you'll rarely need full brightness anyway.

## Adding inputs

These are the inputs Hack Club projects use, with the XIAO RP2040 pins for the code below.

| Input | Schematic connection | XIAO pin |
| --- | --- | --- |
| Tactile or low-profile switch (`SW_Push`) | One side to GPIO, other side to GND. Use the internal pull-up. | D3 |
| EC11 rotary encoder (`RotaryEncoder_Switch_MP`) | A and B to GPIOs, C and MP to GND. The push switch is like a button. | D1 (A), D2 (B) |
| Slide pot (`R_Potentiometer_MountingPin`) | Pin 1 to GND, pin 3 to +3V3, wiper (pin 2) to an ADC pin | D0 / A0 |
| AHT20 temperature sensor (I2C) | VCC to 3V3, GND, SDA and SCL. Add 4.7 kΩ pull-ups if your module doesn't have them. | D4 (SDA), D5 (SCL) |

Feed the slide pot from **3.3 V, not 5 V**, because the ADC pin can't take 5 V. For lots of buttons, use a key matrix with a 1N4148 diode per key, like the Blueprint MIDI guide.

**Example: a ring you can steer.** This firmware is something you write now and flash once your board is built. Turning the encoder moves a lit pixel around a 12-pixel ring, the button changes its colour and the slide pot sets brightness.

```cpp
#include <Adafruit_NeoPixel.h>

#define PIXEL_PIN D6   // through the level shifter
#define NUM_PIXELS 12
#define ENC_A D1
#define ENC_B D2
#define BUTTON D3
#define SLIDER A0

Adafruit_NeoPixel ring(NUM_PIXELS, PIXEL_PIN, NEO_GRB + NEO_KHZ800);

int position = 0;     // which pixel is lit
uint16_t hue = 0;     // its colour (0-65535 goes all the way round)
int lastA = HIGH;
int lastButton = HIGH;

void setup() {
  ring.begin();
  pinMode(ENC_A, INPUT_PULLUP);
  pinMode(ENC_B, INPUT_PULLUP);
  pinMode(BUTTON, INPUT_PULLUP);  // reads LOW when pressed
}

void loop() {
  // Encoder: when A falls, B tells us which way it turned
  int a = digitalRead(ENC_A);
  if (a == LOW && lastA == HIGH) {
    position += (digitalRead(ENC_B) == HIGH) ? 1 : -1;
    position = (position + NUM_PIXELS) % NUM_PIXELS;  // wrap around
  }
  lastA = a;

  // Button: one step round the colour wheel per press
  int b = digitalRead(BUTTON);
  if (b == LOW && lastButton == HIGH) hue += 65536 / 6;
  lastButton = b;

  // Slider: 0-1023 becomes brightness 5-80 (keeps current low)
  int level = map(analogRead(SLIDER), 0, 1023, 5, 80);

  ring.clear();
  ring.setPixelColor(position, ring.gamma32(ring.ColorHSV(hue, 255, level)));
  ring.show();
  delay(2);
}
```

The encoder counts each time pin A falls, and B decides the direction.

The slider sets the HSV "value", so brightness stays low without calling `setBrightness()` over and over. On an ESP32 the ADC reads 0-4095, so change the `map()` range.

The same thing in MicroPython (XIAO RP2040, so `Pin(0)` is D6):

```python
from machine import Pin, ADC
import neopixel
import time

NUM = 12
np = neopixel.NeoPixel(Pin(0), NUM)       # D6
enc_a = Pin(27, Pin.IN, Pin.PULL_UP)      # D1
enc_b = Pin(28, Pin.IN, Pin.PULL_UP)      # D2
button = Pin(29, Pin.IN, Pin.PULL_UP)     # D3
slider = ADC(26)                          # D0 / A0

COLOURS = [(1, 0, 0), (1, 1, 0), (0, 1, 0), (0, 1, 1), (0, 0, 1), (1, 0, 1)]
position = 0
colour = 0
last_a = 1
last_button = 1

while True:
    a = enc_a.value()
    if a == 0 and last_a == 1:            # A just fell
        position = (position + (1 if enc_b.value() else -1)) % NUM
    last_a = a

    b = button.value()
    if b == 0 and last_button == 1:       # button just pressed
        colour = (colour + 1) % len(COLOURS)
    last_button = b

    level = 5 + slider.read_u16() * 75 // 65535   # brightness 5-80
    r, g, bl = COLOURS[colour]
    np.fill((0, 0, 0))
    np[position] = (r * level, g * level, bl * level)
    np.write()
    time.sleep_ms(2)
```

**Add a sensor (Hermes-style).** This CircuitPython snippet turns the ring into a thermometer. 15 °C lights nothing, 35 °C lights all 12 pixels.

```python
import time
import board
import neopixel
import adafruit_ahtx0

pixels = neopixel.NeoPixel(board.D6, 12, brightness=0.2, auto_write=False)
sensor = adafruit_ahtx0.AHTx0(board.I2C())    # uses SDA (D4) and SCL (D5)

while True:
    lit = int((sensor.temperature - 15) / 20 * 12)
    lit = max(0, min(12, lit))                 # keep it between 0 and 12
    for i in range(12):
        pixels[i] = (255, 60, 0) if i < lit else (0, 0, 0)
    pixels.show()
    time.sleep(1)
```

When you flash it, copy `neopixel.mpy`, `adafruit_pixelbuf.mpy`, `adafruit_ahtx0.mpy` and the `adafruit_bus_device` folder from the CircuitPython library bundle into `lib`. In Arduino, the `Adafruit AHTX0` library does the same job with `aht.getEvent(&humidity, &temp)`.

## Coding it

Displays week asks for firmware, so you write it now and flash it once your board is built.

**Arduino:** use [Adafruit_NeoPixel](https://github.com/adafruit/Adafruit_NeoPixel) for simplicity, or [FastLED](https://github.com/FastLED/FastLED) for fancy effects and power limiting. Install either from the Library Manager.

**MicroPython:** the [`neopixel` module](https://docs.micropython.org/en/latest/library/neopixel.html) is built into the ESP32 and RP2 (Pico, XIAO RP2040) firmware, so there's nothing to install. **CircuitPython:** use [Adafruit_CircuitPython_NeoPixel](https://github.com/adafruit/Adafruit_CircuitPython_NeoPixel), which also handles brightness for you.

Hello world in Arduino (Adafruit_NeoPixel):

```cpp
#include <Adafruit_NeoPixel.h>

#define PIN D6         // XIAO RP2040 (Pico: 6)
#define NUM_PIXELS 8

Adafruit_NeoPixel strip(NUM_PIXELS, PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  strip.begin();            // set up the data pin
  strip.setBrightness(40);  // about 15%, call this once here
}

void loop() {
  for (int i = 0; i < NUM_PIXELS; i++) {
    strip.clear();                                     // all off (in memory)
    strip.setPixelColor(i, strip.Color(0, 150, 255));  // pixel i sky blue
    strip.show();                                      // send it to the LEDs
    delay(100);
  }
}
```

This chases one blue pixel down the strip. `setPixelColor()` only changes memory. Nothing lights up until `show()`.

Change `NUM_PIXELS` to match your strip.

The same idea with FastLED, plus a rainbow and a power cap:

```cpp
#include <FastLED.h>

#define DATA_PIN D6  // XIAO RP2040
#define NUM_LEDS 8
CRGB leds[NUM_LEDS];

void setup() {
  FastLED.addLeds<WS2812B, DATA_PIN, GRB>(leds, NUM_LEDS);
  FastLED.setMaxPowerInVoltsAndMilliamps(5, 400);  // never draw over 400 mA
}

void loop() {
  static uint8_t hue = 0;
  fill_rainbow(leds, NUM_LEDS, hue++, 255 / NUM_LEDS);  // shifting rainbow
  FastLED.show();
  delay(20);
}
```

`setMaxPowerInVoltsAndMilliamps()` dims everything automatically if your colours would pull too much current, which matters on USB power. Raise the 400 if you have a bigger supply.

Hello world in MicroPython:

```python
from machine import Pin
import neopixel

np = neopixel.NeoPixel(Pin(0), 8)  # 8 pixels on GP0 (Pico) or D6 (XIAO RP2040)
np.fill((0, 0, 20))                # dim blue everywhere
np[0] = (40, 0, 0)                 # first pixel red
np.write()                         # nothing changes until write()
```

Colours are `(red, green, blue)` tuples from 0 to 255. MicroPython has no brightness setting, so keep the numbers small. 20-50 is bright enough indoors. For RGBW pixels, add `bpp=4` and use four-number tuples.

## Make it yours

Use these as starting points for a design of your own.

- **Ring clock.** Show hours, minutes and seconds as three colours on a 60-pixel ring, set with an encoder ([mircemk's build](https://hackaday.io/project/182304)). Teaches mapping time to pixel positions.
- **VU meter.** Light up more pixels the louder it gets ([ericBcreator's stereo ring meter](https://projecthub.arduino.cc/ericBcreator/3b76c1b6-0942-41a0-ac17-3b0942867aae)). Teaches fast analog sampling and smoothing.
- **1D Pong.** Two players with a button each share one strip and hit the ball when it reaches your zone ([ogermer/1d-pong](https://github.com/ogermer/1d-pong)). Teaches game loops and reaction timing.
- **Mood thermometer.** Fade from blue to red as your AHT20 warms up, then add humidity as a second ring. Teaches I2C sensors and colour blending.
- **Pixel game on a grid.** Build Tetris or Snake on an LED grid with buttons ([Adafruit's NeoPixel Tetris](https://learn.adafruit.com/tetris-building)). Teaches turning (x, y) into a pixel number on a zigzag grid.
- **Smart lamp.** Put an ESP32 on your board and control the pixels from your phone with [WLED](https://github.com/wled/WLED). Teaches Wi-Fi firmware and power design for big strips.

When you're done, submit your schematic, PCB and firmware.

## Sources

- [Adafruit NeoPixel Überguide: The Magic of NeoPixels](https://learn.adafruit.com/adafruit-neopixel-uberguide/the-magic-of-neopixels)
- [Adafruit NeoPixel Überguide: Basic Connections](https://learn.adafruit.com/adafruit-neopixel-uberguide/basic-connections)
- [Adafruit NeoPixel Überguide: Best Practices](https://learn.adafruit.com/adafruit-neopixel-uberguide/best-practices)
- [Adafruit NeoPixel Überguide: Powering NeoPixels](https://learn.adafruit.com/adafruit-neopixel-uberguide/powering-neopixels)
- [Adafruit NeoPixel Überguide: Arduino Library Use](https://learn.adafruit.com/adafruit-neopixel-uberguide/arduino-library-use)
- [Adafruit NeoPixel Überguide: Python & CircuitPython](https://learn.adafruit.com/adafruit-neopixel-uberguide/python-circuitpython)
- [Adafruit_NeoPixel on GitHub](https://github.com/adafruit/Adafruit_NeoPixel)
- [FastLED on GitHub](https://github.com/FastLED/FastLED) and [FastLED wiki: Basic usage](https://github.com/FastLED/FastLED/wiki/Basic-usage)
- [FastLED power management docs](https://fastled.io/docs/d3/d1d/group___power.html)
- [MicroPython neopixel module docs](https://docs.micropython.org/en/latest/library/neopixel.html)
- [Adafruit_CircuitPython_AHTx0](https://github.com/adafruit/Adafruit_CircuitPython_AHTx0) and [Adafruit_AHTX0 Arduino example](https://raw.githubusercontent.com/adafruit/Adafruit_AHTX0/master/examples/adafruit_aht_test/adafruit_aht_test.ino)
- [ShawnHymel/neopixel-workshop](https://github.com/ShawnHymel/neopixel-workshop)
- [Hackaday: Cheating at 5V WS2812 control to use a 3.3V data line](https://hackaday.com/2017/01/20/cheating-at-5v-ws2812-control-to-use-a-3-3v-data-line/)
- [Arduino Getting Started: WS2812B LED strip](https://arduinogetstarted.com/tutorials/arduino-ws2812b-led-strip)
- [Core Electronics: WS2812 addressable LEDs quickstart](https://core-electronics.com.au/guides/arduino/ws2812-addressable-leds-arduino-quickstart-guide/)
- [WS2812B datasheet (5050)](https://www.luxalight.eu/sites/default/files/downloads/2020-03/Datasheet_WS2812B.pdf)
- [WS2812B-2020-V6 datasheet (Worldsemi)](https://www.tme.com/Document/f57342b0dc3e7d573aa1f8b6b5942fcf/WS2812B-2020-V6_V1.2_EN.pdf)
- [TI SN74AHCT1G125 datasheet](https://www.ti.com/lit/gpn/sn74ahct1g125) and [Adafruit 74AHCT125 product page](https://www.adafruit.com/product/1787)
- [JLCPCB part C965555 (WS2812B-2020)](https://jlcpcb.com/partdetail/Worldsemi-WS2812B2020/C965555)
- [DigiKey PCB trace width calculator](https://www.digikey.com/en/resources/conversion-calculators/conversion-calculator-pcb-trace-width)
- [TechOverflow: JST XH current rating](https://www.techoverflow.net/2023/02/21/what-current-rating-do-jst-xh-connectors-have)
- [Seeed Studio XIAO RP2040 wiki](https://wiki.seeedstudio.com/XIAO-RP2040/)
- [KaiPereira/midi-keypad](https://github.com/KaiPereira/midi-keypad), [Starbie Week 1 Guide](https://github.com/SharKingStudios/Starbie/blob/main/Week%201%20Guide.md), [Blueprint MIDI starter project](https://blueprint.hackclub.com/starter-projects/midi), [Pathfinder](https://pathfinder.hackclub.com/) and [Hermes](https://hermes.hackclub.com/)
- [mircemk: DIY WS2812 analog-style ring clock](https://hackaday.io/project/182304)
- [ericBcreator: Stereo NeoPixel Ring VU Meter](https://projecthub.arduino.cc/ericBcreator/3b76c1b6-0942-41a0-ac17-3b0942867aae)
- [ogermer/1d-pong](https://github.com/ogermer/1d-pong)
- [Adafruit: MIT Green Building NeoPixel Tetris](https://learn.adafruit.com/tetris-building)
- [WLED on GitHub](https://github.com/wled/WLED)
