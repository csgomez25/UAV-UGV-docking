# Team Roles & Interfaces

> **Docking (2026-09-10 →):** see [Docking roles](#docking-roles-2026-09-10--proposed-confirm-at-the-first-meeting) directly below — confirm the assignment at the first project meeting.
>
> **GPS-denied phase (background):** the layer below describes how the original
> 2-person team split the work — **State Estimation/VIO + autonomy-loop integration**
> on one side, **hardware** on the other. It's kept as reference for anyone picking up
> that track; the current docking assignment is what governs day to day.
>
> **Team evolution:** the project started as **2 people** (software/autonomy + structural/
> electrical) over summer 2026, and is growing into a full senior-design team for the
> docking project — adding a planning teammate and a control teammate as the course starts.
>
> This doc pins the **interface contracts** between roles — the seams are where modular teams succeed or bleed time.
>
> Companion docs: [README.md](./README.md) · [BUILD.md](./BUILD.md) · [ROADMAP.md](./ROADMAP.md) · [archive/SUMMER.md](./archive/SUMMER.md)

---

## Docking roles (2026-09-10) — *proposed, confirm at the first meeting*

The senior project is now cooperative UAV–UGV docking ([`docking/`](./docking)). The
split below replaces the GPS-denied data flow for the critical path; the sections after it
still describe the GPS-denied layer.

```
 [UGV]                      [Autonomy]                                          [Flight control]
 encoders+IMU ─▶ /ugv/broadcast ─┐
                                 ├─▶ relative-state estimator ─▶ docking controller ─▶ PX4 (offboard)
 downward camera ─▶ tag pose ────┘       (KF, latency comp.)     (chase | coop)
                                                                          │
 [Test / ground truth] ◀── touchdown truth, contact speed, 20-trial matrix ┘
```

| Role | Owns | Produces |
|---|---|---|
| **Autonomy** | Tag relative pose, relative-state estimator, both controllers, integration (topics, frames, launch) | Relative state; setpoints to PX4 |
| **UGV** | The ground robot: repeatable speed, the velocity/heading broadcast, the pad | `/ugv/broadcast` at an agreed rate, frame and timestamp |
| **Hardware** | X500 build, downward camera + rangefinder mounts, vibration isolation, safety items | A manually-flyable airframe with the docking sensors |
| **Flight control** | PX4 config, failsafes, kill switch, land/disarm behaviour on a moving pad | Reliable base flight (the scoring half of Rule 6) |
| **Test / ground truth** | Sim worlds, the trial harness, touchdown-truth method, lab notebook | The distributions — and the evaluator's tests |

With fewer people: UGV + hardware merge, and test folds into autonomy for the Fall sim study.

**Seams to write down before coding:**
1. **UGV → autonomy:** the broadcast message, frame (ENU world vs. UGV body), rate, and
   whose clock stamps it. Latency is a variable in the study, so the stamp is not optional.
2. **Camera → autonomy:** tag family and size, the tag-bundle layout (one file:
   the code repo's `docking/config/pad_layout.yaml`), camera→body transform, and the lens —
   D0 shows the lens decides the approach altitude.
3. **Autonomy → flight control:** velocity vs. position setpoints, and who decides touchdown
   and disarm (PX4's land detector cannot be relied on over a moving pad — see the code
   repo's `DOCKING.md`).
4. **Everyone → test:** what counts as a successful dock (pad tolerance), fixed before the
   first moving trial and never changed after.

---

# GPS-denied layer roles (summer 2026)

---

## The data flow (who owns each box)

```
         [Hardware]                    [VIO / Integration]        [Planning]              [Control]
 RealSense ───────────────▶ cam/IMU ─▶ cuVSLAM ─▶ EKF2 ─▶ pose/odom ─▶ nvblox map ─▶ A* ─▶ Path ─▶ traj+track ─▶ PX4
   (mounted, damped,          stream    (calib,    (vision-   + TF        (occupancy   (plan   (way-   (min-snap,
    powered, wired)                      tune)      aided)                  /ESDF)       around  points) tracking)
                                                                                         obstacles)
            └────────── VIO/Integration also owns: the message contracts, TF tree, launch, closing the loop ──────────┘
```

The VIO/Integration role sits at the **front of the autonomy chain** (localization) and
is the **integrator** who makes the whole chain work end-to-end.

---

## State Estimation / VIO lead + autonomy-loop integration  (co-owns planning)

**Owns (primary):**
- **VIO / localization:** integrate + calibrate (Kalibr) + tune **cuVSLAM** on the RealSense; configure PX4 **EKF2** for GPS-denied / vision-aided; feed VIO odometry in; **characterize drift**.
- **Autonomy-loop integration (software architect):** the ROS 2 workspace, node graph, TF tree, launch files, and — most importantly — **owning the message contracts** between every box. This is the person who makes localization → planning → control actually work as one loop on hardware.

**Co-owns (shared, hands off as the team grows):**
- **Planning + obstacle avoidance** — built in sim first, then handed to a planning teammate while staying the integration owner.

**Learn:** ROADMAP.md's VIO development track (Labbe → OpenVINS/EuRoC → toy VI-EKF → Kalibr → cuVSLAM).
**Produces:** reliable pose/odom + `map→base_link` TF. **Consumes:** sensor stream (from Hardware), and eventually a `nav_msgs/Path` from the planning role.

---

## Hardware — Structural + Electrical  *(confirmed, from summer 2026)*

**Owns — Mechanical:** X500 assembly; payload mounts (camera, Jetson tray, battery, prop guards); **vibration isolation for camera/IMU** — *the make-or-break detail; bad damping silently wrecks VIO*; cooling; weight budget; CAD for custom mounts (and the eventual custom-frame stretch).

**Owns — Electrical:** power distribution, wiring, soldering, ESC/motor, battery management, telemetry + ELRS RC link, sensor power/data cabling, kill-switch wiring.

**Delivers:** a powered, sensor-mounted, manually-flyable platform with clean vibration isolation. **Needs from Integration:** compute/sensor specs, mounting + vibration constraints, cabling needs.
**Learn:** Oscar Liang (hardware/soldering/ELRS), PX4 assembly docs, Holybro X500 build guide.

---

## Planning & Obstacle Avoidance  *(joins as the team grows; co-owned, then handed off)*

**Owns:** nvblox occupancy/ESDF consumption → A*/D* planning → obstacle inflation → mid-flight replanning → `nav_msgs/Path` output. Inherits a working sim version as a starting point.
**Consumes:** pose + TF + map (from Integration). **Produces:** collision-free `Path` (to Control).
**Learn:** ROADMAP Phase-1 planning track (LaValle, nvblox docs).

---

## Flight Control & Autopilot

**Owns:** PX4 firmware config, flight modes, failsafes, **kill switch + geofence**; turning the planner's `Path` into **min-snap trajectories** + **tracking** (owns the `offboard_manager`); PID tuning for the real all-up weight; uXRCE-DDS / MAVLink link health.
**Consumes:** the `Path`. **Produces:** tracked flight.
**Learn:** ROADMAP Phase-1 control track (UPenn Aerial Robotics, min-snap paper, Tedrake), PX4 PID-tuning docs.

---

## Test / Sim / Integration  *(a dedicated person, or shared)*

**Owns:** Gazebo worlds + obstacle courses; evaluation metrics (success rate, closest-approach margin, replan latency, drift); logging/analysis (ulog + rosbag + PlotJuggler); flight-test safety procedures; documentation.
> If no dedicated person: sim worlds → Integration; metrics/logging → Control; safety procedures → Hardware.

---

## The seams to nail in writing

**1. Sensor stream (Hardware ↔ Integration).** VIO needs tight camera–IMU time-sync; the planning role needs depth. Agree **one shared RealSense launch** (owned by Integration) and what topics/rates it publishes.

**2. Localization → Planning (Integration ↔ Planning).** Integration publishes pose/odom + `map→base_link` TF in a defined frame (recommend ENU `map`). Pin the topic, frame, and update rate the planner can rely on, plus a **drift characterization** so the planner knows how much to trust the pose over distance.

**3. Planning → Control (Planning ↔ Control).** The `Path`/trajectory handoff: exact message (`nav_msgs/Path`), frame, waypoint acceptance radius, and mid-flight replan behavior. Integration arbitrates this even when it doesn't own either side.

> Write these contracts down before anyone codes. 90% of integration pain is an undocumented frame convention or topic mismatch at exactly these seams.

---

## Scaling the team

| Team size | Split |
|---|---|
| **Summer = 2** | **Software:** all software (VIO + planning + integration, in sim) · **Hardware:** assembly → manual flight → mounts/vibration. |
| **Fall = 3** | **A:** VIO lead + integration (+ co-owns planning) · **B:** planning **+** control · **C:** hardware + test. |
| **Fall = 4** | **A:** VIO lead + integration · **B:** planning · **C:** control · **D:** hardware (+ test folded in or a 5th person). |

Merge *roles*, never blur *interfaces* — the contracts above hold at every size. The
summer head-start means the software side can **lead** in the fall instead of scrambling.
