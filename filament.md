# LED Filaments

## What it is

An LED filament is a strip of tiny LED chips packed so close together that they glow as one continuous line. You've probably seen them inside "Edison-style" bulbs, where four or more stand up inside clear glass.

![A clear Philips E27 LED filament bulb with four orange filaments standing inside the glass](/art/starters/displays/filament-1.jpg)
*Photo: W.carter, CC BY-SA 4.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:A_clear_Philips,_E27,_4.3W,_LED_filament_light_bulb,_upright.jpg)*

There are three kinds you'll come across:

- **Flexible 3 V filaments** ("nOOds", "LED noodles"): 60-600 mm long, about 1.5-2 mm thick, coated in coloured silicone. You can bend them into letters and shapes. These are the best place to start.
- **Rigid filaments** (usually 38 mm long): a ceramic or glass strip, sold in 3 V and 12 V versions. They work well for seven-segment digits, but snap like thin china.
- **Filaments pulled from mains bulbs**: these need roughly 55-100 V. **Skip these until you're experienced**, because the voltages can give you a real shock.

Each filament has two metal tabs. **The tab with a tiny hole is the anode (+).** Wire it backwards and it stays dark, so double-check which pad is the anode in your schematic and footprint.

## How it works

Under the silicone is a row of blue LED chips on a thin metal or ceramic strip. A yellow phosphor coating turns the blue light into warm white, red, pink and so on. In the close-up below, the filament is running at only 0.5 mA, so you can see its 28 separate chips through the phosphor.

![A 38 mm rigid LED filament glowing dimly, showing 28 separate LED chips under yellow phosphor](/art/starters/displays/filament-2.jpg)
*Photo: wdwd, CC BY-SA 4.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:LED_Filament_CloseUp_at_5_%25_power.JPG)*

In a 3 V filament the chips are wired in parallel, so the whole thing behaves like one big LED with a forward voltage (Vf) of about 2.4-3 V. 12 V filaments have chips in series groups, so they need about 12 V to light.

An LED's current shoots up when the voltage rises just a little. Adafruit measured this on one of their 3 V noodles:

| Voltage across filament | Current |
|---|---|
| 2.5 V | 10 mA |
| 2.7 V | 60 mA |
| 2.9 V | 100 mA |
| 3.0 V | 120 mA |

A 0.5 V change gives 12 times the current, so **always limit the current. Never connect a filament straight to a battery or power pin.**

### How much current?

There's no proper datasheet for hobby filaments, and sellers disagree a lot:

| Seller | What they say |
|---|---|
| Adafruit nOOds (3 V) | treat as 50 mA max |
| Pimoroni LED Noodles (3 V) | tested to 200 mA, 50-100 mA for long life |
| Makerguides' 130 mm filament | 100 mA max |
| Brickstuff 38 mm rigid (3 V) | about 120 mA |
| SuperLightingLED flexible (3 V) | 300 mA max |
| Oznium flexible (12 V) | 130 mA |

**Design for 50 mA per 3 V filament unless your listing clearly says more.** That's bright enough, and the filament will last.

### Two ways to limit the current

1. **A series resistor.** Pick R = (supply voltage - Vf) ÷ current. From 5 V at 50 mA with Vf = 2.5 V: (5 - 2.5) ÷ 0.05 = 50 Ω. Round *up* to 51 Ω or 56 Ω. This works well when your supply is steady.
2. **A constant-current driver.** This holds the current fixed even if the supply sags (like a battery running down). Kevin Walters' Instructable builds one from two BC337 transistors: the current is roughly 0.6 V ÷ the sense resistor, so 13 Ω gives about 48 mA. There are also driver chips and boards, like the AW9523 (no per-filament resistor needed).

Use 5 V, not 3.3 V, for 3 V filaments. With 3.3 V there's only about 0.5 V left for the resistor. Then small differences between filaments make big differences in brightness.

### Why your microcontroller pin can't do it alone

A GPIO pin can only supply a small current. The RP2040 (Pico, XIAO RP2040) can give at most 12 mA per pin. A filament wants 50-100 mA.

So the pin controls a transistor switch, and the transistor carries the filament current from the 5 V rail. Use a logic-level N-channel MOSFET (it fully turns on from 3.3 V, like the AO3400A) or an NPN transistor such as the BC337. The switch goes on the ground side of the filament ("low-side").

### PWM dimming

To dim a filament, the microcontroller flicks the transistor on and off hundreds or thousands of times per second. This is PWM (pulse-width modulation). On 50% of the time looks about half as bright, and your eye never sees the flicker.

