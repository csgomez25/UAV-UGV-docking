# Cooperative UAV–UGV Docking — Requirements, Parts & Theory

**Project 2 · RSCL@CPP · Role: Autonomy**
Minimum research platform — buy by function, not by catalog. Verify current models/prices before purchase.

Companions: [`ONE_PAGER.md`](./ONE_PAGER.md) · [`FIRST_MEETING_PACKET.md`](./FIRST_MEETING_PACKET.md).

---

## Build strategy: simulate first, then validate

The existing PX4 + ROS 2 (Jazzy) SITL stack runs the **entire cooperative pipeline in Gazebo first** — a simulated UGV publishing velocity, the UAV fusing it with a simulated downward camera, and the full chase-vs-cooperative trial matrix. That is the Fall result. Hardware then tests whether it transfers. Procurement delays don't block the research, and hardware starts knowing what "success" looks like.

## Simulation — what PX4 already provides, and what it doesn't

PX4 (v1.18-alpha, the checkout in use) ships a `moving_platform` world, a `MovingPlatformController` Gazebo plugin (platform speed/heading set by `PX4_GZ_PLATFORM_VEL` / `PX4_GZ_PLATFORM_HEADING_DEG`), and an `x500_mono_cam_down` airframe. That is the starting point, with four gaps:

| Gap | Why it matters | Fix |
|-----|----------------|-----|
| The stock platform is a **5 × 5 m, 2 m tall, 10 t** deck | That is a ship, not a UGV — docking on it proves nothing about a ~50 cm pad | ✅ **Done 2026-09-10:** own world with a 0.6 m pad (deck at 0.30 m) carrying a 5-tag bundle. Static for D0/D1 — the plugin has no horizontal position hold, so even at zero speed its pad wanders |
| The plugin **does not publish platform state** | There is no "UGV broadcast" for the cooperative strategy to use | Add Gazebo's `OdometryPublisher` to the overlay, bridge it to ROS 2, and pass it through a node that adds **latency, noise and dropouts** — otherwise the cooperative arm gets oracle data and the comparison is unfair |
| `mono_cam` renders **1280 × 960** | Gazebo camera memory leaks at frame bandwidth | ✅ **Measured 2026-09-10:** 111.8 MB/s at 1280 × 960, 26.6 MB/s at 640 × 480 — so every camera leaks, 640 × 480 included. Overlay at 640 × 480, short runs on a fresh sim |
| Platform disturbance is **fixed** (hard-coded noise amplitude) | A disturbance axis in the trial matrix needs levels | Fork the plugin if disturbance becomes a matrix axis |

Two PX4 behaviours to design around:
- **Land detection on a moving pad.** `LNDMC_XY_VEL_MAX` defaults to 1.5 m/s: a UAV resting on a pad moving faster than that still looks airborne to PX4. Touchdown must be detected independently (pad contact sensor + truth poses), and disarm handled explicitly.
- **PX4's `NAV_LAND` descends in the world frame** while the pad drives away. Docking stays in offboard control until contact.

**Baseline choice to state up front:** PX4 has in-tree precision-landing estimators (`vision_target_estimator`, `landing_target_estimator`). Either include one as a third comparison arm or say why not — a reviewer will ask.

**Known sim-vs-real gap:** Gazebo cameras are effectively global-shutter. The rolling-shutter penalty on a moving marker (below) only shows up on hardware. The sim also has no depth of field, so the near-touchdown results are geometry, not optics.

**Measured, 2026-09-10 (gate D0):** the tag bundle gives an accurate pose from touchdown to **1.25 m** (≥ 98% of frames, ≤ 1.1 cm and ≤ 0.7° at p95) on a 640 × 480, 100° camera. Above that the 0.18 m tags stop decoding. **The camera choice sets the approach altitude:** a narrower lens (~70°) or ≥ 1280 px roughly doubles it. **PX4 also took up to 61 s to detect a landing on the raised pad, even stationary** — so the land-detection item above applies before the pad ever moves.

---

## UAV (the drone) — minimum viable

