# Hardware Bring-Up Plan: UGV, Drone and the Link Between Them

*Docking team · written October 30, 2026 · Role: Autonomy*

The simulation shows that the drone lands on a turning pad when the ground robot shares its motion: coop 18 of 20 vs camera-only 0 of 20 on the sine path at 1.0 m/s, in simulation ([STATUS_AND_PLAN.md](./STATUS_AND_PLAN.md)). This plan gets real hardware to the point where that test can be flown outdoors. The UGV comes first, because every physical test needs it.

**Field constraint:** tests run over empty fields or on the school mountain, with **no Wi-Fi or internet**. Every link must be point to point, radio carried on the vehicles, and must work with no network infrastructure.

## 1. What the hardware has to do

These numbers come from the simulation, so the scored comparison can transfer.

| Need | Value | Why |
|---|---|---|
| UGV speed | 1.0 m/s held within ±5%; top speed about 1.3 m/s or more with the deck and a 2 kg stand-in load | The D3 cells fly at 1.0 m/s |
| UGV turning | Arcs of 1 m radius at 1.0 m/s (about 1 rad/s, about 1 m/s² sideways) | The rounded square's corners |
| UGV path following | Line, sine and rounded square repeatably, by itself (scripted, not driven by hand) | Paired trials need the same pad motion each time |
| Broadcast to the drone | Speed, heading, turn rate at **20 Hz**, timestamped; delay measured, about **200 ms or less** | What coop uses; the simulation was scored at 200 ms |
| Deck | 0.6 × 0.6 m, about 0.25 m high, flat, AprilTag bundle on top | Matches the simulated pad |
| Safety | A physical kill switch, a wireless e-stop, and the motors stop by themselves if the link or the computer goes silent | Required before a drone flies over it |

## 2. The system

```
        DRONE                                                  UGV
┌───────────────────────────┐   UGV ↔ drone radio   ┌───────────────────────────────┐
│ Companion computer        │◄─────────────────────►│ Raspberry Pi 5 (ROS 2 Jazzy)  │
│ (Pi 5 or Jetson Orin Nano)│   915 MHz, 20 Hz      │  ugv_broadcaster, path        │
│  tag camera, pad filter,  │   speed/heading/turn  │  follower, logging            │
│  docking_controller       │                       │  IMU · GPS · (UWB later)      │
└────────────┬──────────────┘                       └──────────────┬────────────────┘
             │ serial (uXRCE-DDS)                                  │ USB serial
┌────────────┴──────────────┐                       ┌──────────────┴────────────────┐
│ Pixhawk 6 (PX4)           │                       │ Teensy 4.1: 4 encoders, speed │
│  GPS, own telemetry radio │                       │ loop, watchdog                │
└────────────┬──────────────┘                       └──────────────┬────────────────┘
             │                                                     │
   RC transmitter (safety pilot)                     Cytron MDD10A → left / right motors
   + ground station telemetry (QGroundControl)       wireless e-stop on the motor power
```

**Three separate radio links, each with one job:**

| Link | Hardware | Carries | Why separate |
|---|---|---|---|
| UGV ↔ drone | A pair of **915 MHz telemetry radios** (RFD900x, or Holybro SiK v3 to start) | The coop broadcast: a small fixed packet, about 40 bytes at 20 Hz (under 1 KB/s) | No infrastructure, long range, steady delay. A bridge node on each side turns packets back into the `/ugv/broadcast` ROS 2 topic, so the docking code does not change |
| Drone ↔ ground station | The Pixhawk's own telemetry radio and QGroundControl | Flight status, mode, battery, geofence | Standard PX4 setup, kept apart so docking traffic never competes with it |
| Safety pilot | RC transmitter to the Pixhawk; the UGV's wireless e-stop | Manual takeover and kill | Must work even if every computer fails |

**Use 915 MHz in the US.** It's licence-free ISM. Check each radio's settings before flying: the UGV ↔ drone pair and the ground station pair must be on different network IDs.

**Optional: a battery-powered travel router on the UGV.** It doesn't need internet. It makes its own local network so a laptop can see ROS 2 topics during bench tests and short-range debugging. It's never part of the docking link.

**Clock sync:** the pad filter corrects for the broadcast's delay, so both sides need the same clock. Both vehicles have GPS, so use GPS time on each (PX4 already syncs the companion over uXRCE-DDS; on the UGV, `chrony` with the GPS). Then the delay can be measured from the packet's own timestamp.

## 3. Parts

| Part | Pick | Status | Rough cost |
|---|---|---|---|
| Chassis, tracks, 4 DC gear motors with Hall encoders | Existing tracked tank | **Have** (motor model unknown) | — |
| Motor driver | 2 × L298N (Envistia Mall) for bench tests, then **Cytron MDD10A** | **Have** L298N; MDD10A to order | $25 |
| Microcontroller | Arduino Uno R3 for bench tests, then **Teensy 4.1** (or ESP32-S3) | **Have** Uno; Teensy to order | $30 |
| UGV computer | **Raspberry Pi 5, 8 GB**, with a heatsink fan and a 64 GB card | To order | $100 |
| IMU | **BNO085** (heading and turn rate) | To order | $25 |
| GPS | u-blox M9N module | To order | $40-60 |
| UGV ↔ drone radios | RFD900x pair, or Holybro SiK v3 pair to start | To order | $50-300 |
| Power | 4S LiPo (or the existing pack), 5 V 5 A buck for the Pi, fuse, physical kill switch, wireless e-stop relay | To order | $60-100 |
| Deck | 0.6 × 0.6 m light rigid sheet (polycarbonate or aluminium composite), matte AprilTag print | To make | $30-50 |
| **Drone companion computer** | Pi 5 or Jetson Orin Nano, plus a down-facing camera | **Decision needed** (section 6) | $100-500 |
| UWB (later, mission loop only) | DWM1001 dev boards, one per vehicle | Later | about $30 each |