The resistor sets the *maximum* current, so any PWM value is safe. Filaments have more capacitance than normal LEDs (about 4-8 nF), but Kevin's tests showed PWM at about 2 kHz still switches cleanly.

![A lit LED filament bulb close up, showing rows of glowing chips on each filament](/art/starters/displays/filament-3.jpg)
*Photo: Emce wola, CC BY-SA 4.0, via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Modu%C5%82y_led_filament_podczas_dzia%C5%82ania.jpg)*

## Using it in a project

### Parts for your design

These are the parts that go in your schematic and your BOM. They're bought with your funding once your design is approved.

| Part | What it's for | Typical driver | Notes |
|---|---|---|---|
| Flexible 3 V filament, 130-300 mm (Adafruit nOOds, Pimoroni LED Noodles) | Signs, lamps, wearables | Logic-level MOSFET + resistor, or AW9523 | Can't be cut. Anode tab has a hole. |
| Rigid 3 V filament, 38 mm | Seven-segment digits, mini lamps | MOSFET + resistor; 74HC595 shift register + transistors for many | Extremely fragile. About 2.6 V, 120 mA (Brickstuff). |
| Rigid or flexible 12 V filament | Bigger signs, longer runs | Low-side MOSFET from a 12 V supply | Some include a resistor, which gets warm. Check the listing. |
| AO3400A MOSFET (SOT-23, LCSC C20917) | PCB switch, one per filament | – | 30 V, 5.7 A, 48 mΩ at 2.5 V gate. |
| BC337 NPN transistor (TO-92) | Through-hole switch | – | Needs a 1 kΩ base resistor. |
| 51-56 Ω resistors, 1/4 W | Current limit at 5 V | – | One per filament. |
| Fine wire, heat-shrink, clear heat-shrink | Connections and gluing | – | The tabs are too thin to connect directly, so they need wires. |

**Designing around the filament:**

- **Flexible filaments bend front-to-back only**, never sideways. A full twist every 25 mm is the limit, so draw your shapes with that in mind.
- **Each bend needs holding in place.** Repeated flexing breaks chips, so give your design a channel or mount that fixes every bend.
- **Glue won't stick to the silicone.** Plan to hold filaments with clear heat-shrink and glue that instead.
- **Filaments can't be cut shorter.** Pick lengths that fit, and plan where any extra length hides.

### On a PCB

This section assumes a Seeed XIAO RP2040 and KiCad, like Pathfinder and the MIDI keypad guides. A Pico works the same way.

#### 1. Schematic

- Press `A` and add the XIAO. Get its symbol and footprint from the Hack Club KiCad library used in the MIDI keypad guide.
- For each filament, add one channel. Use `Transistor_FET:AO3400A` (search "AO3400") as the switch.
- Add a `Device:R` 56 Ω current-limit resistor between `+5V` and the filament anode.
- Add a `Device:R` 100 Ω from the XIAO pin to the gate, and 100 kΩ from gate to GND. The pull-down keeps the filament off while the board boots.
- Use `Device:LED` or `Connector_Generic:Conn_01x02` for the filament itself.
- Use net labels (`L`) like `FIL1`, `FIL2` instead of long wires. Power labels point up, ground points down.
- Add a 10 µF capacitor (`Device:C_Polarized` or `Device:C`) across +5V and GND. PWM switches lots of current on and off, and this smooths it out.
- Press `Q` to mark unused XIAO pins with no-connect flags.

Which pins to connect in your schematic:

| Connection | Pico | XIAO RP2040 |
|---|---|---|
| PWM to transistor | GP15 | D8 (GPIO2) |
| 5 V for filament | VBUS (pin 40, USB 5 V) | 5V |
| Ground | GND (pin 38) | GND |
| Knob / slide pot wiper | GP26 (ADC0) | A0 (GPIO26) |
| Button (to GND) | GP14 | D3 (GPIO29) |

**Power budget:** a USB port gives about 500 mA. Kevin measured six filaments at 85 mA each pulling 0.54 A in total. So keep to about four filaments at 50-85 mA on USB, or design in a separate 5 V supply.

**12 V filaments:** same circuit, but the resistor's top end goes to a 12 V supply. Size the resistor from your listing, since some 12 V filaments have one built in. **Never connect 12 V to any pin on your dev board.** Only the shared GND links the two.

**Schematic checks:**

- Every filament has a current-limit resistor. Without one, the filament (or your pin) burns out.
- No filament sits straight on a GPIO pin. It would be dim and stress the chip.
- The filament supply is 5 V, not the 3V3 pin. On 3V3 the current varies a lot between filaments, and the regulator isn't meant for it.

#### 2. Footprints

