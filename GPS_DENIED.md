# The GPS-denied research layer

The project began in summer 2026 as a GPS-denied drone: a quadrotor that navigates without GPS
with all compute onboard. Since 2026-09-10 the senior project is cooperative UAV–UGV docking
(see the [README](./README.md)); the GPS-denied work continues as the **research/mission layer**
on top of it (a drone that inspects under structures without GPS and docks on a moving ground
robot to recharge), off the critical path. Docking takes its relative pose from a fiducial, so it
never waits on this layer. This page holds that layer's status and plan, moved here from the
README on 2026-10-08 without changes. Full day-by-day history: [archive/](./archive).

> **GPS-denied layer status:** the aircraft flies with GNSS fusion off; nothing estimates
> its pose yet. The perception→planning→flight loop closes in SITL on a map the
> aircraft builds itself, and the X500 has flown a full autonomous square with no GNSS
> fusion at any point, on a pose supplied from outside the autopilot. Say that with
> **"the pose source is simulator truth, not an estimator"** in the same breath. The
> remaining work is the estimator itself. Full breakdown and history:
> [archive/GPS_DENIED_LOG.md](./archive/GPS_DENIED_LOG.md); current status and next
> steps: [Where things stand](#where-things-stand).

## Why the project changed direction, 2026-09-10

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

*(The competition pairing above is as decided on 2026-09-10. It was revised on 2026-10-08 after
reading the 2027 rules: NASA Gateways to Blue Skies is now the primary competition and C-UASC the
flight-platform option; see the README.)*

## The plan for this layer

**GPS-denied layer (background):** stand the full autonomy stack up in **simulation**
first, learn **VIO** deeply, and secure the **long-lead parts**. Build on the
**PX4-documented reference platform** (Holybro X500 V2 + Jetson Orin Nano + RealSense +
ROS 2 Jazzy + Isaac ROS cuVSLAM/nvblox) rather than a risky custom airframe, keeping a
custom frame + custom flight-controller PCB as gated stretch goals. Prove obstacle
avoidance in sim → bring up hardware → close the loop indoors → then outdoors. Ship a
defensible deliverable via a **scope ladder** so the project can't fully fail.

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

## Phases

### GPS-denied layer — off the critical path since 2026-09-10

1. **Sim** — obstacle avoidance working in SITL · **~46%** (the three gates above; obstacle
   avoidance ✅, GNSS-off flight ✅, the *estimator* ❌)
2. **Hardware bring-up** — X500 manual flight; cuVSLAM + nvblox on the bench with the real RealSense · **0%**
3. **Integration** — closed-loop indoor autonomous obstacle avoidance · **0%** *(Gate B is
   this phase's core question, answered early in sim)*
4. **Outdoor / under-structure** — the GPS-denied "anywhere" demo · **0%**
5. *(stretch)* custom 3D-printed frame · *(stretch)* custom STM32H7 flight-controller PCB · **0%**

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

## Code for this layer

In the code repository [`UAV-UGV-docking-CodeStack`](https://github.com/csgomez25/UAV-UGV-docking-CodeStack) (private; ask to be added):

| Doc | What it's for |
|---|---|
| `GPS_DENIED_PLAN.md` | The three gates, every estimator attempt and what it measured |
| `gps_denied_autonomy/CLOSED_LOOP.md` | **Gate B**: the flight, the three silent bugs, and §8 the error budget |
| `gps_denied_autonomy/results/gate_a/README.md` | Gate A run tables and how to read them |
| `HANDOFF.md` | Picking the stack up cold: environment invariants, what works, where to pick up |

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