| Function | Part (standard choice) | Why / notes |
|----------|------------------------|-------------|
| Flight controller | Pixhawk-class, PX4-compatible (e.g. Pixhawk 6C) | SITL firmware/code transfers directly |
| Airframe + power | ~500-class quad dev kit (frame, motors, ESCs, props, power module, battery) | X500-style PX4 dev kits bundle most of this |
| Companion computer | Raspberry Pi 4/5 **or** Jetson Orin Nano | Jetson if running onboard vision; Pi is enough for the estimator alone |
| **Downward camera** | **Global-shutter** camera (mono is fine), **~70° lens or ≥ 1280 px wide** | Rolling shutter skews/blurs the marker on a moving pad — this matters. Gate D0: at 640 × 480 and 100° the pad's tags decode only to 1.25 m |
| Near-pad altitude | Downward rangefinder (LiDAR: TFmini / VL53L1X-class) | Tight height control over the pad |
| Position hold without GNSS | Optical-flow sensor (PMW3901-class) | For the GPS-denied research layer — the scored flight uses GPS |
| Cooperation + telemetry link | Telemetry radio (SiK 915 MHz) or WiFi/ESP-NOW | Carries UGV velocity broadcast + MAVLink |
| *(Optional, later)* relative ranging | UWB tag (drone side) | Backup/complement to vision for relative position |

## UGV (the moving platform) — minimum viable

| Function | Part (standard choice) | Why / notes |
|----------|------------------------|-------------|
| Mobile base | Differential-drive or skid-steer rover with a flat top plate (~40–60 cm) | Must carry a landing pad and move at a controlled, repeatable speed. **Confirm whether the AVL ground platform is available before buying one** |
| Landing pad marker | Printed AprilTag **bundle**: large tags far, a small centre tag close (the sim pad: 4 × 0.18 m + 1 × 0.10 m on 0.6 m, tag36h11) | Keeps a marker in view from cruise altitude down to touchdown. A bundle rather than nested tags, because OpenCV cannot decode a tag drawn inside another |
| Motion sensing | Wheel encoders + IMU | Source of the velocity/heading the UGV broadcasts |
| Compute | Raspberry Pi / Jetson running ROS 2 | Publishes platform state over the link |
| Cooperation link | Matching radio to the UAV | UGV = publisher, UAV = subscriber |
| *(Optional, later)* relative ranging | UWB anchor on the pad | Pairs with the drone-side tag |

## Ground truth & safety — the part teams forget

| Need | Simulation (Fall) | Hardware minimum | Hardware better |
|------|-------------------|------------------|-----------------|
| **Touchdown-error ground truth** | Gazebo model poses (exact) + pad contact sensor for the touchdown instant | Overhead camera + fixed fiducials over the test area (DIY, cheap) | Motion capture (OptiTrack/Vicon) if the lab has it |
| Platform-velocity truth | Gazebo platform velocity at contact | UGV encoder odometry (log actual speed at contact) | Mocap / RTK |
| Flight safety | — | Netted area **or** tether, prop guards, hardware kill switch | Dedicated indoor cage |

*Without a ground-truth method there are no touchdown-error distributions — so this is a requirement, not an accessory.*

---

## Theory & skills checklist

**A. Vision & relative pose**
- Camera model + intrinsic/extrinsic calibration
- AprilTag/ArUco detection; pose from tag corners via PnP (`solvePnP`)
- Global vs rolling shutter; why it matters on a moving target

**B. State estimation / sensor fusion**
- Kalman / EKF fundamentals
- Fusing visual relative pose + IMU + broadcast platform velocity into one relative state
- Latency compensation and UAV↔UGV time synchronization (~200 ms latency budget as the starting assumption)

**C. Control — the research core**
- Cascaded position/velocity control (PID or LQR)
- Regulating UAV state in the *platform's moving frame*, not the world frame
- **Velocity feedforward** from the UGV broadcast — this is the "cooperation" that separates the project from a static-pad demo
- Landing/descent trajectory and touchdown criteria

**D. Coordinate frames**
- world / body / camera / platform frames and the transforms between them

**E. Flight fundamentals near the pad**
- Ground effect and rotor downwash over the moving vehicle
- Disturbance rejection (wind, platform wake)
- GPS-denied position/altitude hold (optical flow + rangefinder) — research layer

**F. Communications**
- MAVLink; ROS 2 DDS topics/QoS
- Link latency and packet-loss behavior on the cooperation channel

---

## What can start now (zero hardware, zero cost)
1. ✅ UGV-sized, AprilTag-marked pad in a docking world; the downward-camera X500 flies over it (2026-09-10).
2. ✅ Tag relative-pose detection, scored against simulator truth — gate D0, accurate to 1.25 m (2026-09-10). Next: extend the range (lens, resolution or tag size).
3. Bridge the platform's odometry into ROS 2 through a latency/noise layer — the simulated UGV broadcast.
4. Build both controllers: (a) UAV-only chase, (b) cooperative with UGV velocity feedforward.
5. Run the 20-trial matrix in sim across ≥2 platform speeds → first version of the results table.
6. Only then buy hardware to validate the sim-proven result.

---
*As of Sep 10, 2026. Confirm current part models, availability, and prices — and lab safety approval for flight — before spending money.*