| Symbol | Footprint |
|---|---|
| AO3400A | `Package_TO_SOT_SMD:SOT-23` |
| Current-limit resistor | `Resistor_SMD:R_1206_3216Metric` (or a through-hole `Resistor_THT` 1/4 W) |
| Gate resistors | `Resistor_SMD:R_0805_2012Metric` |
| Filament (plug-in) | `Connector_JST:JST_PH_B2B-PH-K_1x02_P2.00mm_Vertical` |
| Filament (wired straight to the board) | a wire pad from the `Connector_Wire` library (search "SolderWire") |

**Why 1206 for the current-limit resistor?** At 50 mA, a 56 Ω resistor burns 0.05² × 56 = 0.14 W. A typical 0805 is rated 0.125 W, which is too small. A 1206 is usually rated 0.25 W. At 100 mA, use a 1/2 W part.

**Mounting the filament on the board:** Jay Hamlin's clock gives each 20 mm filament a slot cut in the PCB (11.5 × 2.0 mm). The two pads are 14 mm apart. Copy that idea for digits: draw the slot on `Edge.Cuts` inside the board.

#### 3. Layout and routing

- Keep each MOSFET right next to its connector, with the resistor beside it. The filament current loop stays short.
- Signal traces (gate, buttons, I2C): 0.25 mm.
- Filament traces: 0.5 mm.
- The +5V trunk feeding several filaments: 0.8-1.0 mm.
- Pour a ground plane on the back (`B.Cu`).
- Space the current-limit resistors out; they get warm. Kevin put his hot parts at one end of his board.
- For filament wires attached straight to the board, add two holes next to each pad for the wire to thread through (or add a zip-tie slot), so a tug pulls on the board, not the joint. This is called strain relief.
- Put a `+` on the silkscreen by every anode pad.
- To re-route a trace, select it with `U`, delete it and route it again.

#### 4. Check before you submit

- Run DRC. Fix every unconnected-net error.
- Add up your current: number of filaments × current each. If it's over 400 mA, plan a separate 5 V supply with a DC jack or screw terminal.
- Check every AO3400A is the right way round: gate to the XIAO, drain to the filament, source to GND.

## Adding inputs

This is how to wire the inputs from Hack Club's starter projects into your schematic.

- **Buttons (tactile or low-profile Choc switches):** one side to a GPIO, the other to GND. Use the internal pull-up, so pressed reads LOW. For lots of buttons, use a key matrix with a 1N4148 diode per key, all facing the same way.
- **Knob or slide pot (Bourns):** pin 1 to GND, pin 3 to 3V3, pin 2 (wiper) to an ADC pin like A0. Use the `R_Potentiometer_MountingPin` symbol for the slide pot, and tie its mounting pins to GND.
- **EC11 rotary encoder:** A and B to two GPIOs, C (middle) to GND. The push switch is just another button.
- **I2C light sensor (like the VEML7700):** SDA to D4, SCL to D5, plus 3V3 and GND. Add a 100 nF capacitor right next to it. Hermes is a good example of a sensor-on-PCB design.

### Example: knob for brightness, button for candle mode (MicroPython)

This is firmware for a XIAO RP2040. Write it now and flash it once your board is built. In MicroPython, the pin number is the GPIO number.

```python
from machine import Pin, PWM, ADC
import time
import random

filament = PWM(Pin(2), freq=2000)   # D8 = GPIO2, to the MOSFET gate
knob = ADC(26)                      # A0 = GPIO26, pot wiper
button = Pin(29, Pin.IN, Pin.PULL_UP)  # D3 = GPIO29, button to GND

candle = False
was_pressed = False

while True:
    pressed = button.value() == 0
    if pressed and not was_pressed:   # only act on a new press
        candle = not candle
    was_pressed = pressed

    level = knob.read_u16() / 65535   # 0.0 to 1.0
    duty = level * level              # squared, so the knob feels even
    if candle:
        duty *= random.uniform(0.6, 1.0)  # random flicker
    filament.duty_u16(int(duty * 65535))
    time.sleep_ms(60 if candle else 10)
```

The knob sets brightness, and the button toggles a candle-like flicker. Squaring the level matters because your eye notices changes at low brightness much more than at high brightness. Change the `0.6` to make the flicker wilder or calmer.

### Encoder brightness (CircuitPython)

```python
import board
import pwmio
import rotaryio

filament = pwmio.PWMOut(board.D8, frequency=2000)
encoder = rotaryio.IncrementalEncoder(board.D1, board.D2)

while True:
    step = max(0, min(20, encoder.position))  # 21 brightness steps
    filament.duty_cycle = int((step / 20) ** 2 * 65535)
```

