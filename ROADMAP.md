# Learning + Build Roadmap

> Philosophy: **learn just-in-time.** Don't pre-study six months of theory. Learn each concept right before the build step that forces you to use it — you'll retain 10× more. Each phase below pairs *what you build* with *what to learn first* and *where to learn it*.
>
> Companion docs: [README.md](./README.md) (plan going forward) · [BUILD.md](./BUILD.md) (decisions/BOM) · [SIM_WEEK1.md](./SIM_WEEK1.md) (week-1 commands) · [ROLES.md](./ROLES.md) (team) · [SUMMER.md](./SUMMER.md) (pre-work).
>
> **Technical companion:** the code repo's
> [`GPS_DENIED_PLAN.md`](https://github.com/csgomez25/gps-denied-drone-stack/blob/main/GPS_DENIED_PLAN.md)
> — every estimator attempt and what it measured, the **three** gates this document
> treats as one, and the next build with file and parameter names attached. **This
> document wins on priority and schedule; that one wins on mechanism.**
>
> Legend: ⭐ = do this one, it's the highest-leverage resource for the step. 📚 = deeper dive when you have time.

---

## How to use this roadmap
- **Keep a lab notebook** (a git repo or a Markdown journal). Write down every command that worked, every failure mode, every tuning value. Senior design is graded partly on this; future-you needs it too.
- **Do the core math by hand once.** Derive a 1D Kalman filter on paper before you trust a library. Plot an A* expansion. You only need to do it once to stop treating these as magic.
- **Build → break → understand → fix.** When something fails, resist copy-pasting a fix. Form a hypothesis first.

---

## ★ Your personal development track — VIO → autonomy engineer

Your goal isn't "do the project," it's **broaden from CV into an autonomy/localization engineer.** Your owned area is **State Estimation / VIO lead + autonomy-loop integration** (see [ROLES.md](./ROLES.md)). The key principle that makes this safe:

> **Learn VIO deeply on one clock; deliver VIO on another.**

- **Deliver (project clock):** integrate, calibrate, tune, and characterize **cuVSLAM** on the real sensor → feed PX4 EKF2 → measure drift. This *is* real localization engineering and it's what ships.
- **Learn the internals (your clock, offline, can't block the project):** datasets, not the aircraft.

**VIO learning ladder (do in order):**
1. ⭐ *Kalman & Bayesian Filters in Python* (Labbe) — work the EKF notebooks; derive a 1D filter by hand once.
2. ⭐ Run **OpenVINS** (or VINS-Fusion) on the **EuRoC MAV dataset** — watch a real estimator work; read its docs.
3. ⭐ **Implement a toy visual-inertial EKF / MSCKF-lite on EuRoC yourself**, offline. This is where deep understanding happens — and a buggy filter on a dataset crashes nothing.
4. Read the **MSCKF** and **IMU-preintegration (Forster et al.)** papers; 📚 Barfoot *State Estimation for Robotics* for rigor.
5. Learn **Kalibr** (camera–IMU calibration) — you'll use it for real on the RealSense.
6. **cuVSLAM** in sim → on the bench with the real RealSense → into EKF2 → drift number.

**Resume line this builds:** "CV engineer who built the VIO-based localization and integrated the full autonomy loop for a GPS-denied drone." That's a localization/autonomy story anchored in your existing CV strength.

> Don't solo the riskiest box blind: VIO is the schedule-dominating risk (BUILD.md §4). Pair on hard debugging, validate early/offline, and keep a fiducial/mocap localization fallback for flight tests so VIO tuning can't block the demo.

---

## Phase 0 — Foundations + Sim bring-up  (Weeks 1–2) — ✅ **done**
**Build:** everything in [SIM_WEEK1.md](./SIM_WEEK1.md) — ROS 2 Jazzy, PX4 SITL, fly an offboard waypoint, depth into ROS.

> **Status 2026-07-30: ✅ complete.** You write ROS 2 nodes from scratch
> (`planner_node`, `offboard_manager`, `fake_world`, `px4_tf_publisher`), the sim drone
> flies waypoints commanded from ROS, and **depth is now in ROS 2** — the last Phase-0
> item, closed 2026-07-30 (SIM_WEEK1 Day 4). Everything in Phase 0 is done.

**Learn first:**
- **ROS 2 core** — nodes, topics, pub/sub, services, parameters, `tf2`, launch files, `colcon`, workspaces.
  - ⭐ Articulated Robotics (Josh Newans) — YouTube channel + articulatedrobotics.xyz. The best practical ROS 2 teaching anywhere; he builds a real robot.
  - ⭐ Official ROS 2 Jazzy tutorials — https://docs.ros.org/en/jazzy/Tutorials.html (do the Beginner: CLI + Client Libraries tracks).
  - 📚 The Construct — https://www.theconstruct.ai (interactive, browser-based ROS courses).
- **Linux + Python/C++ comfort** — terminal, `apt`, virtualenvs, basic `rclpy`. If shaky on Python, fix that first; you'll write nodes in it.
- **PX4 mental model** — what SITL is, what offboard mode is, NED vs ENU frames.
  - ⭐ PX4 ROS 2 User Guide — https://docs.px4.io/main/en/ros2/user_guide

**Phase-0 done when:** you can write a ROS 2 node from scratch that subscribes to one topic and publishes another, and the sim drone flies a waypoint you command from ROS.

---

## Phase 1 — Full autonomy stack in simulation  (Months 1–2) — 🔄 **perception→flight ✅ · GNSS-off flight ✅ · estimator ❌**
**Build:** Day-7 loop fully fleshed out — cuVSLAM drift study, nvblox map, A* planner, offboard manager. Obstacle avoidance working in sim. (This is the §6 gate before buying hardware.)

> **Status 2026-08-01:** ✅ **A\* planner** (`astar.py`, validated headlessly and on real
> nuScenes HD-map rasters) and ✅ **offboard manager** are done and flew the closed loop
> autonomously in SITL on 2026-07-13. ✅ **Depth into ROS 2** and ✅ **the occupancy
> map** both closed 2026-07-30 — octomap maps three spawned obstacles to their true
> positions with open ground free. ✅ **The loop now plans on the perceived map**
> (2026-07-31, `PHASE1_GATE.md`): two autonomous legs, both planned entirely on a map
> the aircraft built from its own depth camera in flight, arriving to 3 cm and 12 cm
> with a 0.55 m minimum obstacle clearance. That is *perception→planning→flight*.
>
> 🔴 **The drift study ran and returned a negative result.** rtabmap `icp_odometry`
> registers 80–94% of frames after the `Odom/ResetCountdown` fix, but the *trajectory*
> is unusable: ATE 19.0 m over a 58.6 m path. The cause is geometric, not a tuning
> miss — a camera tilted 20° down at 2 m AGL sees mostly ground, and a ground plane
> leaves **x, y and yaw unobservable**. Four candidate fixes were measured and all
> made it worse or silently fake. Full record and the four-row negative-result table:
> `GPS_DENIED_PLAN.md` §3, `VIO.md`.
>
> **So Phase 1 is not ~70% along one axis — it is three separate gates.** Estimate
> (❌ still stuck), closed-loop flight on a non-GPS pose (✅ **flown 2026-08-01**), and
> survives-drift (⬜ not started, but now reachable). The middle one was an
> *integration* problem, not a research problem, and it is what earns the phrase
> "GPS-denied flight" — it had been blocked behind the first for no good reason, and
> took one session once attempted directly. See item 5 in **Start RIGHT NOW**.
>
> ✅ **The aircraft now flies with `EKF2_GPS_CTRL=0`**, on a pose PX4 did not compute.
> The pose is **simulator truth, not an estimator** — say that in the same breath, every
> time. It is not GPS-denied navigation; it is the harness a real estimator drops into.
>
> **Scope change (BUILD.md §0.6):** the sim map is now **octomap**, not nvblox, and the
> VIO front-end is **rtabmap `icp_odometry`**, not cuVSLAM. Isaac ROS remains the Jetson flight stack. This makes
> Phase 1 reachable on the laptop, but it means Phase-1 tuning numbers **will not
> transfer** to nvblox — budget re-tuning in Phase 2, and don't describe this phase as
> validating nvblox or cuVSLAM.
>
> **Track-4 note (quadrotor dynamics/control):** untouched so far. That's fine — the
> offboard position-setpoint interface let you skip it. It becomes real when you tune PID
> for actual all-up weight in Phase 3.

**Learn first — four tracks, in this order:**

**1. State estimation (the heart of GPS-denied).**
- ⭐ *Kalman and Bayesian Filters in Python* — Roger Labbe, free Jupyter book: https://github.com/rlabbe/Kalman-and-Bayesian-Filters-in-Python. Work the notebooks; this is the single best hands-on KF/EKF resource.
- ⭐ Cyrill Stachniss — YouTube ("Mobile Sensing and Robotics", SLAM, EKF lectures). University-quality, free.
- 📚 *State Estimation for Robotics* — Tim Barfoot, free PDF (asrl.utias.utoronto.ca). The rigorous VIO math (Lie groups, factor graphs) when you're ready.
- 📚 *Probabilistic Robotics* — Thrun/Burgard/Fox. The reference text for filters + SLAM.

**2. Visual-Inertial Odometry specifically.**
- ⭐ Read the **VINS-Mono** and **OpenVINS** papers — even if you use cuVSLAM, these explain *why* VIO works (feature tracking, IMU preintegration, initialization, scale).
- ⭐ First Principles of Computer Vision — Shree Nayar (Columbia), YouTube. Camera model, features, stereo — the vision half of VIO.
- NVIDIA Isaac ROS Visual SLAM docs — https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_visual_slam/

**3. Mapping + planning.**
- ⭐ *Planning Algorithms* — Steve LaValle, free book: http://lavalle.pl/planning/. Read the A*, Dijkstra, and sampling (RRT/PRM) sections.
- nvblox docs — https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/ (understand occupancy vs ESDF/TSDF).
- 📚 Cyrill Stachniss has occupancy-grid-mapping lectures too.

**4. Quadrotor dynamics, control & trajectories.**
- ⭐ **Coursera "Robotics: Aerial Robotics"** — Vijay Kumar, UPenn. *The* course for quadrotor dynamics, PD control, and **minimum-snap trajectories** (the Mellinger & Kumar method you'll use). Audit it free.
- ⭐ *Underactuated Robotics* — Russ Tedrake (MIT), free video + notes: https://underactuated.mit.edu. For understanding why quadrotors are controlled the way they are.
- 📚 *Modern Robotics* — Lynch & Park, free book + Coursera + YouTube. Rigid-body kinematics/dynamics foundation.
- 📚 Mellinger & Kumar 2011, "Minimum Snap Trajectory Generation and Control for Quadrotors" — the paper.

**Math backfill (only if these feel shaky — don't over-invest):**
- ⭐ 3Blue1Brown "Essence of Linear Algebra" (YouTube) — rotations, vectors, eigenstuff intuition.
- Probability basics (any intro) — you need Gaussians, covariance, Bayes' rule for the filters.

**Phase-1 done when:** sim drone autonomously avoids an inserted obstacle ✅, you can
*explain* how the estimator computes pose and how A\* found the path, and you have a
measured VIO drift number ✅ (19.0 m ATE — a real measurement, and a failing one).

> **Sharpened exit criterion — and it is now half met.** "A measured drift number" is
> satisfied by a bad number, which is how Phase 1 read as ~70% while the aircraft had
> still never flown without GPS. The replacement: **the aircraft flies with
> `EKF2_GPS_CTRL=0`, on a pose PX4 did not compute.** ✅ Done 2026-08-01 — on the square,
> on simulator truth, and it says so out loud. What remains for Phase 1 is the same
> flight on a pose from a real *estimator*, which is Gate A and still open.

---

## Phase 2 — Hardware bring-up  (Months 2–4)
**Build:** assemble the X500, PX4 config + calibration, manual flight, then Jetson Orin Nano + RealSense bring-up, cuVSLAM on the *real* sensor on the bench.

**Learn first:**
- **Drone hardware** — frames, motors (KV, sizing), ESCs, props, LiPo/Li-ion safety, power budget, soldering, ELRS RC link.
  - ⭐ Oscar Liang's blog — https://oscarliang.com. The best practical FPV/drone hardware reference (motor/ESC/prop selection, soldering, ELRS setup, thrust-to-weight).
- **PX4 hardware setup** — firmware flashing, sensor calibration, flight modes, failsafes, geofence, kill switch.
  - ⭐ PX4 Basic Assembly + Standard Configuration docs — https://docs.px4.io/main/en/config/
  - ⭐ Holybro X500 V2 build guide — https://docs.holybro.com/drone-development-kit/px4-development-kit-x500v2
- **Jetson Orin Nano setup** — JetPack flashing, NVIDIA Container Toolkit, running Isaac ROS containers on the Jetson.
  - ⭐ NVIDIA Jetson "Getting Started" + Isaac ROS Getting Started — https://nvidia-isaac-ros.github.io/getting_started/
  - ⭐ The Orin Nano Super + Isaac ROS practical guide (from the BUILD.md sources) — verify your release supports Orin Nano + Jazzy *before* relying on it.
- **RealSense on Jetson** — librealsense + the ROS 2 wrapper.

**Safety (non-negotiable before any powered prop test):**
- Props OFF for all bench/first-power tests. Build the kill switch + geofence in PX4 *before* the first flight. Net/enclosure for indoor.

**Phase-2 done when:** the X500 flies stably under manual control, and the Jetson runs cuVSLAM + nvblox on the real RealSense feed on the bench (off the aircraft).

---

## Phase 3 — Integration & flight test  (Months 4–6)
**Build:** cuVSLAM → PX4 EKF2 (vision-aided, GPS denied) → offboard autonomy. Closed-loop indoor obstacle avoidance, then outdoor / under-structure.

**Learn first:**
- **Vision-aided EKF2 / GPS-denied config** — feeding external odometry into PX4, EKF2 parameter tuning.
  - ⭐ PX4 "Using Vision or Motion Capture Systems for Position Estimation" + EKF2 docs — https://docs.px4.io/main/en/computer_vision/
- **Offboard control robustness** — setpoint rate, mode-switch safety, failsafe behavior on VIO loss.
- **Flight-test methodology** — incremental envelope expansion, logging (PX4 ulog + ROS bags), `PlotJuggler` / Flight Review for analysis.
  - ⭐ PX4 Flight Review + log analysis docs.
- **Controller tuning** — PID tuning for your actual all-up weight.
  - ⭐ PX4 Multicopter PID Tuning Guide.

**Phase-3 done when:** indoor autonomous obstacle avoidance demo passes your success metric (e.g. 3/5 trials, ≥0.5 m margin), then repeated outdoors / under structure.

---

## Phase 4 — Final demo + documentation  (Month 6+)
**Build:** final mission, evaluation report, presentation. **Stretch:** outdoor robustness, then the custom STM32H7 flight-controller PCB (only if Phase 3 finished early — see BUILD.md scope ladder).

**Learn (only if pursuing the PCB stretch):**
- KiCad, STM32 + the ardupilot/PX4 hardware bring-up process, hardware bring-up debugging. This is a whole second project — gate it hard.

---

## Resource library (grouped, for reference)

**ROS 2**
- Articulated Robotics (YouTube + site) ⭐ · ROS 2 Jazzy docs ⭐ · The Construct · *A Gentle Introduction to ROS* (O'Kane, free, ROS1 concepts)

**PX4 / drones / hardware**
- PX4 docs (docs.px4.io) ⭐ · Oscar Liang (oscarliang.com) ⭐ · Holybro docs · ArduPilot docs (good background)

**State estimation / VIO**
- Kalman & Bayesian Filters in Python (Labbe) ⭐ · Cyrill Stachniss (YouTube) ⭐ · State Estimation for Robotics (Barfoot, free) · Probabilistic Robotics (Thrun) · VINS-Mono / OpenVINS papers · First Principles of Computer Vision (Nayar, YouTube)

**Planning / control / dynamics**
- UPenn Aerial Robotics (Coursera, Kumar) ⭐ · Underactuated Robotics (Tedrake, free) ⭐ · Planning Algorithms (LaValle, free) ⭐ · Modern Robotics (Lynch & Park, free) · Mellinger & Kumar min-snap paper

**NVIDIA / Isaac ROS / sim**
- Isaac ROS docs (nvidia-isaac-ros.github.io) ⭐ · NVIDIA DLI courses · Gazebo docs · PX4 SITL docs

**Math foundations (only if needed)**
- 3Blue1Brown Essence of Linear Algebra ⭐ · any intro probability (Gaussians, Bayes, covariance)

---

## Start RIGHT NOW (re-prioritised 2026-07-31 — hardware is budget-locked)

> **What changed.** Parts are ~90% decided (Orin Nano Super + D435i) but the budget
> does not unlock until the course starts. Nothing can be ordered. So the binding
> constraint is no longer money — it is **fall time**, which goes to assembly,
> bring-up, calibration, integration and mentoring two new teammates. The remaining
> summer exists to climb the learning curves *now*, because the fall has no room for
> them. See [SUMMER.md](./SUMMER.md).
>
> Consequence: **the Phase-1 sim gate no longer gates anything.** It existed to decide
> "is it safe to buy hardware?" and budget already decided that. Close it for the
> milestone; don't let it displace items 1–3.

1. ⭐ **VIO ladder, steps 1–3** (Labbé's filters → OpenVINS on EuRoC → your own toy
   VI-EKF). This *is* the career deliverable and the sim work will never produce it.
   Entirely offline, so parts cannot block it. `~/DPVO` already carries
   `evaluate_euroc.py` and EuRoC logs. **Highest-value hours available this summer.**
2. ⭐ **De-risk Isaac ROS on the laptop.** Every decision so far deferred it — octomap
   instead of nvblox, rtabmap instead of cuVSLAM — and BUILD §0.6 flags it as
   "unverified for longer". The dev box can run the container (RTX A3000 6 GB, driver
   595.71, 308 GB free); Docker and the NVIDIA Container Toolkit are not installed.
   Install, pull, run the cuVSLAM quickstart on a dataset. This also discharges the
   §0.5 ⚠️ support-matrix item, which must **not** wait for the budget: if Isaac ROS ×
   Jazzy × Orin Nano needs Humble, that invalidates a lot of Jazzy-specific work and
   July is a much cheaper time to find out than October.
3. **Practice Kalibr on a dataset.** Camera–IMU calibration is precisely what sim
   cannot teach — the sim extrinsic is exact and free, the hardware one is vibration,
   thermal drift and a fiddly toolchain. EuRoC ships calibration data.
4. ✅ **Close the Phase-1 gate** — done 2026-07-31. The swap was the one-line remap it
   was designed to be; everything else in `PHASE1_GATE.md` exists because a *perceived*
   map is not a synthetic one (unbounded extent, unknown space, a goal that may not be
   in the map yet).
5. ✅ **Fly on a pose PX4 did not compute — DONE 2026-08-01.** Armed, flew a 5 m square
   to 3.16 m and landed with **`EKF2_GPS_CTRL=0`**, on a pose fed to
   `/fmu/in/vehicle_visual_odometry` from Gazebo ground truth. EKF2 tracked truth to
   mean 0.073 m / max 0.199 m. Phase-3 work pulled into sim, and it cost one session
   instead of flight tests — exactly the argument for doing it early.

   **It paid off the way decoupling is supposed to.** The flight failed on the first
   attempt, and the cause was a bug that had been corrupting *every integer PX4
   parameter this project sets* since the script was written: MAVLink carries params in
   a float32 field and PX4 reads integers out of it bytewise, so `EKF2_EV_CTRL=15` was
   stored as 1097859072. The read-back verified nothing because it round-trips the same
   corruption. It had never mattered while GNSS was on, and became fatal the instant it
   was switched off. Found against a *perfect* pose in an afternoon; against a drifting
   estimator in October it would have been a week and blamed on the estimator.

   **The budget is measured.** Sweeping injected drift until the gate fails: 0.02 m/s
   passes, 0.05 m/s does not. More useful than the number itself is *why* — **EKF2 does
   not attenuate the drift at all**, because with GNSS off external vision is the only
   aiding source and there is nothing to test it against. Observed error equals injected
   error, one for one.

   So **the spec for Gate A is accumulated position error (~1.5 m), not a drift rate** —
   a longer mission fails at a proportionally lower rate. Measured ICP is ATE 19.0 m over
   58.6 m, which misses that bar by more than an order of magnitude. Detail:
   `CLOSED_LOOP.md` §8. Yaw sweep still to finish, and it is the one that decides whether
   the gap is tuning or structural.
6. ✅🟡 **Day 6 — timeboxed, and the timebox is now spent.** The `ratio=0` cause is
   found and fixed (one failed registration cleared the velocity model, and
   `Odom/ResetCountdown` defaults to *never reset* — so a single bad frame wedged
   odometry permanently; `--Odom/ResetCountdown 1` took registration from 4.3% to
   80.3%). The Gazebo `PosePublisher` ground truth is **not** done and has moved to
   item 5, where it pays twice. **Per this item's own instruction, ICP parameter work
   stops here** — the remaining failure is an unobservable DOF, and a DOF that is
   geometrically unobservable cannot be recovered by tuning. rtabmap does not ship;
   cuVSLAM does.
7. ⬜ **Research track**: the 2-DOF along-track + heading matcher; then RPE and a paired
   significance test.
8. ⬜ Audit-enroll in the **UPenn Aerial Robotics** Coursera course.

*(Deferred to the fall with the budget: verifying Orin Nano vs. the old Nano, and
ordering the D435i and the X500 kit.)*

*(Done: all code under git and pushed; SIM_WEEK1 Days 1–5; the Phase-1 gate flown on the
perceived map; the octomap and rtabmap scope decisions recorded in BUILD.md §0.6; the
Day-4 TF conflict, the Gazebo RAM leak, the octomap ground filter, the planner's
unknown-space policy, the `icp_odometry` reset latch, and the false-groundtruth
measurement error — all found, fixed and measured.)*

> Buying is deferred to the fall by budget, not by the gate. When funds unlock,
> verify the §0.5 ⚠️ items first, then order the long-lead D435i.
