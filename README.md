# AID Challenge 1: Visual vs. Auditory Reaction Time

A small reaction-time experiment that compares how fast people respond to a **visual** stimulus (an LED) and an **auditory** stimulus (a buzzer). A [Processing](https://processing.org/) sketch runs the experiment. It controls an Arduino running **StandardFirmata** by writing raw Firmata bytes over serial, and it saves each participant's results to a CSV file.

## Repository contents

| File | Description |
|------|-------------|
| `Challenge_1.java` | The Processing sketch that runs the experiment |
| `Challenge_01.pdf` | The challenge brief, with wiring and setup notes |

## Hardware

- Arduino Uno (or a compatible board)
- Push button
- LED and a 220 Ω resistor
- Active buzzer (it beeps when its pin is driven HIGH)
- Breadboard and jumper wires

### Wiring

| Component | Arduino pin | Connection |
|-----------|-------------|------------|
| Button | D2 | D2 → button → GND (uses the internal pull-up) |
| LED | D4 | D4 → 220 Ω → LED anode, LED cathode → GND |
| Buzzer | D6 | D6 → buzzer (+), buzzer (−) → GND |

## Setup

1. **Flash the Arduino:** in the Arduino IDE, open *File → Examples → Firmata → StandardFirmata* and upload it to the board.
2. **Install Processing** (version 4 recommended). The sketch uses the built-in `processing.serial` library.
3. **Open the sketch:** Processing expects the sketch to live in a folder with the same name and to use a `.pde` extension. For example:
   ```
   Challenge_1/
   └── Challenge_1.pde   (copy or rename of Challenge_1.java)
   ```
4. **Choose the serial port:** when the sketch starts, it prints the available ports to the console. Set `SERIAL_INDEX` to the index of your Arduino's port.

## Configuration

These settings are at the top of the sketch:

| Setting | Default | Meaning |
|---------|---------|---------|
| `participant` | `"P01"` | Participant ID, written to the CSV and used in its filename |
| `TRIALS_PER_TYPE` | `10` | Trials per stimulus type (10 visual + 10 auditory = 20 in total) |
| `FOREPERIOD_MIN` / `FOREPERIOD_MAX` | `1000` / `4000` ms | Range of the random wait before each stimulus |
| `RESPONSE_WINDOW` | `2000` ms | Maximum time allowed to respond |
| `ITI` | `800` ms | Pause between trials |
| `SERIAL_INDEX` | `4` | Index of the Arduino in `Serial.list()` |
| `BAUD` | `57600` | Must match `Firmata.begin(...)` in StandardFirmata |

Change `participant` before each session so that runs don't get mixed up.

## Running the experiment

1. Connect the Arduino and run the sketch. Wait about 2.5 s, because the board resets when the port opens.
2. Check the hardware. The status line at the bottom of the window shows the button state live. If it says `NO DATA from Arduino`, check the port and the Firmata upload.
   - **L** turns the LED on or off.
   - **B** turns the buzzer on or off.
3. Press **SPACE** to start.
4. For each trial, wait during "Get ready...", then press the button as soon as the LED lights up or the buzzer sounds.

Visual and auditory trials come in a random order. Each trial ends in one of three ways:

| Outcome | When it happens |
|---------|-----------------|
| `ok` | The button was pressed after the stimulus, within the response window |
| `false_start` | The button was pressed before the stimulus ("Too early!") |
| `timeout` | No press came within `RESPONSE_WINDOW` ("Too slow") |

## Output

When the session ends, the sketch saves a CSV file to the sketch's `data/` folder:

```
data/<participant>_reaction_times_<YYYYMMDD>_<HHMM>.csv
```

| Column | Description |
|--------|-------------|
| `participant` | Participant ID |
| `trial` | Trial number (starting at 1) |
| `stimulus` | `visual` or `auditory` |
| `foreperiod_ms` | Random wait before the stimulus |
| `rt_ms` | Reaction time in ms (left empty unless the outcome is `ok`) |
| `outcome` | `ok`, `false_start` or `timeout` |

The sketch also shows a summary on screen and prints it to the console. For each stimulus type, the summary gives the median and mean reaction time, the number of valid trials, and the number of errors.

## How it works

- **Firmata without a library:** the sketch sends Firmata commands directly. It uses `SET_PIN_MODE` (0xF4), `SET_DIGITAL_PIN` (0xF5) and `REPORT_DIGITAL` (0xD0) to configure the pins and drive them, and it parses incoming `DIGITAL_MESSAGE` (0x90) bytes in `serialEvent()` to read the button.
- **Timing:** the moment of a button press is recorded with `System.nanoTime()` as soon as the serial data arrives, and stimulus onset is recorded the same way. The reaction time is the difference between the two, so it does not depend on the sketch's frame rate. It does still include serial and USB latency.
- **State machine:** the experiment moves through these states: `IDLE → PAUSE (ITI) → FORE (foreperiod) → STIM (waiting for a response) → PAUSE ... → DONE`.