Each click moves one brightness step. `rotaryio` turns on the pull-ups for you.

### Auto-dim with a light sensor (CircuitPython)

```python
import time
import board
import pwmio
import adafruit_veml7700

filament = pwmio.PWMOut(board.D8, frequency=2000)
sensor = adafruit_veml7700.VEML7700(board.I2C())  # D4 = SDA, D5 = SCL

while True:
    room = min(sensor.light / 2000, 1.0)  # print sensor.light to find your range
    filament.duty_cycle = int((0.1 + 0.9 * room) ** 2 * 65535)
    time.sleep(0.2)
```

A bright room makes the filament brighter, and a dark room dims it so it isn't blinding. The `2000` is a starting guess you can tune once your board is built. Place the sensor where it can't see its own filament.

## Coding it

You write this firmware now and flash it once your board is built. There's no special library to install. Filament control is just PWM, which is built in everywhere.

- **Arduino:** [`analogWrite()`](https://github.com/arduino/reference-en/blob/master/Language/Functions/Analog%20IO/analogWrite.adoc) takes 0-255. For the XIAO RP2040 or Pico, install the [arduino-pico core](https://arduino-pico.readthedocs.io/en/latest/analog.html), which uses 1 kHz by default.
- **MicroPython:** [`machine.PWM`](https://docs.micropython.org/en/latest/library/machine.PWM.html) with `duty_u16()` from 0 to 65535.
- **CircuitPython:** [`pwmio.PWMOut`](https://docs.circuitpython.org/en/latest/shared-bindings/pwmio/index.html) with `duty_cycle` from 0 to 65535. It's the easiest route if you want encoders (`rotaryio`) and Adafruit sensor libraries.

### Hello world in Arduino

```cpp
// Breathe a filament up and down.
// Pin goes to the transistor/MOSFET, never straight to the filament.
const int FILAMENT_PIN = D8;  // XIAO RP2040. Pico: 15.

void setup() {
  // analogWrite sets the pin up itself, no pinMode needed
}

void loop() {
  for (int b = 0; b <= 255; b++) {
    analogWrite(FILAMENT_PIN, b);
    delay(8);
  }
  for (int b = 255; b >= 0; b--) {
    analogWrite(FILAMENT_PIN, b);
    delay(8);
  }
}
```

This fades the filament from off to full and back, about every 4 seconds. Change `delay(8)` to speed it up or slow it down. Because your resistor already caps the current, 255 is safe.

Makerguides' tutorial limits `analogWrite` to 60. That's because their driver board could push up to 910 mA, far too much for a 100 mA filament. With the right resistor or driver, you don't need that trick.

### Hello world in MicroPython

```python
from machine import Pin, PWM
import time

filament = PWM(Pin(2), freq=2000)  # XIAO RP2040 D8. Pico: Pin(15)

while True:
    for b in range(0, 65536, 512):        # fade up
        filament.duty_u16(b)
        time.sleep_ms(10)
    for b in range(65535, -1, -512):      # fade down
        filament.duty_u16(b)
        time.sleep_ms(10)
```

Same breathing effect, but with 16-bit steps. Change `512` for bigger or smaller steps. With several filaments, give each its own `PWM` object and pin.

If several filaments switch on at the same moment, your supply sees one big current spike. Kevin's trick is to give each pair of channels a slightly different frequency (like 1907, 1986 and 2034 Hz). That way the "on" moments drift apart.

## Make it yours

Use the circuits in this guide as a starting point and design your own thing. Some ideas:

1. **Floating seven-segment clock.** Make digits from 38 mm or 20 mm filaments and keep time over WiFi with an ESP32. *Teaches:* multiplexing with shift registers and NTP time. See Jay Hamlin's [test-tube clock](https://hackaday.io/project/184731-using-led-filaments-to-make-a-digital-clock), the [12 V Maxitron digits](https://hackaday.io/project/182655-led-filament-based-seven-segment-displays) and andypugh's ["Charlotte's Web" clock](https://www.instructables.com/Charlottes-Web-Style-LED-Filament-Clock/).
2. **Neon-style name sign.** Press flexible filaments into a 3D-printed channel to spell your name or a doodle. *Teaches:* bend limits, strain relief and one MOSFET per letter.
3. **Candle lantern.** Three filaments that flicker out of sync, with an encoder for brightness. *Teaches:* random PWM and multiple channels. The [upcycled light](https://magazine.raspberrypi.com/articles/upcycled-light) in HackSpace is good inspiration.
4. **Self-dimming desk lamp.** Add an I2C light sensor so the lamp follows the room. *Teaches:* I2C and mapping sensor readings to brightness.
5. **Ring clock.** One filament per minute and hour, driven by shift registers. *Teaches:* driving dozens of outputs from a few pins. See [FilamenTIME](https://hackaday.com/2025/03/25/led-filaments-become-attractive-time-piece/) (72 filaments, 9 shift registers).
6. **Glowing sculpture.** Wire filaments into a 3D shape where every edge lights up. *Teaches:* series and parallel paths and equal current. See [the graph theory of circuit sculptures](https://hackaday.com/2026/02/03/the-graph-theory-of-circuit-sculptures/).

## Sources

- [Kevin Walters: Constant Current Circuit for Flexible Filament LEDs (Instructables)](https://www.instructables.com/Constant-Current-Circuit-for-Flexible-Filament-LED/)
- [Makerguides: Control COB LED filament with Arduino](https://www.makerguides.com/?p=13486)
- [andypugh: "Charlotte's Web" Style LED Filament Clock (Instructables)](https://www.instructables.com/Charlottes-Web-Style-LED-Filament-Clock/)
- [LED filament based seven-segment displays (Hackaday.io)](https://hackaday.io/project/182655-led-filament-based-seven-segment-displays)
- [Using LED filaments to make a digital clock (Hackaday.io)](https://hackaday.io/project/184731-using-led-filaments-to-make-a-digital-clock)
- [Hackaday: LED filament tag](https://hackaday.com/tag/led-filament/)
- [Hackaday: LED filaments become attractive time piece](https://hackaday.com/2025/03/25/led-filaments-become-attractive-time-piece/)
- [Hackaday: A simple tip for gluing those LED filaments](https://hackaday.com/2025/05/29/a-simple-tip-for-gluing-those-led-filaments/)
- [Hackaday: The graph theory of circuit sculptures](https://hackaday.com/2026/02/03/the-graph-theory-of-circuit-sculptures/)
- [HackSpace: Upcycled light](https://magazine.raspberrypi.com/articles/upcycled-light)
- [Adafruit: nOOds 3 V 300 mm warm white product page](https://www.adafruit.com/product/5503)
- [Adafruit: nOOds überguide overview](https://learn.adafruit.com/noods-uberguide)
- [Adafruit: nOOds electrical properties](https://learn.adafruit.com/noods-uberguide/electrical-properties-2)
- [Adafruit: nOOds physical properties](https://learn.adafruit.com/noods-uberguide/physical-properties)
- [Pimoroni: LED Noodles](https://shop.pimoroni.com/products/led-noodles)
- [Brickstuff: Rigid 3 V DC LED filament (Tindie)](https://www.tindie.com/products/brickstuff/rigid-3v-dc-led-filament-5-pack-38mm/)
- [Oznium: COB LED filament (12 V)](https://www.oznium.com/flexible-led-strips/cob-led-filament)
- [SuperLightingLED: 3 V flexible COB LED filament](https://www.superlightingled.com/3-volt-diy-cob-flexible-led-filament-dimmable-light-25pcs-p-6409.html)
- [LCSC: AO3400A (C20917)](https://www.lcsc.com/product-detail/C20917.html)
- [Arduino: analogWrite() reference source](https://github.com/arduino/reference-en/blob/master/Language/Functions/Analog%20IO/analogWrite.adoc)
- [arduino-pico: Analog I/O](https://arduino-pico.readthedocs.io/en/latest/analog.html)
- [MicroPython: machine.PWM](https://docs.micropython.org/en/latest/library/machine.PWM.html)
- [CircuitPython: pwmio](https://docs.circuitpython.org/en/latest/shared-bindings/pwmio/index.html)
- [CircuitPython: rotaryio](https://docs.circuitpython.org/en/latest/shared-bindings/rotaryio/index.html)
- [Adafruit CircuitPython VEML7700 (GitHub)](https://github.com/adafruit/Adafruit_CircuitPython_VEML7700)
- [Seeed Studio: XIAO RP2040 wiki](https://wiki.seeedstudio.com/XIAO-RP2040/)
- [Kai Pereira: MIDI keypad KiCad guide](https://github.com/KaiPereira/midi-keypad)
- [SharKingStudios: Starbie Week 1 guide](https://github.com/SharKingStudios/Starbie/blob/main/Week%201%20Guide.md)
- [Hack Club Blueprint: MIDI starter project](https://blueprint.hackclub.com/starter-projects/midi)
- [Hack Club Pathfinder](https://pathfinder.hackclub.com/)
- [Hack Club Hermes](https://hermes.hackclub.com/)
