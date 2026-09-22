# Cooperative UAV–UGV Docking — Project Hub

**Senior project as of 2026-09-10:** an autonomous drone that lands on a ground robot
**while the robot is still moving**, by having the two vehicles cooperate — the UGV
broadcasts its velocity and the UAV fuses that with visual relative pose — rather than
chasing a passive target (RSCL@CPP Project 2). This folder is the planning + lab-notebook
home for the team.

The repo was renamed from `GPS-Denied-Drone` to **`UAV-UGV-docking`** on 2026-09-15 (the
code repo to `UAV-UGV-docking-CodeStack` the same day; old URLs redirect). The project
began as a smaller GPS-denied-drone effort over summer 2026; that work is not abandoned —
it continues as the **research/mission layer** (a drone that inspects under structures
without GPS and docks on a moving ground robot to recharge), no longer the critical path.
See [Direction change](#direction-change--2026-09-10). Full pre-pivot history:
[archive/](./archive).

> **Docking status (2026-09-21): the drone lands on a MOVING pad by camera alone, 20/20 at 0.5 m/s, in sim.**
>
> | Gate | Result | Date |
> |---|---|---|
> | M0 — the pad robot drives its path | ✅ 9/9 (circle, rounded square, square × 0.5/1.0/1.5 m/s), position p95 ≤ 1.7 cm | 09-14 |
> | D0 — the tag gives a relative pose | ❌ 0–1.25 m at 640×480 → ✅ **0–2.5 m at 1280×960**, p95 ≤ 2.3 cm, ≤ 1.1° | 09-10 / 09-14 |
> | H0 — hold over the pad by camera alone | ✅ 3/3 heights, hold p95 ≤ 2.8 cm, from takeoff 0.94 m off the pad | 09-15 |
> | D1 — land on the static pad, 20 trials | ❌ 19/20 → ✅ **20/20**, touchdown error median 1.2 cm, max 2.7 cm (tolerance 10 cm) | 09-15 |
> | D2 — chase baseline, land on the moving pad, 20 trials | ✅ **20/20 at 0.5 m/s**, touchdown error median 2.1 cm, max 4.2 cm, scored against the pad where it was at contact | 09-21 |
>
> **Next: D3**, the cooperative controller (the UGV's velocity broadcast) against this chase
> baseline. At 0.5 m/s the chase is already near-perfect, so D3 needs faster pads (1.0 and
> 1.5 m/s) to show a difference. Still true: nothing is cooperative yet, and all of it is
> simulation. Results and runbook: the code repo's
> [`docking/`](https://github.com/csgomez25/UAV-UGV-docking-CodeStack/tree/main/docking), one folder per gate under
> [`docking/gates/`](https://github.com/csgomez25/UAV-UGV-docking-CodeStack/tree/main/docking/gates).

> **GPS-denied layer status:** the aircraft flies with GNSS fusion off; nothing estimates
> its pose yet. The perception→planning→flight loop closes in SITL on a map the
> aircraft builds itself, and the X500 has flown a full autonomous square with no GNSS
> fusion at any point, on a pose supplied from outside the autopilot. Say that with
> **"the pose source is simulator truth, not an estimator"** in the same breath. The
> remaining work is the estimator itself. Full breakdown and history:
> [archive/GPS_DENIED_LOG.md](./archive/GPS_DENIED_LOG.md); current status and next
> steps: [Where things stand](#where-things-stand).

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

**GPS-denied layer (background):** stand the full autonomy stack up in **simulation**
first, learn **VIO** deeply, and secure the **long-lead parts**. Build on the
**PX4-documented reference platform** (Holybro X500 V2 + Jetson Orin Nano + RealSense +
ROS 2 Jazzy + Isaac ROS cuVSLAM/nvblox) rather than a risky custom airframe, keeping a
custom frame + custom flight-controller PCB as gated stretch goals. Prove obstacle
avoidance in sim → bring up hardware → close the loop indoors → then outdoors. Ship a
defensible deliverable via a **scope ladder** so the project can't fully fail.

---

## Team

Expanding into a full senior-design
team for the docking project. Current role split, interface contracts, and how the team
scales as people join: [ROLES.md](./ROLES.md).

---

## Document map

| Doc | What it's for |
|---|---|
| [docking/](./docking) | ⭐ **The current project** — one-pager, first-meeting packet, requirements/parts/theory |
| [BUILD.md](./BUILD.md) | Locked technical decisions, frame/sensor/compute choices, BOM, weight math, risks |
| [ROADMAP.md](./ROADMAP.md) | Learn-just-in-time roadmap + curated resources + the VIO development track |
| [ROLES.md](./ROLES.md) | Team split, the data flow, and the interface contracts to nail |
| [archive/](./archive) | Pre-pivot solo/summer-era logs (day-by-day sim bring-up, the original plan, the full GPS-denied status history) |

**And in the code repo**, which moves faster than this one and wins on mechanism:

| Doc | What it's for |
|---|---|
| `DOCKING.md` | 🛬 **The docking mechanism** — what PX4 already provides, the four sim gaps, the node and topic plan, gates D0–D5 (plus M0 and H0), with results |
| `docking/README.md` | 🛬 **The docking runbook** — every gate's commands, nodes and topics |
| `docking/gates/` | 🛬 **One folder per gate** (`d0`, `m0`, `h0`, `d1`, `d2`): its scorer, runner, plot, launch file and `results/`. `gates/README.md` is the index |
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
are now **symlinks** into that repo, which keeps `colcon build` working from `~/ws_px4`
while the code lives under version control in one place.

> **Two repos total:**
>
> | Repo | Holds |
> |---|---|
> | [`csgomez25/UAV-UGV-docking`](https://github.com/csgomez25/UAV-UGV-docking) | this one — planning, roadmap, lab notebook (was `GPS-Denied-Drone`) |
> | [`csgomez25/UAV-UGV-docking-CodeStack`](https://github.com/csgomez25/UAV-UGV-docking-CodeStack) **(private)** | all the code — `docking/` + `gps_denied_autonomy/` + `bev_gps_denied/` (was `GPS_Denied`) |
>
> The code repo was briefly named `GPS_Denied_drone`, then `GPS_Denied`; both URLs redirect.
> The local checkout keeps its folder name, `~/gps-denied-drone-stack/`.
> Unrelated: `csgomez25/IGVC_BEV` is the ground-vehicle competition project, not this.

---

## Where things stand

### The three gates (GPS-denied layer)

Phase 1 is tracked as three separate gates rather than one blended percentage, because
they fail for different reasons:

| Gate | Question | State | % |
|---|---|---|---|
| **A — Estimate** | Is the pose estimate accurate enough to fly on? | ❌ **Open.** Two candidates closed on evidence, a third built and blocked | ~30% |
| **B — Closed loop** | Will it *fly* on an external pose, GNSS fusion off? | ✅ **Flown**, budget measured | 100% |
| **C — Survives drift** | Does the map stay usable, does margin hold, under drift? | ⬜ Not started — but **reachable without an estimator** | ~5% |
| Supporting stack (SITL) | flight, depth, map, planner, gates, harness | ✅ | ~95% |
| **Overall → GPS-denied navigation in sim** | | | **~46%** |

Gate A is the whole remaining GPS-denied problem: three VIO/estimator candidates have
been tried (`icp_odometry`, `rgbd_odometry`, OpenVINS) and none has produced a usable
pose on the aircraft yet — OpenVINS is the live blocker, stuck in initialisation.
The measured error budget any estimator has to meet: **yaw drift ≤ 0.5°/s** (hard wall),
**≤ ~1.5 m accumulated position error**, latency ≤ 200 ms, no noise ceiling found below
0.6 m. Full detail, every candidate's numbers, and the measurement-bug history:
[archive/GPS_DENIED_LOG.md](./archive/GPS_DENIED_LOG.md).

### Research track — stage 1 BEV localization — **~70%**

Full pipeline runs end-to-end on all 10 nuScenes mini scenes with real DPVO drift. Met
on injected drift (mean ATE 1.30 → 0.82 m, 37%), not yet met on real VIO drift — the
along-track/cross-track finding and the path to close it are in the archive log.

### Everything downstream — **0%**

Phases 2–5 (hardware bring-up onward) have not begun. No hardware ordered, no funding
confirmed. Overall GPS-denied project ≈ **15%**; docking is now the active critical path.

---

## Phases (scope ladder — each is a defensible deliverable)

### Docking — the critical path since 2026-09-10

| Gate | Deliverable | When | % |
|---|---|---|---|
| **D0** | Tag relative pose in sim, checked against truth | Week 1 | 🔄 **flown 2026-09-14, PASS** at 1280×960, 0–2.5 m |
| **D1** | Repeatable stationary-pad landing in sim | Week 2 | ✅ **flown 2026-09-15, 20/20** |
| **D2** | UAV-only chase landing on a moving pad — the baseline | Weeks 3–4 | ✅ **flown 2026-09-21, 20/20 at 0.5 m/s** |
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

## Do next — docking

1. ⬜ **First project meeting** with the three `docking/` docs. 
2. ✅ **D2 done 2026-09-21** — the chase baseline lands on the moving pad, 20/20 at 0.5 m/s.
3. ⬜ **D3 next** — `ugv_broadcaster` (latency, noise, dropout) and the cooperative controller,
   then chase vs. coop at 1.0 and 1.5 m/s, where the chase should start to struggle.
4. ✅ **Write the evaluator's test before the first number.** Done for every gate through D2
   (`check_<gate>_score.py`, synthetic runs with known verdicts). D3's needs its own.
5. ⬜ **Blue Skies NOI by Oct 12** if the backup is to stay live.
6. ⬜ **C-UASC registration opens Nov 1.**

## Do next — GPS-denied research layer (off the critical path)

> This no longer gates the senior project; it gates the Blue Skies stretch version and
> the estimator work. Full detail, including every ruled-out hypothesis:
> [archive/GPS_DENIED_LOG.md](./archive/GPS_DENIED_LOG.md).

1. ⭐ **Unblock OpenVINS initialisation** — the live Gate A blocker. Tracks 47 features
   and never leaves init. Verified sound on EuRoC (ATE 0.221 m), so the fault is
   Gazebo-side; next step is a mono-config EuRoC comparison, not another threshold tweak.
2. ⭐ **De-risk Isaac ROS in a container on the dev box** — the Isaac ROS × Jazzy × Orin
   Nano support matrix still hasn't been checked, and it gates hardware bring-up.
3. ⬜ **Measure yaw drift rate** — the budget term that actually binds, and nothing
   currently measures it.
4. ⬜ **Gate C** — `fake_vio` can inject drift today, so map smear and collision margin
   under drift are measurable without waiting on an estimator.
5. ⬜ **Kalibr practice** on a public dataset — camera–IMU calibration, the one thing sim
   can't teach.

*(Don't buy the rest of the BOM until parts are needed for docking hardware, D4.)*

---

## Progress log

| Date | Track | What happened |
|---|---|---|
| 2026-09-21 | docking | **Gate D2 passed: the chase baseline lands on the moving pad, 20/20 at 0.5 m/s**, touchdown error median 2.1 cm, max 4.2 cm. Every landing came after the pad's first corner, where the chase overshoots ~25 cm and pauses its descent. The code repo's `docking/` is now organized one folder per gate. |
| 2026-09-15 | docking | **Weeks 1–2 done in sim.** At 1280×960 the tag pose holds to 2.5 m. The drone lands on the stationary pad by camera alone, 20/20 trials, touchdown error median 1.2 cm. |
| 2026-09-14 | docking | **M0 and H0 flown.** The pad robot drives its path (9/9 patterns/speeds); the drone hovers over the pad by camera alone (3/3 heights). D0 re-flown at 1280×960: usable range extends 1.25 m → 2.5 m. |
| 2026-09-10 | docking | **Gate D0 flown: FAIL, usable envelope 0–1.25 m against a 1.5 m gate.** The limit is sensing geometry (tag size vs. camera FOV/resolution), not the estimator. |
| 2026-09-10 | plan | **Senior project re-scoped to cooperative UAV–UGV docking** (RSCL Project 2). GPS-denied work becomes the research/mission layer. |

Full pre-pivot log (GPS-denied phase, 2026-06-02 through 2026-09-10):
[archive/GPS_DENIED_LOG.md](./archive/GPS_DENIED_LOG.md).