Prices are rough and must be checked when ordering. The essentials for the UGV come to about **$250-350**.

## 4. Bring-up stages

Each stage ends in a measured pass, like the simulation's gates. Don't start the next until the current one passes, and log every test to a file.

| Stage | What | Pass when | Hardware |
|---|---|---|---|
| **U0** Motor ID | Find the motor model (labels on the can or gearbox). Otherwise measure: rated voltage, encoder pulses per output-shaft turn (turn by hand and count), no-load RPM, drive sprocket diameter | Ticks-to-metres known; predicted top speed = RPM × π × sprocket diameter / 60 | Uno, L298N |
| **U1** Wiring and safety | Each side's motor pair on one H-bridge channel; fuse and kill switch in; each motor spins the right way; each encoder counts the right way | Every motor and encoder checked alone; the kill switch cuts the motors | Uno, L298N |
| **U2** Open-loop speed | Time 5 m at fixed PWM steps, tracks on the ground, with the deck stand-in load | Speed against PWM table filled in; top speed known. **Decides whether the motors reach 1.0 m/s and whether the driver must change** | Uno, L298N |
| **U3** Closed-loop speed | Speed control on the encoders, per side | Holds 0.5 and 1.0 m/s within ±5% over 10 m; stopping distance from 1.0 m/s measured | Teensy, MDD10A |
| **U4** IMU | BNO085 heading and turn rate; compare against encoder turn rate | Turn rate within about 5% of a timed 360° spin; encoder slip in turns measured | Pi or Teensy |
| **U5** Pi and ROS 2 | Pi runs ROS 2 Jazzy: odometry from the Teensy, IMU, `ugv_broadcaster` with real data | `/ugv/broadcast` at 20 Hz on the Pi; the Teensy stops the motors within 0.5 s of the Pi going quiet | Pi 5, Teensy |
| **U6** Paths | The path follower drives line, sine and rounded square at 1.0 m/s | Repeats each path; error measured against GPS or a tape-marked course | Full UGV |
| **U7** Link and clocks | Radios between the UGV Pi and a laptop standing in for the drone; GPS time on both | Delay and dropout measured at 10, 50, 100 and 200 m in the field; compare with the simulation's 200 ms | Radios, GPS |
| **U8** Deck | Mount the deck and tags; check tilt and tipping in the sharpest corner at 1.0 m/s | Deck within 2° of level; no wheel lift; tags decode from 2 m with the drone's camera held by hand | Full UGV |

**Then the joint stages** (the D4/D5 plan, January to April): the drone hovers over the stationary UGV by camera (like H0), lands on it stationary (D1), follows the moving UGV without landing, and finally lands on the moving UGV, chase and coop.

## 5. Week by week

| Week | Goal |
|---|---|
| **Nov 2** | U0 and U1 with the Uno and L298N; strip the chassis; identify the motors |
| **Nov 9** | U2 (top speed). Order the Teensy, MDD10A, Pi 5, BNO085, power parts and radios, now that the motor numbers are known |
| Nov 16-23 | U3 closed-loop speed and U4 IMU, as parts arrive |
| Nov 30 - Dec 7 | U5 Pi and ROS 2; the first real `/ugv/broadcast` |
| **December presentation** | Simulation results, plus the UGV holding 1.0 m/s and broadcasting |
| January | U6 paths, U7 link in the field, U8 deck. The drone's own bring-up starts in parallel |
| Feb-Apr | Joint stages, D4 and D5 |

## 6. Open decisions

1. **The drone's companion computer.** A Pixhawk 6 alone cannot run the tag camera, the pad filter and the docking controller. The drone needs a companion computer connected to the Pixhawk's TELEM2 port. Choose a **Pi 5** (enough for the AprilTag and estimator code, cheaper, the same as the UGV) or a **Jetson Orin Nano** (more headroom for vision). Also confirm which Pixhawk 6 it is (6C or 6X).
2. **The radio pair.** SiK v3 (cheap, fine for the bench and short range) or RFD900x (more range and margin for the mountain tests).
3. **Budget and who orders.**
4. **Field rules.** Permission to fly at each site (school and land owner), FAA rules for the flights, and AMA membership if the team enters C-UASC.
5. **Ground truth for scoring hardware landings:** RTK GPS on both vehicles, or a camera on the deck looking up. Not needed until the joint stages.

## 7. Safety rules for every test

- Bench tests run with the tracks off the ground first.
- The kill switch is within reach on every UGV run. The wireless e-stop works before the UGV drives faster than walking pace.
- The UGV stops by itself if the link or the Pi goes silent (tested in U5, re-tested in the field).
- No drone flies over the UGV until U3, U5 and U8 pass. A safety pilot with an RC transmitter is ready for every flight.
