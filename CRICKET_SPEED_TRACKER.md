# 🏏 Cricket Ball Speed Tracking System

> A portable, hardware-powered system that measures the speed of a cricket ball in real-time and displays it on your phone — no manual guesswork, no cheating.

---

## 📖 Table of Contents

1. [What Is This System?](#-what-is-this-system)
2. [How It Works — The Big Picture](#-how-it-works--the-big-picture)
3. [Components Overview](#-components-overview)
4. [Hardware Details](#-hardware-details)
5. [Wiring and Connections](#-wiring--connections)
6. [Physical Setup and Field Placement](#-physical-setup--field-placement)
7. [How the Ball Speed is Detected](#-how-the-ball-speed-is-detected)
8. [ESP32 Firmware Logic](#-esp32-firmware-logic)
9. [Bluetooth BLE Communication](#-bluetooth-ble-communication)
10. [Mobile App Flow](#-mobile-app-flow)
11. [Full Delivery Walkthrough](#-full-delivery-walkthrough-one-ball-start-to-finish)
12. [What If Something Goes Wrong](#-what-if-something-goes-wrong)
13. [Calibration and Performance](#-calibration--performance)
14. [Hardware Validation Checklist](#-hardware-validation-checklist)
15. [Approximate Cost](#-approximate-cost)
16. [Current Limitations](#-current-limitations)

---

## 🧠 What Is This System?

This is a **Cricket Ball Speed Tracking System** built for **box cricket matches**.

In box cricket, there is no umpire or official equipment to measure bowling speed. Players often argue about ball speed, and manual judgement is unreliable. This system solves that.

### What it does

- Measures the **exact speed of a cricket ball** as it is bowled
- Sends the speed **wirelessly** to a mobile phone
- Displays the result **within ~1 second** of the ball passing the sensor
- Keeps a **history** of all deliveries in the match

### What it uses

| Component | Purpose |
|---|---|
| 24GHz Radar Sensor | Detects ball movement and measures speed |
| ESP32 Microcontroller | Processes radar data and sends it via Bluetooth |
| Bluetooth Low Energy (BLE) | Wireless communication to the phone |
| Flutter Mobile App | Displays speed on your phone |
| Mi Power Bank 20,000mAh | Powers everything portably |

---

## 🔁 How It Works — The Big Picture

Think of it like this:

```
Bowler bowls the ball
         |
         v
Radar Sensor detects the ball and measures its speed
         |
         v
ESP32 receives, filters, and processes the speed data
         |
         v
ESP32 sends the result via Bluetooth (BLE)
         |
         v
Flutter App on your phone shows the speed
```

### Simplified Flow Diagram

```
+------------+    UART     +----------+     BLE      +--------------+
|   RADAR    | ----------> |  ESP32   | -----------> | FLUTTER APP  |
| HLK-LD303  | (raw speed) | processes|  (final km/h)|  (displays   |
|            |             | &filters)|              |  on screen)  |
+------------+             +----------+              +--------------+
      ^
      |
   Cricket Ball
 (flying toward radar)
```

---

## 📦 Components Overview

| # | Component | Model | Role |
|---|---|---|---|
| 1 | 24GHz Radar Sensor | HLK-LD303 | Detects ball and generates speed |
| 2 | ESP32 Microcontroller | DevKit V1 | Brain — processes and sends data |
| 3 | Jumper Wires | Male-to-Female / Male-to-Male | Connects radar to ESP32 |
| 4 | Power Bank | Mi 20,000mAh | Powers radar and ESP32 |
| 5 | Tripod / Mount | Any standard tripod | Holds the radar at correct height |
| 6 | Logic Level Shifter | BSS138 / TXS0102 | Only needed if radar uses 5V logic |
| 7 | 100uF Capacitor | Electrolytic | Stabilizes power line |
| 8 | Enclosure | Optional | Protects ESP32 and wiring |

---

## 🔩 Hardware Details

### 📡 24GHz Radar Sensor — HLK-LD303

> **In simple words:** This is the "eye" of the system. It shoots out invisible radio waves, they bounce off the cricket ball, and come back. By measuring the frequency change of the returning wave, it calculates how fast the ball is moving. This is the same principle used in traffic speed guns.

#### How it detects speed

```
Radar emits radio wave
         |
         v
Wave hits the cricket ball
         |
         v
Ball reflects the wave back
         |
         v
Frequency of the reflected wave is slightly different
         |
         v
Radar calculates speed from that difference (Doppler Effect)
         |
         v
Sends speed value to ESP32 via UART
```

#### Key Specifications

| Spec | Value |
|---|---|
| Operating Frequency | 24 GHz |
| Operating Voltage | 5V DC |
| UART Logic Level | 3.3V — directly compatible with ESP32 |
| Current Consumption | 150 to 300 mA |
| UART Baud Rate | 115200 bps |
| Data Format | 8 data bits, No parity, 1 stop bit |
| Detection Range | Up to 5 to 8 meters |
| Output | Speed values in km/h or m/s via serial |
| Operating Temperature | -20 to +60 degrees Celsius |

#### Why HLK-LD303?

- Outputs speed directly — no complex math needed on ESP32
- 3.3V UART logic — plugs directly into ESP32 without extra parts
- 5 to 8 meter range — perfect for a box cricket pitch
- Compact and easy to mount on a tripod

> **Important for the engineer:** Verify the UART TX voltage of the exact module you receive. If it outputs 5V logic, you must add a logic level shifter to protect the ESP32 GPIO pins. ESP32 is NOT 5V tolerant.

---

### 🧠 ESP32 Microcontroller

> **In simple words:** This is the "brain" of the system. It receives the raw numbers from the radar, cleans up the data (removes noise from bat swings or player movement), picks the highest valid speed, and sends it wirelessly to your phone.

#### What it does step by step

```
Receive raw speed data from radar via UART
         |
         v
Read and parse incoming values
         |
         v
Filter out noise (bat swings, player movement, random triggers)
         |
         v
Detect a valid cricket ball delivery
         |
         v
Find the maximum speed during the delivery
         |
         v
Send result via Bluetooth BLE to Flutter app
         |
         v
Wait 3 seconds cooldown before ready for next ball
```

#### GPIO Pin Mapping

| ESP32 Pin | Function | Connected To |
|---|---|---|
| GPIO16 — RX2 | UART2 Receive | Radar TX |
| GPIO17 — TX2 | UART2 Transmit | Radar RX |
| VIN 5V | Power Output | Radar VCC |
| GND | Common Ground | Radar GND |

> Always use UART2 on GPIO16 and GPIO17. Do NOT use UART0 on GPIO1 and GPIO3 — that is reserved for USB debug. Do NOT use GPIO6 through GPIO11 — those are internally connected to the flash memory.

---

### 🔋 Power Supply

> **In simple words:** One power bank powers everything. Simple, portable, no extra wires or adapters needed.

```
Mi Power Bank 20,000mAh
         |
         |-----> ESP32 (via micro-USB cable)
         |
         `-----> Radar Sensor (via ESP32 VIN 5V pin)
```

#### Estimated Power Consumption

| Component | Current Draw |
|---|---|
| ESP32 | 80 to 250 mA |
| Radar Sensor | 100 to 300 mA |
| **Total** | **180 to 550 mA** |

#### Estimated Battery Life

- **12 to 20 hours** depending on radar activity and BLE usage
- A 20,000mAh power bank is more than enough for a full day of matches

---

## 🔌 Wiring & Connections

### Connection Diagram

```
+-----------------------------------------------+
|               WIRING DIAGRAM                  |
|                                               |
|  RADAR (HLK-LD303)       ESP32 DevKit V1      |
|  +----------------+      +----------------+  |
|  |                |      |                |  |
|  |  TX  ----------+------+--> GPIO16 RX2  |  |
|  |                |      |                |  |
|  |  RX  ----------+------+--> GPIO17 TX2  |  |
|  |                |      |                |  |
|  |  VCC ----------+------+--> VIN (5V)    |  |
|  |                |      |                |  |
|  |  GND ----------+------+--> GND         |  |
|  |                |      |                |  |
|  +----------------+      +-------+--------+  |
|                                  |           |
|                             [USB Port]       |
|                                  |           |
|                          Mi Power Bank       |
|                          (20,000mAh)         |
+-----------------------------------------------+
```

### If Logic Level Shifter Is Needed (only if radar UART is 5V)

```
Radar TX (5V)
      |
      v
  [LV pin] --- Level Shifter (BSS138 / TXS0102) --- [HV pin]
               Shifter HV  --> 5V
               Shifter LV  --> 3.3V (ESP32 3.3V pin)
               Shifter GND --> Common GND
      |
      v
  ESP32 GPIO16 RX2 (receives safe 3.3V signal)
```

### UART Initialization Code for ESP32

```
Serial2.begin(115200, SERIAL_8N1, 16, 17);
```

| Setting | Value |
|---|---|
| UART Port | Serial2 — UART2 |
| RX Pin | GPIO16 |
| TX Pin | GPIO17 |
| Baud Rate | 115200 |
| Data Bits | 8 |
| Parity | None |
| Stop Bits | 1 |
| Buffer Size | 256 bytes recommended |

---

## 📐 Physical Setup & Field Placement

### Where to Place the Radar

> **The radar must face the bowler, placed directly behind the batter's wicket.**

```
         BOX CRICKET FIELD — TOP VIEW
___________________________________________________
|                                                 |
|   [BOWLER]                                      |
|      |                                          |
|      |   <------- 10 to 14 meters pitch ------> |
|      |                                          |
|      v  (ball delivery direction)               |
|      v                                          |
|      v                                          |
|   ═══════════                                   |
|   ║ STUMPS ║  <-- Wicket line                   |
|   ═══════════                                   |
|                                                 |
|   <---- 1 to 2 meters behind wicket ---->       |
|                                                 |
|   +-------------+                               |
|   |  RADAR      |  <-- Faces toward bowler      |
|   |  HLK-LD303  |                               |
|   +------+------+                               |
|          |                                      |
|   +------+------+                               |
|   |   ESP32     |                               |
|   | + PowerBank |                               |
|   +-------------+                               |
|   [TRIPOD STAND]                                |
|_________________________________________________|

Ball travel direction:  Bowler ----> Radar
```

### Side View (Elevation)

```
BOWLER
  |
  |  (ball flying low, at stump height)
  |
  v
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ (pitch ground)

                         +---------------------+
  <-- 1-2m from wicket   |   RADAR SENSOR      | <-- Mount at stump
                         |   (faces bowler)    |     height
                         |                     |     65 to 75 cm
                         +----------+----------+     from ground
                                    |
                         +----------+----------+
                         |      ESP32          |
                         |   + Power Bank      |
                         +----------+----------+
                                    |
                               TRIPOD LEGS
                              /     |     \
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ (ground)
```

### Why This Placement?

| Placement Rule | Reason |
|---|---|
| Radar behind batter, facing bowler | Ball travels directly toward radar — strongest signal return |
| Radar at stump height 65 to 75 cm | Aligns with the actual path of the ball |
| 0 degree angle — perfectly horizontal | Any tilt creates angular error and reduces accuracy |
| 1 to 2 meters behind wicket | Safe distance while still within detection range |

### What NOT to Do

```
 Radar placed to the SIDE of the pitch:

        Radar
          |
          |   (not facing the ball)
          |
Ball ----->

 This creates angular error and inaccurate speed readings.
 Always keep radar directly in the ball's travel path.
```

### Mounting Assembly — Step by Step

| Step | Action |
|---|---|
| Step 1 | Set up tripod so radar center sits 65 to 75 cm from ground |
| Step 2 | Place tripod 1 to 2 meters directly behind the batter wicket |
| Step 3 | Mount radar on tripod head with antenna facing exactly toward the bowler at 0 degrees tilt |
| Step 4 | Attach ESP32 and power bank to tripod leg using velcro strap |
| Step 5 | Connect all wires as per the wiring diagram |
| Step 6 | Ensure no wire crosses the ball travel path |
| Step 7 | Fully charge power bank before the match starts |

---

## 🎯 How the Ball Speed is Detected

> **The radar does not know cricket rules.** It simply detects moving objects and measures their speed. The ESP32 decides whether it is a cricket ball delivery, a bat swing, or a fielder moving.

### What the Radar Sees During a Delivery

```
Time 1 -->   0 km/h   (no movement — waiting)
Time 2 -->  65 km/h   (ball entering detection zone)
Time 3 -->  98 km/h   (ball approaching closer)
Time 4 --> 116 km/h   (ball at peak speed near radar)
Time 5 --> 113 km/h   (ball just passing)
Time 6 --> 100 km/h
Time 7 -->  60 km/h   (ball moving away)
Time 8 -->   0 km/h   (ball gone)
```

### What ESP32 Does With This Data

```
ESP32 collects: [65, 98, 116, 113, 100, 60]
                              ^
                       Maximum = 116 km/h
                       This is sent to the app
```

### How Bat Swings and Player Movement Are Filtered Out

| Movement Type | Typical Speed | ESP32 Decision |
|---|---|---|
| Cricket ball delivery | 70 to 150 km/h | Track and report |
| Bat swing | 30 to 50 km/h | May trigger — raise threshold to 40 km/h after calibration |
| Player walking | 5 to 10 km/h | Ignored — below 20 km/h threshold |
| Random noise spike | Unpredictable | Filtered by duration checks and range limits |

---

## ⚙️ ESP32 Firmware Logic

### Filtering Configuration Values

| Setting | Value | Purpose |
|---|---|---|
| Minimum valid speed | 20 km/h | Ignores slow objects like players or bat at rest |
| Maximum valid speed | 200 km/h | Ignores random noise spikes above this |
| Tracking start threshold | 20 km/h | Begins a tracking session when crossed |
| Tracking stop threshold | 10 km/h | Ends the tracking session when speed drops here |
| Cooldown after delivery | 3000 ms | Ignores all readings for 3 seconds after result is sent |
| Max tracking duration | 2000 ms | If object is tracked longer than 2s it is not a ball |
| Min tracking duration | 100 ms | If event is shorter than 100ms it is noise — discard |
| UART read interval | 50 ms | How often ESP32 reads radar data |

### Firmware Decision Flowchart

```
START
  |
  v
Read speed from radar every 50ms
  |
  v
Speed > 20 km/h ?
  |
  |-- NO --> Keep waiting, read again
  |
  `-- YES --> Start tracking session
                |
                v
            Collect all incoming speed values into buffer
                |
                v
            Speed drops below 10 km/h ?
                |
                |-- NO, but time > 2000ms --> DISCARD (not a ball, too slow)
                |
                `-- YES --> End tracking session
                                |
                                v
                            Tracking duration > 100ms ?
                                |
                                |-- NO --> DISCARD (too fast, noise)
                                |
                                `-- YES --> Extract MAX speed from buffer
                                                |
                                                v
                                            Send MAX speed via BLE
                                                |
                                                v
                                            Start 3000ms cooldown
                                                |
                                                v
                                            After cooldown --> READY again
```

---

## 📶 Bluetooth BLE Communication

> **In simple words:** The ESP32 acts like a Bluetooth device. Your phone finds it, connects to it, and subscribes to receive speed updates — similar to subscribing to push notifications.

### BLE Device Identity

| Property | Value |
|---|---|
| BLE Device Name | CricketSpeedTracker |
| Service UUID | 4FAFC201-1FB5-459E-8FCC-C5C9C331914B |
| Characteristic UUID | BEB5483E-36E1-4688-B7F5-EA07361B26A8 |
| Characteristic Mode | NOTIFY and READ |

> These UUIDs must match exactly in both the ESP32 firmware and the Flutter app code. Do not change them unless you update both sides together.

### BLE Data Packet Sent After Each Delivery

```
{
  "speed": 116,
  "unit": "km/h",
  "timestamp": "1715702010",
  "delivery": 3,
  "match": "Friday Match"
}
```

### BLE Connection Flow

```
ESP32 powers on
      |
      v
Auto-starts BLE advertising
Device name shown: CricketSpeedTracker
      |
      v
Flutter app scans for nearby BLE devices
      |
      v
User selects CricketSpeedTracker from the list
      |
      v
BLE connection established
      |
      v
Flutter subscribes to the speed characteristic
      |
      v
Every time a ball is bowled:
ESP32 sends BLE notification --> Flutter receives --> Speed shown on screen
```

### BLE Reconnection Flow (If Phone Disconnects)

```
Flutter app disconnects
      |
      v
ESP32 detects the disconnect event
      |
      v
ESP32 automatically restarts BLE advertising
No physical reset required
      |
      v
Radar continues working in the background
No speed data is lost
      |
      v
Flutter reconnects --> resumes receiving speed data normally
```

### Delivery-by-Delivery BLE Data Stream

```
Delivery 1:
ESP32 --> BLE --> { "speed": 122, "unit": "km/h", "delivery": 1 }
App shows: 122 km/h

  (3 second cooldown)

Delivery 2:
ESP32 --> BLE --> { "speed": 108, "unit": "km/h", "delivery": 2 }
App shows: 108 km/h   replaces 122

  (3 second cooldown)

Delivery 3:
ESP32 --> BLE --> { "speed": 134, "unit": "km/h", "delivery": 3 }
App shows: 134 km/h   replaces 108
```

---

## 📱 Mobile App Flow

### All App Screen States

```
+--------------------------------------------------+
|  STATE 1 — WAITING FOR DELIVERY                  |
|  +--------------------------------------------+  |
|  |  Connected — CricketSpeedTracker           |  |
|  |                                            |  |
|  |           READY                            |  |
|  |    Waiting for delivery...                 |  |
|  |                                            |  |
|  |    Last Speed: 116 km/h                    |  |
|  +--------------------------------------------+  |
+--------------------------------------------------+

+--------------------------------------------------+
|  STATE 2 — SPEED RECEIVED                        |
|  +--------------------------------------------+  |
|  |  Connected — CricketSpeedTracker           |  |
|  |                                            |  |
|  |           124 km/h                         |  |
|  |                                            |  |
|  |       Ball Speed Detected                  |  |
|  |         Delivery #1                        |  |
|  +--------------------------------------------+  |
+--------------------------------------------------+

+--------------------------------------------------+
|  STATE 3 — COOLDOWN                              |
|  +--------------------------------------------+  |
|  |  Connected — CricketSpeedTracker           |  |
|  |                                            |  |
|  |           124 km/h                         |  |
|  |                                            |  |
|  |    Next ball in 3s...                      |  |
|  |    [==========>    ]  2s remaining         |  |
|  +--------------------------------------------+  |
+--------------------------------------------------+

+--------------------------------------------------+
|  STATE 4 — DISCONNECTED                          |
|  +--------------------------------------------+  |
|  |  Device disconnected — Reconnecting...     |  |
|  |                                            |  |
|  |    Last known speed: 124 km/h              |  |
|  +--------------------------------------------+  |
+--------------------------------------------------+
```

### Speed Update Between Deliveries

```
Previous Ball: 122 km/h       New Ball: 108 km/h

+----------------+            +----------------+
|   122 km/h     |  ------->  |   108 km/h     |
|   (last ball)  |            |   (new ball)   |
+----------------+            +----------------+

122 km/h moves to history list.
108 km/h takes the main display spot.
```

---

## 🏏 Full Delivery Walkthrough — One Ball, Start to Finish

> Let us follow one complete delivery from the moment the bowler releases the ball to the moment the speed appears on the phone screen.

**Scenario:** Friday Box Cricket — Bowler: Ravi — Batter: Suresh

---

### Phase 1 — System Ready

```
+---------------------------------------------+
|  Power Bank: ON                             |
|  ESP32: Running                             |
|  Radar: Active, scanning continuously       |
|  BLE: Connected to Flutter app              |
|  Current radar reading: 0 km/h             |
+---------------------------------------------+

App Screen:
+---------------------------------------------+
|  Connected — CricketSpeedTracker            |
|                                             |
|           READY                             |
|    Waiting for delivery...                  |
|                                             |
|    Last Speed: 116 km/h                     |
+---------------------------------------------+
```

---

### Phase 2 — Ravi Bowls the Ball (T = 0 ms)

Ball leaves Ravi's hand and flies toward the batter at high speed.
The radar has not detected it yet — the ball is still far from the sensor.

```
App Screen: (no change)
+---------------------------------------------+
|  Connected — CricketSpeedTracker            |
|           READY                             |
|    Waiting for delivery...                  |
|    Last Speed: 116 km/h                     |
+---------------------------------------------+
```

---

### Phase 3 — Ball Enters Radar Detection Zone (T = 300 ms approx)

Ball approaches the radar. Radio waves reflect back. ESP32 starts collecting speed values.

```
Radar raw output streaming to ESP32:
  "45"  -->  45 km/h
  "89"  -->  89 km/h
  "112" --> 112 km/h
  "124" --> 124 km/h

+---------------------------------------------+
|  ESP32: Speed exceeded 20 km/h             |
|  Tracking session STARTED                   |
|  Buffer so far: [45, 89, 112, 124]          |
+---------------------------------------------+

App Screen: (still no change — ball still in flight)
```

---

### Phase 4 — Ball Passes the Radar (T = 550 ms approx)

Ball passes the sensor and moves away. Speed readings start dropping.

```
Radar raw output:
  "119" --> 119 km/h
  "98"  -->  98 km/h
  "61"  -->  61 km/h
  "8"   -->   8 km/h  <-- dropped below 10 km/h

+---------------------------------------------+
|  ESP32: Speed dropped below 10 km/h        |
|  Tracking session ENDED                     |
|  Full buffer: [45, 89, 112, 124, 119,       |
|               98, 61, 8]                    |
|  Maximum value found: 124 km/h             |
|  Tracking duration: ~250ms — VALID          |
+---------------------------------------------+
```

---

### Phase 5 — ESP32 Sends Speed via BLE (T = 600 ms approx)

```
BLE Packet sent to Flutter app:
{
  "speed": 124,
  "unit": "km/h",
  "timestamp": "1715702310",
  "delivery": 1
}

ESP32: 3000ms cooldown STARTED
Radar input is now ignored during cooldown
```

---

### Phase 6 — App Displays the Speed (T = 650 ms approx)

```
App Screen:
+---------------------------------------------+
|  Connected — CricketSpeedTracker            |
|                                             |
|            124 km/h                         |
|                                             |
|        Ball Speed Detected                  |
|           Delivery #1                       |
+---------------------------------------------+

Speed 124 km/h saved to delivery history.
Previous speed 116 km/h moved to history list.
```

---

### Phase 7 — Cooldown Period (T = 600 ms to 3600 ms)

```
+---------------------------------------------+
|  ESP32: Cooldown active                     |
|  All radar readings ignored                 |
|  Reason: prevents bat swing or fielder      |
|  movement from triggering a false reading   |
+---------------------------------------------+

App Screen: Speed stays on screen with optional countdown timer
```

---

### Phase 8 — System Resets, Ready for Next Ball (T = 3600 ms)

```
+---------------------------------------------+
|  ESP32: Cooldown finished                   |
|  Radar listening again                      |
|  Ready for Delivery #2                      |
+---------------------------------------------+

App Screen:
+---------------------------------------------+
|  Connected — CricketSpeedTracker            |
|                                             |
|           READY                             |
|    Waiting for delivery...                  |
|                                             |
|    Last Speed: 124 km/h                     |
+---------------------------------------------+
```

---

### Complete Timeline Summary

| Time | Event |
|---|---|
| T = 0 ms | Bowler releases ball |
| T = 300 ms | Ball enters radar zone, ESP32 starts tracking |
| T = 550 ms | Ball passes radar, tracking session ends |
| T = 600 ms | ESP32 finds max speed 124 km/h and sends BLE packet |
| T = 650 ms | Flutter app receives speed and updates screen |
| T = 650 ms | 3-second cooldown begins on ESP32 |
| T = 3650 ms | Cooldown ends, system ready for next delivery |

> **Total time from ball release to phone screen: approximately 0.65 seconds**
> **Total time before system is ready for next ball: approximately 3.65 seconds**

---

## ⚠️ What If Something Goes Wrong?

| Scenario | What Happens | How It Is Handled |
|---|---|---|
| Bat swing triggers the system | Bat swing at 35 km/h is detected and reported | Raise minimum threshold to 40 km/h after calibration |
| BLE disconnects mid-match | ESP32 detects disconnect and restarts advertising automatically | Radar keeps running in background — Flutter reconnects and resumes. No reset needed. |
| Ball hits the radar directly | Radar reads a spike above 200 km/h | ESP32 ignores any value above 200 km/h — nothing sent to app |
| Power bank runs out | ESP32 shuts down, BLE disconnects | App shows Device disconnected — recharge power bank and reconnect |
| Multiple people moving near radar | Multiple low-speed detections | Filtered out by the 20 km/h minimum threshold |

---

## 🎛️ Calibration & Performance

### Calibration Steps

1. Place radar at the correct field position — behind batter, facing bowler, at stump height
2. Bowl a few test deliveries
3. Compare speeds shown on the app with manual estimates or a reference speed gun
4. Adjust filtering threshold values in firmware if readings seem off
5. Repeat test deliveries until readings are stable and consistent

### Performance Targets

| Metric | Target |
|---|---|
| Minimum detectable speed | 20 km/h |
| Maximum expected speed | 150 km/h for box cricket |
| Accuracy after calibration | plus or minus 2 to 3 km/h |
| Acceptable error margin | plus or minus 5 km/h |
| Latency from ball to screen | Under 1 to 2 seconds |
| Detection range | 5 to 10 meters |

---

## ✅ Hardware Validation Checklist

> For the electrical engineer to complete before handing over to the software team.

### Step 1 — Power Check
- [ ] Radar powers on from 5V — verify with multimeter
- [ ] ESP32 powers on from VIN 5V — verify with multimeter
- [ ] Both radar and ESP32 share a common GND
- [ ] Total current draw is within 180 to 550 mA range

### Step 2 — UART Communication Check
- [ ] Connect ESP32 to PC via USB
- [ ] Open Serial Monitor at 115200 baud
- [ ] Verify raw speed packets are being received from radar
- [ ] Data format matches expected output — speed values are readable
- [ ] No garbage data — if garbage appears, check baud rate or logic level

### Step 3 — Logic Level Check
- [ ] Measure radar TX voltage with multimeter
- [ ] If voltage is greater than 3.3V, install a logic level shifter
- [ ] After shifter is installed, re-measure and confirm 3.3V at ESP32 RX pin

### Step 4 — BLE Check
- [ ] Flash basic BLE advertise firmware to ESP32
- [ ] Open nRF Connect app on mobile phone
- [ ] Confirm CricketSpeedTracker appears in the device scan list
- [ ] Confirm BLE service and characteristic UUIDs are visible and correct

### Step 5 — End-to-End Check
- [ ] Flash full firmware to ESP32
- [ ] Connect Flutter app via BLE
- [ ] Move a hand or object in front of radar
- [ ] Confirm speed value appears on Flutter app screen
- [ ] Simulate a disconnect from Flutter app
- [ ] Confirm ESP32 auto-advertises again without needing a physical reset

---

## 💰 Approximate Cost

| Component | Estimated Price INR |
|---|---|
| 24GHz Radar Sensor HLK-LD303 | 4000 to 6000 |
| ESP32 DevKit V1 | 400 to 800 |
| Jumper Wires | 100 to 200 |
| Logic Level Shifter if needed | 50 to 150 |
| 100uF Capacitor | 10 to 30 |
| Tripod / Mount | 300 to 800 |
| Enclosure Optional | 300 to 800 |
| **Total Approximate** | **5160 to 8780** |

> Power bank is excluded from the estimate as it is already available.

---

## 🚧 Current Limitations

| Limitation | Details |
|---|---|
| Small ball calibration required | Cricket balls are small and fast — initial calibration is important for accuracy |
| Nearby movement can cause noise | People moving close to the radar may occasionally trigger a false reading |
| Placement sensitivity | Even a slightly wrong angle can reduce accuracy |
| Multiple moving objects | If two objects move at the same time, the faster one is picked |
| Real match testing needed | Lab results may differ from actual match conditions |

---

## 📋 Quick Reference Card

```
+------------------------------------------------------------------+
|           CRICKET SPEED TRACKER — QUICK REFERENCE               |
+------------------------------------------------------------------+
|  Radar Model        :  HLK-LD303 (24GHz)                        |
|  Microcontroller    :  ESP32 DevKit V1                           |
|  Communication      :  BLE (Bluetooth Low Energy)               |
|  BLE Device Name    :  CricketSpeedTracker                       |
|  Power Source       :  Mi Power Bank 20,000mAh                   |
+------------------------------------------------------------------+
|  Radar Position     :  Behind batter, facing bowler             |
|  Radar Height       :  65 to 75 cm from ground (stump height)   |
|  Radar Distance     :  1 to 2 meters behind wicket              |
|  Radar Angle        :  0 degrees — perfectly horizontal          |
+------------------------------------------------------------------+
|  Min Speed Threshold   :  20 km/h                               |
|  Max Speed Threshold   :  200 km/h                              |
|  Cooldown Period       :  3000 ms — 3 seconds                   |
|  Detection Latency     :  0.5 to 1 second                       |
|  Target Accuracy       :  +/- 2 to 3 km/h after calibration     |
+------------------------------------------------------------------+
|  UART  :  Serial2 — GPIO16 RX — GPIO17 TX — 115200 baud        |
|  Battery Life  :  12 to 20 hours                                |
|  Total Cost    :  5160 to 8780 INR approx                       |
+------------------------------------------------------------------+
```

---

*Document prepared from Cricket Ball Speed Tracking System project specifications.*
