# Learning + Build Roadmap

> Philosophy: **learn just-in-time.** Don't pre-study six months of theory. Learn each concept right before the build step that forces you to use it — you'll retain 10× more. Each phase below pairs *what you build* with *what to learn first* and *where to learn it*.
>
> Companion docs: [README.md](./README.md) (plan going forward) · [BUILD.md](./BUILD.md) (decisions/BOM) · [ROLES.md](./ROLES.md) (team) · [archive/SIM_WEEK1.md](./archive/SIM_WEEK1.md) (week-1 commands, historical) · [archive/SUMMER.md](./archive/SUMMER.md) (original pre-work plan, historical).
>
> **Technical companion:** the code repo's
> [`GPS_DENIED_PLAN.md`](https://github.com/csgomez25/UAV-UGV-docking-CodeStack/blob/main/GPS_DENIED_PLAN.md)
> — every estimator attempt and what it measured, the **three** gates this document
> treats as one, and the next build with file and parameter names attached. **This
> document wins on priority and schedule; that one wins on mechanism.** Its companions
> `HANDOFF.md` (pick-up state + environment invariants) and `ISSUES.md` (every bug,
> grouped by the 8 recurring shapes) are the other two worth knowing exist.
>
> Legend: ⭐ = do this one, it's the highest-leverage resource for the step. 📚 = deeper dive when you have time.

---

## ★ The senior project is now cooperative docking (2026-09-10)

The phases further down were written for the GPS-denied quadrotor. They still describe the
**GPS-denied research layer**, but the schedule that matters is now this one. The code
repo's `DOCKING.md` is its technical companion, the same way `GPS_DENIED_PLAN.md` is for
the GPS-denied layer. The plan itself: [`docking/`](./docking).

| When | Build | Gate | Learn first |
|---|---|---|---|
| Week 1 | UGV-sized AprilTag pad on PX4's moving-platform world; downward camera; tag → relative pose | D0 — ✅ **PASS 2026-09-14 at 1280×960, 0–2.5 m** (FAIL 09-10 at 640×480, 1.25 m) | Camera model + calibration; AprilTag detection; PnP (`solvePnP`) |
| Week 2 | Moving pad drives its path (M0 ✅ 09-14); hover over the pad by camera (H0 ✅ 09-15); repeatable stationary-pad landing | D1 — ✅ **PASS 2026-09-15, 20/20** (19/20 first), median 1.2 cm | Descent profiles and touchdown criteria; PX4 offboard velocity setpoints |
| Weeks 3–4 | Pad moving at one speed; UAV-only chase controller; baseline dataset | D2 — ⬜ **next** | Control in a moving frame; cascaded position/velocity control; a Kalman filter on the relative state |
| Weeks 5–6 | UGV velocity broadcast (with latency/noise) fused into the relative estimate + feedforward; 20-trial matrix | D3 | KF with latency compensation; time sync between vehicles |
| December | **Presentation of the sim result** — distributions, failure cases named | — | — |
| Jan–Feb | Minimal hardware; airframe flying repeatably before any docking attempt | D4 | PX4 hardware setup, safety gate (the Phase 2 material below applies) |
| Mar–Apr | Physical docking reproducing the sim numbers | D5 | Ground-truth methods; flight-test methodology |
| Jun 4–6, 2027 | C-UASC flight event | — | — |

**Competition dates** (verify on the official pages): Blue Skies NOI **Oct 12, 2026** ·
C-UASC registration **Nov 1, 2026 – Feb 1, 2027** · Blue Skies proposal + video
**Feb 22, 2027** · C-UASC design submission **May 1, 2027**.

**The VIO ladder below keeps its place.** It no longer gates the senior
project; it gates the Blue Skies version.

---

## How to use this roadmap
- **Keep a lab notebook** (a git repo or a Markdown journal). Write down every command that worked, every failure mode, every tuning value. Senior design is graded partly on this; future-you needs it too.
- **Do the core math by hand once.** Derive a 1D Kalman filter on paper before you trust a library. Plot an A* expansion. You only need to do it once to stop treating these as magic.
- **Build → break → understand → fix.** When something fails, resist copy-pasting a fix. Form a hypothesis first.

---

## ★ The VIO / state-estimation development track

For whoever owns **State Estimation / VIO lead + autonomy-loop integration** (see
[ROLES.md](./ROLES.md)), the goal is broader than shipping the project: build real
depth in VIO/localization, not just wire up a library. The key principle that makes
this safe:

> **Learn VIO deeply on one clock; deliver VIO on another.**

- **Deliver (project clock):** integrate, calibrate, tune, and characterize **cuVSLAM** on the real sensor → feed PX4 EKF2 → measure drift. This *is* real localization engineering and it's what ships.
- **Learn the internals (offline, can't block the project):** datasets, not the aircraft.

**VIO learning ladder (do in order):**
1. ⭐ *Kalman & Bayesian Filters in Python* (Labbe) — work the EKF notebooks; derive a 1D filter by hand once.
2. ⭐ Run **OpenVINS** (or VINS-Fusion) on the **EuRoC MAV dataset** — watch a real estimator work; read its docs.
3. ⭐ **Implement a toy visual-inertial EKF / MSCKF-lite on EuRoC yourself**, offline. This is where deep understanding happens — and a buggy filter on a dataset crashes nothing.

> ⚠️ **"Do in order" was not advice, it turns out (2026-08-08).** OpenVINS was built and
> pointed straight at the aircraft, skipping step 2. It stalls in initialisation
> (`not enough feats to compute disp: 0,47 < 15`), and because it has **never been run on
> a dataset where it is known to work**, there is no reference to diff against — every
> hypothesis has to be ruled out from first principles instead of by comparison. Step 2
> is now simultaneously the learning-ladder priority *and* the cheapest diagnostic available
> for the live blocker. Do it.
>
> ✅ **Step 2 done 2026-09-01 — and it paid immediately.** OpenVINS on **EuRoC MH_01**:
> **ATE 0.221 m over 73 m of path** (0.114 m once the filter settles), stock `euroc_mav`
> config, same binary that stalls on the aircraft. It initialises correctly and recovers a
> gyro bias matching EuRoC's documented value, so **the build and the estimator core are
> sound and the aircraft's fault is on the Gazebo side** — which is exactly the halving of
> the search space that three weeks of first-principles debugging had not produced. Full
> record in the code repo: `gps_denied_autonomy/results/gate_a/openvins_euroc_ref/`.
>
> **The lesson, stated plainly because it will recur on Kalibr and on cuVSLAM.** The
> ladder's order is not pedagogical politeness — a component run on data where it is
> *known* to work is a diagnostic instrument, and skipping that step does not save the
> time, it spends it later at a worse exchange rate. One session bought what three weeks
> of theorising had not. Steps 3 and 5 are unstarted; do them before they are needed, not
> after.
4. Read the **MSCKF** and **IMU-preintegration (Forster et al.)** papers; 📚 Barfoot *State Estimation for Robotics* for rigor.
5. Learn **Kalibr** (camera–IMU calibration) — you'll use it for real on the RealSense.
6. **cuVSLAM** in sim → on the bench with the real RealSense → into EKF2 → drift number.

> Don't leave the riskiest box to one person working alone: VIO is the schedule-dominating risk (BUILD.md §4). Pair on hard debugging, validate early/offline, and keep a fiducial/mocap localization fallback for flight tests so VIO tuning can't block the demo.

---

## Phase 0 — Foundations + Sim bring-up  (Weeks 1–2) — ✅ **done**
**Build:** everything in [archive/SIM_WEEK1.md](./archive/SIM_WEEK1.md) — ROS 2 Jazzy, PX4 SITL, fly an offboard waypoint, depth into ROS.

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
> 🔴 **Update 2026-08-13 — two more candidates, and Gate A has still not moved.**
> `rgbd_odometry` (the "then use the RGB" answer, which *does* make yaw observable) was
> flown, scored, and **closed on evidence**: with `Odom/ResetCountdown=0` it tracks for
> 14 s at ATE 0.033 m and then latches dead; with `=1` it survives the mission and drifts
> **11–35% of path**, the same band as the ICP it was meant to beat. **OpenVINS** is
> built from source and stuck in initialisation, so it has produced no pose at all.
> **The binding budget — 0.5 °/s yaw drift — has never been measured for any candidate**,
> because none has survived long enough on position for yaw to be the deciding term.
>
> ⚠️ **And the instrument was wrong before the estimators were.** Six bugs in
> `eval_vio_drift.py` were fixed on 2026-08-02/03, all the same shape — *it reported a
> good number for a bad run.* Gate A swung 30 → 25 → 35 → 30% in one day with the
> aircraft untouched. **No Gate A number from before 2026-08-03 is comparable with one
> after.** The fix that matters is `check_vio_score.py`: ten synthetic flights with
> answers known by construction, ~0.1 s, no ROS and no aircraft. Every one of the six
> survived for weeks precisely because a real flight has no known answer, so the only
> check available was whether the number looked plausible — and all six looked plausible.
> `ISSUES.md` §E.
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
>
> **Sharpened once more, 2026-08-13 — because "a drift number" failed a second time.**
> Three candidates have now produced drift numbers and none is a pose you can fly on. A
> number is not the criterion; **meeting the measured budget is**: `coverage ≥ 0.8`,
> accumulated position error **≤ ~1.5 m over the mission**, and yaw drift **≤ 0.5 °/s** —
> that last one being the term that actually binds and the one nothing currently
> measures. Scored by `eval_vio_drift.py` against simulator truth, with
> `check_vio_score.py` green first. **Read `coverage` before ATE**: on a mission that
> returns to its origin, an estimate that barely moves scores a *good* ATE for doing
> nothing, and that is how three invalid runs got quoted.

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

## GPS-denied layer — current priorities

The detailed, dated blow-by-blow of this list (budget-lock reasoning, every VIO
candidate's numbers, what's already closed out) has been superseded several times over
and is preserved in [archive/GPS_DENIED_LOG.md](./archive/GPS_DENIED_LOG.md). The
standing priorities, current as of the docking pivot:

1. ⭐ Unblock OpenVINS initialisation — the live Gate A blocker.
2. ⭐ De-risk Isaac ROS in a container on the dev box (Docker + NVIDIA Container Toolkit
   still not installed; the Isaac ROS × Jazzy × Orin Nano support-matrix check gates
   hardware bring-up and should not wait for it).
3. Practice Kalibr on a public dataset (EuRoC).
4. Measure yaw drift rate — the error-budget term that actually binds and that nothing
   currently measures.
5. Gate C (drift survivability) — reachable today via `fake_vio`, no estimator needed.
6. Research track: the 2-DOF along-track + heading matcher, then RPE + significance test.

The VIO ladder (Labbé's filters → OpenVINS/EuRoC → a toy VI-EKF) above stays the
priority skill-building path for whoever owns this track.

> Buying is deferred to the fall by budget, not by the gate. When funds unlock,
> verify the §0.5 ⚠️ items first, then order the long-lead D435i.
