# namma-metro
ESP32-based Namma Metro Passenger Information Display System prototype  
# Namma Metro Passenger Information Display System (PIDS)

**Location:** National College Metro Station, Bengaluru, Karnataka, India  
**Platform:** ESP32 NodeMCU (ESP-WROOM-32, 30-pin)  
**Display:** Two 16×2 LCDs with I²C interfaces  
**Timetable source:** JSON files hosted on GitHub  
**Project type:** Embedded Systems / IoT Mini Project

---

## 1. Project Overview

This project aims to build a miniature Passenger Information Display System (PIDS) for National College Metro Station on the Namma Metro Green Line.

The system uses an ESP32 microcontroller to retrieve timetable data from GitHub, determine upcoming trains, estimate train arrival times at intermediate stations, and display information on two LCDs.

One LCD represents trains travelling towards Madavara, while the other represents trains travelling towards Silk Institute.

The prototype will also include a push button for recording observed train delays and advancing the displayed train, along with red and green LED indicators for each direction.

## 2. Project Objectives

- Display upcoming train information for both directions.
- Store train schedules in separate JSON files for weekdays, Saturdays and Sundays.
- Select the appropriate timetable automatically using network time.
- Calculate estimated arrival times using the route's station-to-station travel intervals.
- Record observed delays through a physical push button.
- Indicate system status using four LEDs.
- Keep the hardware compact inside a custom-built enclosure.
- Make timetable updates possible through GitHub without rewriting the ESP32 firmware.

## 3. System Architecture

```text
        GitHub Repository
        ├── weekday.json
        ├── saturday.json
        └── sunday.json
                |
                | Wi-Fi / HTTPS
                v
          ESP32 NodeMCU
          ├── NTP time
          ├── Day selection
          ├── JSON parsing
          ├── Train selection
          ├── Arrival estimation
          ├── Delay recording
          └── LED control
                |
          I²C Level Shifter
                |
        ┌───────┴────────┐
        v                v
   LCD 0x27          LCD 0x26
   Toward            Toward
   Madavara          Silk Institute

   Push button ──> ESP32
   Four LEDs   <── ESP32
```

## 4. Hardware Components

| Component | Quantity | Purpose |
|---|---:|---|
| ESP32 NodeMCU ESP-WROOM-32, 30-pin | 1 | Main controller |
| 16×2 LCD with PCF8574 I²C adapter | 2 | Direction-specific train displays |
| BSS138 I²C logic-level shifter | 1 | Interface between 3.3 V ESP32 and 5 V LCD I²C |
| Breadboard | 1 | Prototyping connections |
| Momentary push button | 1 | Record delay and advance train |
| Red LEDs | 2 | Delay indication, one per direction |
| Green LEDs | 2 | Normal/ready indication, one per direction |
| 220–330 Ω resistors | 4 | LED current limiting |
| Jumper wires | As required | Electrical connections |
| USB cable and suitable power source | 1 | ESP32 power |
| Cardboard enclosure | 1 | Housing and front panel |

A buzzer is not included in the final design.

## 5. Final GPIO Pin Map

| Function | ESP32 GPIO |
|---|---:|
| I²C SDA — both LCDs | GPIO 21 |
| I²C SCL — both LCDs | GPIO 22 |
| Push button | GPIO 18 |
| Madavara red LED | GPIO 25 |
| Madavara green LED | GPIO 26 |
| Silk Institute red LED | GPIO 27 |
| Silk Institute green LED | GPIO 23 |

These assignments are the intended pin plan and must be kept consistent between the wiring and final firmware.

### LCD I²C Addresses

| LCD | I²C address | Direction |
|---|---|---|
| LCD 1 | `0x27` | Toward Madavara |
| LCD 2 | `0x26` | Toward Silk Institute |

### Push-Button Wiring

The push button connects between GPIO 18 and GND.

```cpp
pinMode(18, INPUT_PULLUP);
```

- Released: `HIGH`
- Pressed: `LOW`

The firmware must debounce the button to avoid registering one press multiple times.

### LED Wiring

Each LED must have its own 220–330 Ω series resistor.

