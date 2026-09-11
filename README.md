# Cooperative UAV–UGV Docking — Project Hub

**Senior project as of 2026-09-10:** an autonomous drone that lands on a ground robot
**while the robot is still moving**, by having the two vehicles cooperate — the UGV
broadcasts its velocity and the UAV fuses that with visual relative pose — rather than
chasing a passive target (RSCL@CPP Project 2). This folder is the planning + lab-notebook
home.

The repo keeps its name from the GPS-denied phase. That work is not abandoned: it is the
**research/mission layer** (a drone that inspects under structures without GPS and docks
on a moving ground robot to recharge), no longer the critical path. See
[Direction change](#direction-change--2026-09-10).

> **Docking status (2026-09-10): gate D0 flown — FAIL, usable to 1.25 m against a 1.5 m gate.**
> The pad's pose from the downward camera is accurate from touchdown to 1.25 m (≥ 98% of
> frames, ≤ 1.1 cm and ≤ 0.7° at p95). Above that, the tags are too small for a 640×480,
> 100° camera to decode. **Next: choose lens, resolution or tag size** (code repo
> `DOCKING.md` §6.1), re-fly D0, then D1. Plan docs: [`docking/`](./docking).

> GPS-denied layer status (2026-08-13): **the aircraft flies with GNSS fusion off; nothing estimates
> its pose yet.** The perception→planning→flight loop closes in SITL on a map the
> aircraft builds itself (Days 4–5, 7), *and* the X500 has flown a full autonomous
> square with **`EKF2_GPS_CTRL=0`** — no GNSS fusion at any point — on a pose supplied
> from outside the autopilot (2026-08-01). Say that with **"the pose source is
> simulator truth, not an estimator"** in the same breath. The remaining work is the
> estimator: **three candidates tried, two closed on evidence, the third produces no
> pose yet.** No flying hardware — by design.
> See [Where things stand](#where-things-stand).
>
> **Phase 1 is not one gate, it is three** — Estimate ❌ / Closed-loop ✅ / Survives-drift ⬜.
> Treating them as one is what let "~85%" describe a stack that had never flown without
> GPS. Detail: [ROADMAP.md](./ROADMAP.md) Phase 1, and the code repo's `GPS_DENIED_PLAN.md`.

---

## Direction change — 2026-09-10

**What:** the senior project is now **Cooperative UAV–UGV Docking: Reducing Relative
Landing Error Under Platform Motion and Disturbance**. Hypothesis: a controller that fuses
the UGV's broadcast velocity with visual relative pose lands with lower touchdown error,
and succeeds at higher platform speeds, than a UAV-only chase controller.

**Why this project:** the existing stack is already the UAV half (PX4 + ROS 2 offboard
control, TF, sim harnesses, the trial-sweep methodology), and the AVL perception work is
the ground-robot half. The new research is the piece in between: relative-state fusion
and a controller that works in the moving pad's frame.

**How it is sequenced:** build the plain docking version first, with GPS on for anything
scored; keep GPS-denied inspection as the mission framing and the research layer on top.
Docking takes relative pose from the fiducial, so it never depends on Gate A closing.

| | Docs |
|---|---|
| The plan | [`docking/ONE_PAGER.md`](./docking/ONE_PAGER.md) · [`docking/FIRST_MEETING_PACKET.md`](./docking/FIRST_MEETING_PACKET.md) |
| Parts, ground truth, theory, sim gaps | [`docking/REQUIREMENTS_AND_THEORY.md`](./docking/REQUIREMENTS_AND_THEORY.md) |
| Mechanism (code repo) | `DOCKING.md` — what PX4 provides, the gaps, the node plan, gates D0–D5 |

**Competitions:** primary **C-UASC 2027** (registration Nov 1, 2026 – Feb 1, 2027; design
May 1; flight Jun 4–6). Backup **NASA Gateways to Blue Skies 2027** (NOI Oct 12, 2026;
proposal + video Feb 22, 2027).

**What stays true from the GPS-denied phase:** every claim rule below. Nothing estimates
pose yet, so "GPS-denied navigation" is still not claimable, and the Blue Skies framing
must say the GPS-denied half is in progress.

---

## The plan, in one paragraph

**Docking (2026-09-10 →):** six weeks of simulation — tag pose, stationary-pad landing,
a chase baseline on a moving pad, then the cooperative controller, ending in a 20-trial
matrix (2 strategies × ≥2 platform speeds) reported as distributions. December
presentation on the sim result. Jan–Feb build minimal hardware, gated on the airframe
flying repeatably. Mar–Apr reproduce the sim numbers physically, for C-UASC in June.

**GPS-denied phase (summer 2026, original plan):** Use the summer (no deadline) to **de-risk and learn**: stand the full autonomy stack up in **simulation** first, learn **VIO** deeply offline, and secure the **long-lead parts** — so the fall course starts past the painful early phase instead of scrambling. Build on the **PX4-documented reference platform** (Holybro X500 V2 + Jetson Orin Nano + RealSense + ROS 2 Jazzy + Isaac ROS cuVSLAM/nvblox) rather than a risky custom airframe, keeping the custom frame + custom flight-controller PCB as gated stretch goals. Prove obstacle avoidance in sim → bring up hardware → close the loop indoors → then outdoors. Ship a defensible deliverable via a **scope ladder** so the project can't fully fail.

---

## Your path (career angle)

You're broadening from **CV → autonomy/localization engineer.** You lead **State Estimation / VIO** + own **autonomy-loop integration**, co-own planning, and hand off pure detection CV. Principle: **learn VIO deeply on your own clock (offline, EuRoC); deliver VIO on the project clock (cuVSLAM, calibrated + tuned).** See [ROADMAP.md](./ROADMAP.md) + [ROLES.md](./ROLES.md).

---

## Team

- **Summer (2):** you = all software/autonomy (in sim) · friend = structural + electrical (assembly → manual flight → mounts + vibration isolation).
- **Fall (3–4):** add a **planning** teammate (you mentor) + a **control** teammate; friend continues hardware. → [ROLES.md](./ROLES.md)

---

## Document map

| Doc | What it's for |
|---|---|
| [docking/](./docking) | ⭐ **The current project** — one-pager, first-meeting packet, requirements/parts/theory |
| [BUILD.md](./BUILD.md) | Locked technical decisions, frame/sensor/compute choices, BOM, weight math, risks |
| [SIM_WEEK1.md](./SIM_WEEK1.md) | Day-by-day commands to stand up sim (ROS 2 Jazzy + PX4 SITL + cuVSLAM/nvblox + the Day-7 autonomy loop) |
| [ROADMAP.md](./ROADMAP.md) | Learn-just-in-time roadmap + curated resources + your VIO development track |
| [ROLES.md](./ROLES.md) | Team split, the data flow, and the interface contracts to nail |
| [SUMMER.md](./SUMMER.md) | What you + your friend do *right now*, and what to buy now vs. fall |

**And in the code repo**, which moves faster than this one and wins on mechanism:

| Doc | What it's for |
|---|---|
| `DOCKING.md` | 🛬 **The docking mechanism** — what PX4 already provides, the four sim gaps, the node and topic plan, gates D0–D5 |
| `HANDOFF.md` | 🔁 **Picking the project up cold.** Environment invariants to check *every session*, what works vs. what is structurally broken, the measured budget, and where to pick up. Read §1–§2 first |
| `TOUR.md` | 🧭 **New here.** Every node and script in a paragraph or less |
| `GPS_DENIED_PLAN.md` | The three gates, every estimator attempt and what it measured. **This roadmap wins on priority; that one wins on mechanism** |
| `ISSUES.md` | 🐛 Every bug long-form, grouped by the **8 shapes** that keep recurring, plus the open list ranked by cost |
| `gps_denied_autonomy/CLOSED_LOOP.md` | **Gate B** — the flight, the three silent bugs, and §8 the error budget |
| `gps_denied_autonomy/results/gate_a/README.md` | Gate A run tables and how to read them |

### Where the actual work lives (outside this repo)

Both working trees now live in **one repo**, `~/gps-denied-drone-stack/`:

| Tree | What it is | Last worked |
|---|---|---|
| `~/gps-denied-drone-stack/gps_denied_autonomy/` | ROS 2 autonomy nodes — `offboard_manager`, `planner_node`, `astar`, `fake_world`, `px4_tf_publisher`, `fake_vio`, `optical_frame_relay`. Runbooks: `README.md`, `SITL_FLIGHT.md`, `DEPTH_SIM.md`, `MAPPING.md`, `PLANNING.md`, `VIO.md`, `PHASE1_GATE.md`, **`CLOSED_LOOP.md`** | 2026-08-13 |
| `~/gps-denied-drone-stack/bev_gps_denied/` | The **research** track — semantic-BEV map-matching localization on nuScenes. Runbook: `README.md`, thesis: `GOAL.md` | 2026-07-13 |
| `~/ws_px4/src/open_vins/` | **A patched clone of `rpng/open_vins`, not a submodule.** 3 Jazzy header migrations (`.h` → `.hpp`) plus a `libceres-dev` dependency; a fresh clone silently removes the patch | 2026-08-08 |
| `~/PX4-Autopilot/`, `~/Micro-XRCE-DDS-Agent/` | Upstream deps, built from source | — |
| `~/DPVO/` | Monocular VIO front-end used offline for the research track's real-drift study | 2026-07-07 |

The old paths still work — `~/ws_px4/src/gps_denied_autonomy` and `~/bev_gps_denied`
are now **symlinks** into that repo. That keeps `colcon build` working from `~/ws_px4`
(verified with a clean rebuild) while the code lives under version control in one place.

> ✅ **Both working trees went under git on 2026-07-30**, then merged into the single
> `gps-denied-drone-stack` repo (`5aa13a5` and `08eda77` are preserved in its history
> as subtrees). Six weeks of results — the DPVO drift study, the causal experiments,
> the first autonomous SITL flight, the Day-4 depth bridge — are now tracked rather
> than loose on this disk.
>
> **Two repos total:**
>
> | Repo | Holds |
> |---|---|
> | [`csgomez25/GPS-Denied-Drone`](https://github.com/csgomez25/GPS-Denied-Drone) | this one — planning, roadmap, lab notebook |
> | [`csgomez25/GPS_Denied`](https://github.com/csgomez25/GPS_Denied) **(private)** | all the code — `gps_denied_autonomy/` + `bev_gps_denied/` |
>
> The code repo was briefly named `GPS_Denied_drone`; that URL still redirects.
> Unrelated: `csgomez25/IGVC_BEV` is the ground-vehicle competition project, not this.

---

## Where things stand

### The three gates — read this before any percentage

Phase 1 was tracked as one number until 2026-08-01, and that is how "~85%" came to
describe a stack that had never flown without GPS. It is three gates, and they fail for
different reasons:

| Gate | Question | State | % |
|---|---|---|---|
| **A — Estimate** | Is the pose estimate accurate enough to fly on? | ❌ **Open.** Two candidates closed on evidence, a third built and blocked | ~30% |
| **B — Closed loop** | Will it *fly* on an external pose, GNSS fusion off? | ✅ **Flown 2026-08-01, budget measured 08-02** | 100% |
| **C — Survives drift** | Does the map stay usable, does margin hold, under drift? | ⬜ Not started — but **reachable without an estimator** | ~5% |
| Supporting stack (SITL) | flight, depth, map, planner, gates, harness | ✅ | ~95% |
| **Overall → GPS-denied navigation in sim** | | | **~46%** |

> **That ~46% is front-loaded onto the easy half.** Everything finished so far was an
> *integration* problem, where effort reliably converts into progress. Gate A is a
> *research* problem and does not have that property. **Do not read a schedule off it.**

### Phase-1 sim gate (SIM_WEEK1)

| Day | Milestone | Status |
|---|---|---|
| 1 | ROS 2 Jazzy | ✅ `/opt/ros/jazzy` |
| 2 | PX4 SITL + Gazebo Harmonic, manual flight | ✅ gz-harmonic 8.12, PX4 builds + flies |
| 3 | Offboard waypoint commanded from ROS 2 | ✅ 2026-07-13 |
| 4 | Depth camera into ROS 2 | ✅ 2026-07-30 — 4 topics at rate, in flight, real TF |
| 5 | Occupancy map from depth | ✅ 2026-07-30 — **octomap**, not nvblox ([BUILD.md §0.6](./BUILD.md)). 3/3 obstacles mapped, 0.0% spurious |
| 6 | VIO + a drift number | ❌ **Gate A, open.** Registration fixed; *three* candidates measured, none viable. See below |
| 7 | Close the loop | ✅ **2026-07-31** — two autonomous legs planned on the perceived map; arrived 0.03 m / 0.12 m from goal, 35.2 m flown for a 16.3 m straight line (the detour) |
| — | **Fly with GNSS fusion off** | ✅ **2026-08-01** — square to 3.16 m, `EKF2_GPS_CTRL=0`, est vs truth max 0.199 m. Not a SIM_WEEK1 day; it is Gate B, pulled forward from Phase 3 |

**The Day-7 loop already flies.** On 2026-07-13 the X500 armed, took off, threaded a
doorway on an A\* path, and landed — 1596 telemetry samples, 32 s, ended at the goal
(NED −0.05, +7.87, −2.98). Evidence + the three integration bugs fixed getting there:
`~/ws_px4/src/gps_denied_autonomy/SITL_FLIGHT.md`.

**Depth is now in ROS 2 (Day 4, 2026-07-30).** Depth image, camera_info, point cloud
and IMU all deliver at rate, captured *during* an autonomous square with a real moving
`map -> base_link -> camera_link` TF. Runbook and numbers:
`~/ws_px4/src/gps_denied_autonomy/DEPTH_SIM.md`.

**The map is now built from the sensor (Day 5, 2026-07-30).** `octomap_server` consumes
`/depth_camera/points` and publishes `/projected_map`: three deliberately asymmetric
obstacles all map to their true positions and **heights** (21–37× denser than the map
as a whole), and a control patch of open ground comes back **0.0% occupied**. Runbook,
the two silent-failure traps, and the defect that was found and fixed:
`gps_denied_autonomy/MAPPING.md`. Visuals: `results/octomap_day5.png`,
`results/octomap_3view.png`, and a one-page `results/progress_board.png`.

**The camera is now tilted 20° down (2026-07-30).** Mounted level, half the depth frame
was sky and grazing-incidence ground returns defeated octomap's plane filter — 18.4% of
the open ground the drone flew over came back *occupied*. At 20° that is zero, frame
utilisation goes 52% → 78%, and 26% more area gets mapped per flight. Tilt beats flying
lower: lowering the aircraft does not move the horizon.

**PX4's `forest` world now maps too** — 25 oaks, 5 pines, 60 469 occupied voxels up to
6.2 m. It costs Gazebo 653 MB vs 490 MB for the empty world, and it is the world Day 6
needs: ICP odometry on a flat plane is degenerate, whereas trees constrain every
direction.

**The loop now closes on the perceived map (Day 7, 2026-07-31).** `planner_node` was
pointed at `/projected_map` and the X500 flew two autonomous legs on it, arriving 0.03 m
and 0.12 m from goal. Leg 2 flew **35.2 m for a 16.3 m straight line** — the direct route
crosses the mapped wall, so the planner routed around it and the aircraft stayed 3.27 m
clear. The obstacles also mapped ~20 points denser than the static Day-5 run, because
`offboard_manager` yaws along the path and swept the camera across many headings.
Runbook, the four things a perceived map breaks, and the figure:
`gps_denied_autonomy/PHASE1_GATE.md`, `results/phase1_flight.png`.

> The swap itself was one remap — `/occupancy_grid` → `/projected_map`, exactly as
> predicted, since the planner already consumed that message type. What was **not**
> free was everything a perceived map does that a synthetic one never did: the goal is
> normally outside the observed grid, nothing publishes `/current_pose` in the real
> stack, inflation swallows the aircraft's own cell, and octomap publishes ~25× faster
> than a plan takes. Each failed silently. `PHASE1_GATE.md` §2.

**Gate B is closed, and it earned a real phrase (2026-08-01).** The X500 armed, flew a
5 m square to 3.16 m and landed with **`EKF2_GPS_CTRL=0`** — no GNSS fusion at any point
— on a pose fed to `/fmu/in/vehicle_visual_odometry` from Gazebo ground truth. EKF2
tracked truth to **mean 0.073 m / max 0.199 m**, 1160 of 1160 armed samples valid. This
was Phase-3 work pulled into sim, and it cost one session instead of flight tests.

> It also paid off exactly the way decoupling is supposed to. The first attempt failed,
> on a bug that had been corrupting **every integer PX4 parameter this project sets**
> since the script was written: MAVLink carries params in a `float32` and PX4 reads
> integers out of it bytewise, so `EKF2_EV_CTRL=15` was stored as `1097859072`. The
> read-back verified nothing because it round-trips the same corruption. It never
> mattered while GNSS was on. Found against a *perfect* pose in an afternoon; against a
> drifting estimator in October it would have been a week, and blamed on the estimator.

**The error budget is measured (2026-08-02, 20 runs, four knobs).** This is the
specification any estimator has to meet:

| knob | budget | shape |
|---|---|---|
| **yaw drift** | **0.5 °/s** | **hard wall** — 1.0 °/s never completes the mission at all |
| **position drift** | **0.02 m/s**, really **~1.5 m accumulated** | crossing; scales with mission length |
| latency | 200 ms | flat to 100 ms, then refuses to arm at 400 |
| position noise | **none found — ≥ 0.6 m** | never broke; a lower bound, not a measurement |

**EKF2 does not attenuate drift — observed error equals injected error, one for one.**
With GNSS off, external vision is the only aiding source, so in flight there is nothing
to check the pose against. But it **filters zero-mean noise hard** (12× the sigma buys
1.5× the error). *An estimator for Gate A may be noisy; it may not be biased.* The budget
is therefore an accumulated **displacement**, not a rate — a longer mission fails at a
proportionally lower rate, so "0.035 m/s" means nothing without "over a ~40 s flight."

**Gate A is the whole remaining project, and three candidates are now measured.**

- **`icp_odometry` — closed, geometric.** A depth camera over mostly-ground leaves yaw
  unobservable; ATE/path **32.4%**. An unobservable DOF has no rate to tune down, so this
  is a structural finding, not a tuning miss.
- **`rgbd_odometry` — closed 2026-08-03, both configurations exhausted.** With
  `Odom/ResetCountdown=0` it tracks continuously for 14 s at **ATE 0.033 m** and then
  **latches dead** for the remaining 60 s while the aircraft flies a full square. With
  `=1` it survives the whole mission and drifts **11–35% of path** — the same band as the
  ICP it was meant to beat. Accurate-but-dead, or alive-but-wrong.
- **OpenVINS — built 2026-08-08, produces no pose.** Standalone `ov_msckf` from source;
  rtabmap's own build reports `With OpenVINS: false`. It starts, tracks 47 features, and
  never leaves initialisation: `not enough feats to compute disp: 0,47 < 15`. **This is
  the live blocker.**

**Yaw drift rate — the budget that actually binds — has never been measured for any
candidate**, because none has yet survived long enough on position for yaw to be the
deciding term.

> **One measurement worth carrying.** rtabmap's gravity alignment alone (roll/pitch only,
> no position or yaw propagation) extended `rgbd_odometry`'s tracked span **2.81 m →
> 15.69 m**, ~5.6×. Attitude drift was demonstrably feeding the matching failure, which
> is the concrete, *measured* argument for a visual-inertial candidate rather than a
> general one. n = 1 valid run; do not quote it without a repeat.

**What may and may not be claimed.** ✅ "Flies autonomously with GNSS fusion disabled, on
an externally supplied pose" — **naming the source as simulator truth, not an estimator,
in the same sentence.** ✅ "The planner flies on a map the aircraft built from its own
depth camera." ❌ **Never "GPS-denied navigation."** Nothing is estimating anything.
Per [BUILD.md §0.6](./BUILD.md) the map half was met with **octomap rather than nvblox**,
so that claim is "architecture proven," not "nvblox validated."

### The measurement crisis — the most useful thing that happened in August

**Gate A moved 30% → 25% → 35% → 30% on 2026-08-03 with the aircraft and the estimator
untouched.** Every swing came from fixing the instrument or testing a claim that had
never been tested. **Three separate readings of the same four flights were each
confidently wrong**, and each looked reasonable at the time.

Six bugs were fixed in `eval_vio_drift.py` across 2026-08-02/03, **all the same shape:
it reported a good number for a bad run.** It PASSed a flight that never armed; its yaw
alignment fitted on pre-takeoff samples so `atan2(0,0)` returned `+0.0°` on *every run
this project had ever done*; its headline used final drift, which a returning square
drives to zero; the alignment then applied its offset **backwards**; it spanned the whole
flight and rotated two real runs ~180° to flatter them; and nothing measured whether the
estimate had *moved at all*. Worst of the six: **`lost/null: 0` was never true** — rtabmap
signals a dropout with an **all-zero pose, not a NaN**, and `isfinite()` scored every one
as "the aircraft is at the origin."

**Consequence for the record: no Gate A number produced before 2026-08-03 is comparable
with one after.** Three of the four flights behind the old "ATE/path 5–7%, ~5× better
than ICP" headline are not valid measurements at all.

Two things came out of it that are worth more than the bug fixes:

1. **`check_vio_score.py`** — the numeric core extracted with **no ROS dependency**, plus
   ten synthetic flights with answers known by construction (frozen, noisy-frozen,
   mirrored, fifth-scale, runaway, never-armed, a known 13.5° offset, and a good run that
   must still pass). Runs in ~0.1 s with no ROS, no Gazebo and no aircraft. Every one of
   the six bugs survived for weeks because a *real* flight has no known answer, so the
   only available check was whether the output looked plausible — and all six produced
   output that did.
2. **Read `coverage`, never ATE alone.** On a mission that returns to its origin, an
   estimate that barely moves sits near the centroid and scores a *good* ATE for doing
   nothing. Still open, and not fixable by any statistic: **the Gate A mission returns to
   its start.** A mission that does not return would remove the flaw instead of measuring
   around it — worth doing before the next candidate is ranked.

The full catalogue is the code repo's `ISSUES.md`: 32 entries grouped by the **8 shapes**
that keep recurring, each hit 4–13 times. Its single highest-value line: **in all eight
classes the failure mode is silence or a plausible number, never an exception.** That is
why "it ran and produced a number" has repeatedly meant nothing here.

**The one defect found was fixed the same day.** ~18% of cells *behind* the aircraft
came back occupied where nothing exists. Plotting the occupied voxel centres in plan +
two elevations identified them as a sheet of ground returns at Up ≈ 0.2 m, not phantom
structure — invisible in the 2D map, obvious in 3 views. Raising `occupancy_min_z` took
them to **0.0%** and sharpened obstacle contrast from 4.6–9.5× to **21–37×**. So the
rewire is unblocked.

**Two blockers found during Day 4, both now fixed.** (1) `depth_bridge.launch.py`
defaulted to a placeholder TF that conflicted with `px4_tf_publisher`, giving
`camera_link` two parents; the default is now off. (2) `gz sim` grew at a flat
**180 MB/s** and was OOM-killed twice, the second time taking the desktop session down
— root-caused to the OakD-Lite's **unused 1920×1080 RGB camera**, which Gazebo renders
unconditionally. An overlay model deletes that sensor; the sim now holds ~490 MB
indefinitely and Day 4 still passes. The depth camera never leaked.

### Research track — stage 1 BEV localization — **~70%**

Full pipeline runs end-to-end on all 10 nuScenes mini scenes with real DPVO drift.

- **Met on injected drift:** mean ATE 1.30 → 0.82 m (37%) over 10 scenes. That's the
  stage-1 "definition of done" figure.
- **Not met on real VIO drift** — and that's the honest finding. On real DPVO residuals
  the cross-track matcher *regresses* (0.74 → 0.82 m). A causal experiment refuted the
  first explanation (along-track gating) and isolated a bimodally-aliased cross-track
  measurement. The 2026-07-13 ambiguity gate fixed the alias (phantom floor 0.29 →
  0.03 m) but made the matcher **inert, not helpful** — the gated sweep never flips.
- **Conclusion recorded:** stop tuning cross-track; the path is **2-DOF (along-track +
  heading)**, since real drift is along-dominated (0.57 vs 0.40 m).
- Remaining ~30%: the 2-DOF matcher, then RPE + a paired significance test.

### Everything downstream — **0%**

Phases 2–5 have not begun. No hardware ordered, no funding confirmed, and the two ⚠️
BUILD.md §0.5 items are **still unverified** — the Isaac ROS × Jazzy × Orin Nano matrix
has now been carried as urgent since 2026-06-02 and has not been checked. Overall
project ≈ **15%**.

---

## Phases (scope ladder — each is a defensible deliverable)

### Docking — the critical path since 2026-09-10

| Gate | Deliverable | When | % |
|---|---|---|---|
| **D0** | Tag relative pose in sim, checked against truth | Week 1 | 🔄 **flown 2026-09-10, FAIL** — accurate 0–1.25 m, gate asks 1.5 m; blocked on a camera/tag decision |
| **D1** | Repeatable stationary-pad landing in sim | Week 2 | 0% |
| **D2** | UAV-only chase landing on a moving pad — the baseline | Weeks 3–4 | 0% |
| **D3** | Cooperative vs. chase, 20-trial matrix → **the Fall result** (December) | Weeks 5–6 | 0% |
| **D4** | Hardware: airframe flies repeatably; UGV holds a repeatable speed | Jan–Feb | 0% |
| **D5** | Physical docking reproduces the sim numbers → C-UASC | Mar–Apr | 0% |
| *stretch* | GPS-denied approach phase — the Blue Skies version, needs Gate A | — | — |

D3 alone is a complete senior project. Each later rung adds to it; none is required for
it to stand.

### GPS-denied layer — off the critical path since 2026-09-10

1. **Sim** — obstacle avoidance working in SITL · **~46%** (the three gates above; obstacle
   avoidance ✅, GNSS-off flight ✅, the *estimator* ❌)
2. **Hardware bring-up** — X500 manual flight; cuVSLAM + nvblox on the bench with the real RealSense · **0%**
3. **Integration** — closed-loop indoor autonomous obstacle avoidance · **0%** *(Gate B is
   this phase's core question, answered early in sim)*
4. **Outdoor / under-structure** — the GPS-denied "anywhere" demo · **0%**
5. *(stretch)* custom 3D-printed frame · *(stretch)* custom STM32H7 flight-controller PCB · **0%**

---

## Do next — docking (2026-09-10)

1. ⬜ **First project meeting** with the three `docking/` docs. Confirm there: whether the
   AVL ground platform is available, what ground truth the lab has, and CPP eligibility
   for C-UASC.
2. 🔄 **D0 in sim — flown 2026-09-10, FAIL at 1.25 m.** Built: the docking world and a
   0.6 m pad with a 5-tag bundle, `tag_pose_node`, and a harness that scores every frame
   against Gazebo truth. **Decide how to extend the range** — narrower lens (~70°,
   recommended), 1280×960, or bigger corner tags — then re-fly. Code repo `DOCKING.md` §6.1,
   `docking/results/d0/`.
3. ✅ **Write the evaluator's test before the first number.** Done for D0:
   `check_d0_score.py`, 12 synthetic flights with known verdicts. It caught a scorer bug
   before any flight — a pad layout rotated 90° passed. D1's touchdown evaluator needs its
   own, before its first trial.
3a. ⬜ **D1 needs its own touchdown and disarm logic.** PX4 took up to 61 s to detect a
   landing on the raised pad, even a stationary one (code repo `ISSUES.md` K8).
4. ⬜ **Blue Skies NOI by Oct 12** if the backup is to stay live.
5. ⬜ **C-UASC registration opens Nov 1.**

---

## Do next — GPS-denied research layer (off the critical path since 2026-09-10)

> The list below is unchanged. It no longer gates the senior project; it gates the Blue
> Skies version and the VIO career track.

> **Re-prioritised 2026-07-31.** Parts are ~90% chosen (Orin Nano Super + D435i) but
> the **budget does not unlock until the course starts**, so nothing can be ordered.
> The binding constraint is no longer money — it is **fall time**, which will go to
> assembly, bring-up, calibration, integration and mentoring new teammates. The
> remaining summer's job is to arrive with the learning curves already climbed.
> The Phase-1 gate no longer gates anything (budget already decided the buy), so it is
> now a milestone, not a decision. Details: [SUMMER.md](./SUMMER.md).
>
> **Amended 2026-08-13.** Items 1–3 below are still the right priorities and items 2 and
> 3 have still not been started. Item 1's step 2 is **done as of 2026-09-01** — OpenVINS
> on EuRoC MH_01, ATE 0.221 m — and it did exactly what the note under item 1 predicted
> it would. Steps 1 and 3 remain untouched, and they are the half with no project excuse
> attached.


1. ⭐ **The VIO ladder** ([ROADMAP.md](./ROADMAP.md) steps 1–3: Labbé's filters →
   OpenVINS on EuRoC → a toy VI-EKF). The career deliverable, entirely offline, and
   **the fall has no room for it.** `~/DPVO` already has `evaluate_euroc.py` and EuRoC
   logs, so the infrastructure is half-built. Highest-value hours available.

   > **The order got inverted, and that is now costing something (2026-08-08).**
   > OpenVINS was built and pointed at the *aircraft* first, skipping step 2. It has
   > never been run on EuRoC — a dataset where it is known to work, where the config is
   > published, and where a failure is unambiguously yours. So when it stalled at
   > `not enough feats to compute disp: 0,47 < 15`, there was **no working reference to
   > diff against**, and the ruled-out list had to be built from scratch (`ISSUES.md`
   > §I2). **Running EuRoC now is both the career deliverable and the cheapest available
   > diagnostic for the live blocker.** That is the strongest argument this list has ever
   > had for item 1, and it was produced by not doing it.
   >
   > ✅ **Step 2 done 2026-09-01, and the argument above held exactly.** OpenVINS on
   > EuRoC MH_01: **ATE 0.221 m over 73 m of path** (0.114 m once settled), stock
   > `euroc_mav` config, the same binary that stalls on the aircraft. It initialises in
   > 0.0012 s and recovers a gyro bias matching EuRoC's documented value, so **the build
   > and the estimator core are sound and the aircraft's fault is Gazebo-side.** One
   > session bought a halving of the search space that three weeks of first-principles
   > debugging had not. **Steps 1 and 3 are still open** — and they were always the part
   > that no blocker would ever justify for you.
2. ⭐ **De-risk Isaac ROS on the laptop.** Every decision so far deferred it (octomap
   not nvblox, rtabmap not cuVSLAM, and now OpenVINS not cuVSLAM) and
   [BUILD.md §0.6](./BUILD.md) warns it is "unverified for longer". The box can run it —
   RTX A3000 6 GB, driver **595.84**, 308 GB free — but Docker and the NVIDIA Container
   Toolkit are **still not installed**. Install, pull the container, run the cuVSLAM
   quickstart on a dataset. **If the Isaac ROS × Jazzy × Orin Nano matrix needs Humble,
   July is a cheap time to learn that and October with parts on the bench is not.**

   > **This has now been carried as urgent for ten weeks and deferred four times**, each
   > time by a decision that was individually correct. That pattern is the warning:
   > nothing in the sim work will ever force the question, which is exactly what §0.6
   > predicted. It is a documentation check plus a container pull.
   >
   > ⚠️ **Related, and already paid for once (2026-08-03):** the A3000's kernel module
   > was built for `6.17.0-{29,35}-generic` while the box runs `7.0.0-28-generic`, so
   > `nvidia-smi` failed, Gazebo silently fell back to the Intel iGPU and **segfaulted
   > mid-sweep**, invalidating one run and probably two more. RTF read ~1.0 and the
   > arming delays were healthy throughout — **the usual health checks all passed.**
   > Fixed with `linux-modules-nvidia-595-open-generic-hwe-24.04`, which keeps tracking
   > future kernels. Note `prime-select` is `on-demand`, so the driver being loaded and
   > Gazebo *using* it are different claims — check `nvidia-smi --query-compute-apps`.
3. ⬜ **Practice Kalibr on a dataset.** Camera–IMU calibration is the one thing sim
   cannot teach — in sim the extrinsic is exact and free; on hardware it is vibration,
   thermal drift and a fiddly toolchain. EuRoC ships calibration data.
4. ✅ **Close the Phase-1 gate: point `planner_node` at `/projected_map`.** **Done
   2026-07-31** — flown, two legs, 0.03 m and 0.12 m arrival error, with a real detour
   around a mapped obstacle. Took more than the predicted afternoon: the remap was one
   line, but four silent failures had to be fixed around it (`PHASE1_GATE.md` §2–3).
5. ⬜ **Research track:** the 2-DOF (along-track + heading) matcher — the cross-track
   line is closed out with evidence. See `bev_gps_denied/README.md` in the code repo.
   **Untouched since 2026-07-13**; all August effort went to the flight side.
6. ✅🟡 **Day 6 — the timebox is spent, and it produced two closed candidates.** Both
   original blockers are resolved: `/fmu/out/vehicle_odometry` was EKF2's estimate, and
   simulator truth now arrives over DDS via a two-line `dds_topics.yaml` patch (**no
   `PosePublisher` overlay was needed** — the pose topic existed all along, PX4 just
   never exported it). `ratio = 0.000000` was a **latch**, not an inability to register:
   one failed frame cleared the velocity model and `Odom/ResetCountdown` defaults to
   never-reset, so a single bad frame wedged odometry permanently. `=1` took
   registration 4.3% → 80.3%. **Per this item's own instruction, ICP parameter work
   stopped there** — the residual failure is an unobservable DOF, and geometry does not
   respond to tuning.
7. 🔴 **UNBLOCK OPENVINS INITIALISATION — this is the live blocker.** Built, configured
   from the running system, subscribing and tracking 47 features, and it never leaves
   initialisation, so `/odom` never appears and the sweep reports `rig_no_odom`. The
   decisive datum: **halving `init_window_time` left both numbers EXACTLY unchanged**,
   which kills the obvious track-lifetime explanation and points at the feature database
   holding a single timestamp. Next step is to print what `_db` actually contains, not to
   tune another threshold. **Read `ISSUES.md` §I2 before theorising** — it holds the
   evidence and the ruled-out list.

   > **Bounded 2026-09-01, not fixed.** The EuRoC reference now exists (item 1), and the
   > same binary initialises correctly there — so the build and the estimator core are
   > sound and this is a Gazebo-side fault. **The cheapest next experiment is therefore a
   > comparison, not a derivation: re-run EuRoC in the aircraft's *mono* config**
   > (`max_cameras: 1`, `init_window_time: 1.0`, `try_zupt: true`). ~15 minutes, no code.
   > If `0,47` reproduces there, this is a configuration bug and closes outright.
   >
   > ⚠️ **And fix `ISSUES.md` A4 before the first scored flight.** OpenVINS subscribes to
   > the IMU with `SensorDataQoS()` — best-effort, depth 5 — against 200 Hz. On EuRoC that
   > silently turned a healthy run into ATE 14682 m, and only 0.25x playback fixed it. The
   > aircraft cannot replay slower, and Gazebo loads that executor harder than a bag does.
8. ⬜ **Measure yaw drift rate.** 0.5 °/s is the budget that binds and **nothing measures
   it** — `eval_vio_drift.py` has no attitude handling at all. Needs `/odom` orientation
   against `/fmu/out/vehicle_attitude_groundtruth`.
9. ⬜ **Gate C, which needs no estimator and has not been started.** `fake_vio` injects
   drift of a chosen magnitude, so map smear and collision margin under drift are
   measurable *today*, for the same reason Gate B used synthetic truth. This is the only
   remaining sim work that does not depend on Gate A at all.
10. ⬜ Audit-enroll in **UPenn Aerial Robotics** (Coursera).

*(Hardware items — verify Orin Nano vs. old Nano, order the D435i — move to the fall
with the budget. Parts are ~90% decided; the §0.5 ⚠️ **Isaac ROS × Jazzy support matrix**
check does **not** wait, and is folded into item 2.)*

> Don't buy the rest of the BOM until the Phase-1 sim gate passes (except the long-lead RealSense/Jetson).

---

## Progress log

| Date | Track | What happened |
|---|---|---|
| 2026-09-10 | docking | **Gate D0 flown: FAIL, usable envelope 0–1.25 m against a 1.5 m gate.** From touchdown to 1.25 m: ≥ 98% of in-view frames get a pose, p95 ≤ 1.1 cm and ≤ 0.7°. Above that, the 0.18 m tags stop decoding on a 640×480, 100° camera, and the 1–2-tag poses left are 6–30° off. The limit is sensing geometry, not the estimator. Three runs; two found rig defects first (near clip, best-effort image loss). Also found: **every Gazebo camera leaks at frame bandwidth** — including the August "measured safe" one — and Gazebo maps a plane's texture rotated 90° |
| 2026-09-10 | plan | **Senior project re-scoped to cooperative UAV–UGV docking** (RSCL Project 2). GPS-denied work becomes the research/mission layer. Three planning docs reconciled into `docking/`: one schedule, one competition pair (C-UASC primary, Blue Skies backup), and Fall stated as a sim study that hardware tests for transfer. PX4 v1.18-alpha already ships a moving-platform world, its controller plugin and a downward-camera X500; four gaps recorded in `docking/REQUIREMENTS_AND_THEORY.md` |
| 2026-09-01 | drone | **VIO ladder step 2 done, and it bounded the Gate A blocker.** OpenVINS on **EuRoC MH_01: ATE 0.221 m over 73 m of path** (0.114 m once settled), stock config, same binary that stalls on the aircraft — so the build and estimator core are sound and §I2 is Gazebo-side. **Not a Gate A score**; no pose on the aircraft, Gate A stays ~30%. Cost 52 km of phantom path first: OpenVINS subscribes to the IMU with `SensorDataQoS()` (depth 5) against 200 Hz, and at full-rate playback the same run diverges to ATE 14682 m with no warning → `ISSUES.md` A4. **Skipping a known-good control did not save the three weeks, it spent them** |
| 2026-08-13 | drone | **OpenVINS builds and runs — and will not initialise.** Tracks 47 features, never leaves init (`disp: 0,47 < 15`), so no pose. Halving the init window changed the numbers not at all, ruling out track lifetime. Gate A does not move on a candidate that has produced nothing. → `ISSUES.md` §I2 |
| 2026-08-08 | drone | **Candidate 3 built.** Standalone `ov_msckf` from source — rtabmap's packaged build reports `With OpenVINS: false`, along with every other third-party backend. The old claim "already compiled in" came from `--params \| grep OdomOpenVINS`: **a configuration surface is not a capability.** Also: rtabmap gravity alignment alone extended tracked span **2.81 → 15.69 m (~5.6×)**, the measured argument for visual-inertial |
| 2026-08-03 | drone | **`rgbd_odometry` CLOSED, both configurations.** `ResetCountdown=0`: ATE 0.033 m for 14 s, then latches dead. `=1`: survives, drifts 11–35% of path — the same band as the ICP it was meant to beat. **The launch file's stated reasoning was wrong on both halves**, and blocked a ten-minute experiment for a week |
| 2026-08-03 | drone | **Six evaluator bugs fixed; three changed conclusions rather than decimals.** `lost/null: 0` was the evaluator not recognising rtabmap's all-zero lost pose. **No Gate A number from before this date is comparable with one after.** `check_vio_score.py` built as the structural fix — ten synthetic flights, known answers, ~0.1 s, no ROS |
| 2026-08-02 | drone | **Error budget measured** — 20 runs, four knobs. Yaw **0.5 °/s** (hard wall), position **~1.5 m accumulated**, latency 200 ms, no noise ceiling below 0.6 m. EKF2 passes bias through one-for-one and filters zero-mean error hard |
| 2026-08-01 | drone | **Gate B FLOWN** — armed, flew a 5 m square to 3.16 m and landed with `EKF2_GPS_CTRL=0`, on a pose PX4 did not compute. Est vs truth max 0.199 m. Exposed a bug that had corrupted **every integer PX4 parameter this repo sets** since the script was written, and that only became fatal once GPS was off |
| 2026-07-31 | drone | **Phase-1 gate flown** — `planner_node` swapped onto `octomap`'s `/projected_map`; two autonomous legs planned on a map built by the aircraft while flying, arriving 0.03 m / 0.12 m from goal, with a 35.2 m detour around a mapped wall on a 16.3 m straight line. → `PHASE1_GATE.md` |
| 2026-07-30 | drone | Days 4–5 — depth camera into ROS 2, occupancy map from depth (octomap), 20° camera tilt, forest world, unknown-space policy. Gazebo RAM leak root-caused to the unused RGB camera. |
| 2026-07-13 | drone | **First fully autonomous end-to-end SITL flight** — synthetic map → A\* → offboard, no human input. Three integration bugs fixed (topic-name drift, periodic-replan stall, PX4 arming params). → `SITL_FLIGHT.md` |
| 2026-07-13 | research | Ambiguity gate fixed the one-lane alias but made cross-track matching **inert**. Cross-track-only line closed out; 2-DOF is the path. |
| 2026-07-11 | research | Causal experiment (`dpvo-cross` / `dpvo-along`) **refuted** the along-track-gating hypothesis; isolated a biased/aliased cross-track measurement. |
| 2026-07-11 | research | Stage 1b — DPVO wired in for **real** VIO drift on all 10 scenes, replacing the injected random walk. Negative result: matcher regresses (0.74 → 0.82 m). |
| 2026-07-08 | drone | `planner_node` + `astar` + `fake_world` built; A\* validated headlessly and on real nuScenes HD-map rasters. |
| 2026-07-07 | research | Semantic gap closed + filtered fusion → mean ATE 1.30 → 0.82 m over 10 scenes on injected drift. |
| 2026-06-02 | plan | Project plan written (this folder). |