For each LED:

```text
ESP32 GPIO → Resistor → LED anode (+)
LED cathode (-) → GND
```

All component grounds must share a common ground with the ESP32.

### I²C Level Shifter Wiring

- ESP32 GPIO 21 → LV1
- ESP32 GPIO 22 → LV2
- LCD SDA lines → HV1
- LCD SCL lines → HV2
- LV supply → 3.3 V
- HV supply → 5 V
- GND → common ground

Both LCDs share the same SDA and SCL bus but use different I²C addresses.

## 6. GitHub Repository Structure

Repository: [1bi24ec004/namma-metro](https://github.com/1bi24ec004/namma-metro)

The intended repository structure is:

```text
namma-metro/
├── README.md
├── weekday.json
├── saturday.json
├── sunday.json
└── ESP32 firmware source
```

The firmware filename and final folder structure will be confirmed when the code is finalized.

### Raw JSON URLs

```text
[https://raw.githubusercontent.com/1bi24ec004/namma-metro/refs/heads/main/saturday.json]
[https://raw.githubusercontent.com/1bi24ec004/namma-metro/refs/heads/main/sunday.json]
[https://raw.githubusercontent.com/1bi24ec004/namma-metro/refs/heads/main/weekday.json]
```

These URLs assume the three files are saved at the root of the repository's `main` branch.

## 7. Timetable JSON Format

Each timetable file contains the schedule and route information needed by the ESP32.

The common structure includes:

- `schema_version`: JSON schema version.
- `day_type`: timetable day category.
- `station`: current station name.
- `timezone`: `Asia/Kolkata`.
- `display_rules`: display and button behaviour settings.
- `route.stations`: route station names and estimated travel intervals.
- `platforms.madavara`: Madavara-direction timetable and LCD address.
- `platforms.silk_institute`: Silk Institute-direction timetable and LCD address.

Each arrival entry follows this structure:

```json
{
  "time": "07:14",
  "destination": "Madavara"
}
```

The destination field also distinguishes short-turn services where applicable, including Nagasandra, Peenya Industry and Yelachenahalli.

The route intervals are prototype estimates used for arrival calculations, not a guarantee of real-time train positions.

## 8. Train Arrival Display Logic

The intended display behaviour is:

1. Read the selected direction's arrival schedule.
2. Find the relevant upcoming train using the current time.
3. Determine the next train's scheduled arrival at National College.
4. Estimate the train's arrival at National College using the route's travel intervals.
5. Display the remaining time until the train arrives.
6. Update the countdown as time passes.
7. Apply the configured station-estimation and delay-recording rules.

The JSON display rules currently specify:

```json
{
  "station_estimate_until_minutes_remaining": 1,
  "show_delay_placeholder_at_minutes_remaining": 1,
  "start_delay_counter_at_scheduled_arrival": true,
  "record_delay_on_button_press": true,
  "advance_to_next_train_after_button_press": true
}
```

The firmware must implement these rules consistently. In particular, the delay-recording behaviour and the transition to the next train need to be tested on the physical prototype.

## 9. Automatic Timetable Selection

The ESP32 will use Wi-Fi and Network Time Protocol (NTP) to obtain the current date and time.

The intended selection logic is:

| Day | Timetable |
|---|---|
| Monday–Friday | `weekday.json` |
| Saturday | `saturday.json` |
| Sunday | `sunday.json` |

The configured timezone is `Asia/Kolkata`.

The firmware should handle Wi-Fi or timetable-download failures safely and avoid treating an invalid or incomplete JSON response as a valid timetable.

## 10. Push-Button Delay Recording

The physical button is intended to allow the operator to record an observed delay and move to the next train.

Expected sequence:

1. The LCD displays the current train information.
2. The operator observes the train's actual arrival.
3. The operator presses the button to record the event.
4. The firmware records the delay according to the implemented timing rules.
5. The display advances to the next train when appropriate.

The final firmware must define how the delay is measured, stored and shown. A physical button press alone cannot independently determine the actual arrival time unless the operator presses it at the observed event.

## 11. LED Indicator Logic

The final design contains two LEDs per direction.

| Direction | LED | Intended indication |
|---|---|---|
| Toward Madavara | Green | Normal/ready status |
| Toward Madavara | Red | Delay recorded |
| Toward Silk Institute | Green | Normal/ready status |
| Toward Silk Institute | Red | Delay recorded |

The final firmware must define exactly when each LED switches on and off. The green LED should not imply that a train is on time unless the system has enough information to establish that.

## 12. Development Roadmap

### Phase 1 — Hardware foundation

- [x] ESP32 board selected.
- [x] Two LCDs connected.
- [x] LCD I²C communication tested.
- [x] I²C addresses confirmed as `0x27` and `0x26`.
- [ ] Push button wired and tested.
- [ ] Four LEDs wired and tested.
- [ ] Final GPIO assignments verified against the integrated sketch.

### Phase 2 — Timetable preparation

- [x] Route station sequence prepared.
- [x] Weekday timetable data prepared.
- [x] Saturday timetable data prepared.
- [x] Sunday timetable data prepared.
- [ ] Save all three JSON files in the GitHub repository.
- [ ] Validate JSON syntax and required fields.
- [ ] Confirm that the raw GitHub URLs return the correct files.

### Phase 3 — ESP32 software integration

- [ ] Configure Wi-Fi and NTP.
- [ ] Select the correct timetable by day of week.
- [ ] Download and parse the selected JSON.
- [ ] Initialise both LCDs.
- [ ] Display upcoming trains and destinations.
- [ ] Calculate estimated arrival times.
- [ ] Implement countdown and scheduled-arrival transitions.
- [ ] Implement button debouncing and delay recording.
- [ ] Implement next-train advancement.
- [ ] Implement four LED indicators.
- [ ] Handle network and JSON errors safely.

### Phase 4 — Functional testing

- [ ] Test both LCDs simultaneously.
- [ ] Verify the correct train and destination on each display.
- [ ] Test weekday, Saturday and Sunday selection.
- [ ] Verify the midnight and day-transition cases.
- [ ] Test arrival countdown and station estimation.
- [ ] Test delay recording with the button.
- [ ] Verify that one press registers only once.
- [ ] Test the four LED states.
- [ ] Test Wi-Fi disconnection and recovery.
- [ ] Confirm that invalid timetable data does not crash the system.

### Phase 5 — Enclosure and final demonstration

- [ ] Measure the actual LCD modules before cutting openings.
- [ ] Cut openings for both LCDs and the push button.
- [ ] Add four LED mounting holes.
- [ ] Provide USB access at the side or rear.
- [ ] Secure the ESP32, level shifter and breadboard.
- [ ] Route and secure the internal wires.
- [ ] Test the system before closing the enclosure.
- [ ] Complete a full demonstration with both directions operating.

## 13. Known Limitations

- The prototype uses timetable data rather than live train tracking.
- Estimated arrivals depend on the configured station travel intervals.
- Actual train delays are not automatically obtained from metro operational systems.
- Delay recording depends on the operator's button press and the firmware's timing logic.
- Timetable updates require valid JSON files to be available at the configured GitHub URLs.
- Wi-Fi and NTP availability affect automatic timetable selection and timekeeping.

## 14. Future Improvements

- Add a reliable offline fallback timetable.
- Store delay events with timestamps for later analysis.
- Improve error messages when Wi-Fi or GitHub is unavailable.
- Add a configuration page for updating Wi-Fi credentials.
- Add a web dashboard for viewing recorded delays.
- Improve the enclosure with a more durable panel and mounting hardware.

## 15. Project Summary

This project demonstrates an embedded passenger-information display using an ESP32, I²C LCDs, structured JSON timetable data, network time synchronisation, estimated arrival calculations, a physical operator input and LED status indicators.

The aim is to build a compact, understandable and maintainable prototype that models selected functions of a metro passenger-information system.

**Project status:** Hardware foundation and timetable preparation are underway. The remaining work is to finish the input and LED wiring, integrate and validate the firmware, test the complete system, and assemble the enclosure.
