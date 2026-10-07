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

> **Docking status (2026-10-07): D3, the chase-vs-coop comparison, is built, checked, and
> flown as looks. On both turning paths coop now lands half the time where chase never does,
> including the rounded square's sudden corners at a 200 ms link. All in sim, 6 trials per
> cell: looks, not the scored D3 result.**
>
> | Gate | Result | Date |
> |---|---|---|
> | M0 — the pad robot drives its path | ✅ 9/9, then ✅ **15/15** with line and sine paths × 5 speeds, position p95 ≤ 2.4 cm | 09-14 / 09-22 |
> | D0 — the tag gives a relative pose | ❌ 0–1.25 m at 640×480 → ✅ **0–2.5 m at 1280×960**, p95 ≤ 2.3 cm, ≤ 1.1° | 09-10 / 09-14 |
> | H0 — hold over the pad by camera alone | ✅ 3/3 heights, hold p95 ≤ 2.8 cm | 09-15 |
> | D1 — land on the static pad, 20 trials | ✅ **20/20**, touchdown error median 1.2 cm, max 2.7 cm (tolerance 10 cm) | 09-15 |
> | D2 — chase baseline, land on the moving pad, 20 trials | ✅ **20/20 at 0.5 m/s**, median 2.1 cm, max 4.2 cm | 09-21 |
> | D3 — chase vs coop | 🔄 rig built and checked (stage B passed); looks flown on line, sine, rounded square at 1.0 m/s | 10-05 → 10-07 |
>
> **D3 looks, 1.0 m/s, 200 ms link, 6 trials each (docked):**
>
> | Path | Chase | Coop |
> |---|---|---|
> | line (control: no turns) | 6/6 | 6/6 |
> | sine (always turning, smoothly) | **0/6** | **3/6** (5.5-9.1 cm; misses 11.0, 13.6, 15.3 cm) |
> | rounded square (a sudden corner every 5.6 s) | **0/6** | **3/6** (3.7-6.9 cm); every earlier run at 200 ms: 0/6 |
>
> *(Latest controller, 2026-10-06/07. Coop has varied 2-5/6 on the sine across controller
> versions and runs: six trials is a look.)*
>
> **What got coop there.** The broadcast made coop's estimate of the pad's velocity ~5x more
> accurate than chase's camera-only one, but that alone landed nothing: in every turn the
> drone fell ~45 cm behind its own target, because PX4 was given position and velocity but no
> acceleration. **Feeding forward the turn's acceleration** (turn rate × speed, known only to
> coop) cut the drone's offset from ~47 to ~19 cm (p95) and gave the first coop landings. The
> camera-only diagnosis from D2 (a wrong velocity estimate) turned out to be incomplete:
> tracking lag dominated.
>
> **Then the end of the descent.** Every coop miss was the final half-metre running into a
> turn. Two things fixed most of it:
> * **A rig flaw, found 2026-10-06: every D1-D3 descent had run at 0.2 m/s**, not the documented
>   0.3, because PX4's simulator kept a slow-descent cap from D0 in its saved parameters. The
>   runners now set and record every flight parameter, every trial (confirmed by flight). Recorded
>   results stand as what the aircraft did; descriptions are corrected.
> * **A fast final drop, timed to the pad's turning** (coop only knows it): descend slowly to
>   0.5 m, hold, and drop at 0.5 m/s only when the turn is gentle or easing, the offset is under
>   8 cm and the drone is at the hold height; never pause once dropping. That took the rounded
>   square from 0/6 to 3/6 and brought the sine's misses within 11-15 cm. A first version failed
>   (it stalled mid-drop) and an earlier "hold during turns" gate failed too; both are recorded.
>
> **What still limits it.** The final drop takes ~2.1 s, longer than the sine's quiet windows, so
> it still overlaps the next turn. One square landing was good (6.0 cm) and then the drone came
> off the pad at the next corner: keeping a landed drone on a turning pad is its own problem.
> Beyond D3, the planned next level of cooperation is the UGV sharing what it is *about* to do
> (intent) or agreeing to drive straight while the drone lands (a negotiated landing).
>
> **Checked before any D3 number (stage B):** the chase baseline is unchanged with all the coop
> code present (line 6/6, sine 0/6, as D2 measured); chase never heard the broadcast (12/12
> trials, asserted per trial); coop at 0 ms can win (3/6 on the square). Code, results and the
> full account, in the code repo (merged to `main` 2026-10-07):
> [`DOCKING.md`](https://github.com/csgomez25/UAV-UGV-docking-CodeStack/blob/main/DOCKING.md) §4.1-4.4 (mechanism and plan),
> [`docking/gates/d3/results/`](https://github.com/csgomez25/UAV-UGV-docking-CodeStack/tree/main/docking/gates/d3/results) (every run, with what it shows and
> does not show), [`ISSUES.md`](https://github.com/csgomez25/UAV-UGV-docking-CodeStack/blob/main/ISSUES.md) section L (the parameter trap).

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
| **D2** | UAV-only chase landing on a moving pad — the baseline | Weeks 3–4 | ✅ **flown 2026-09-21, 20/20 at 0.5 m/s**; characterised to 1.5 m/s and on 3 path shapes, 09-22 |
| **D3** | Cooperative vs. chase, 20-trial matrix → **the Fall result** (December) | Weeks 5–6 | 🔄 **~50%**: broadcaster, coop estimator, controller guard, scorer, runner built; stage B checks passed; looks flown (coop 3/6 vs chase 0/6 on the sine at 1.0 m/s). Scored cells not flown |
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
3. 🔄 **D3 in progress.** Built: `ugv_broadcaster`, the coop estimator, the controller's strategy
   guard, acceleration feedforward, the timed final drop, `d3_score.py` (+ its check, written
   first) and `run_d3.py`; runners set and record PX4's parameters. Stage B passed. Next, in
   order: (a) shorten the final drop toward ~1.5 s, and look at the drone leaving the pad after
   landing; (b) a chase arm that estimates turn rate from the camera, so the comparison is fair;
   (c) the latency sweep (stage C); (d) the scored 20-pair cells (stage D, ~13 h of sim).
4. ✅ **Write the evaluator's test before the first number.** Done for every gate through D3
   (`check_d3_score.py`: 13 trial cases + 8 pairing/cell checks).
   ⬜ **Beyond D3 (plan, `DOCKING.md` §4.3):** L2 intent (the UGV shares its planned turn rate a
   short time ahead) and L3 negotiated landing (the UGV holds a straight on request).
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
| 2026-10-07 | docking | **Timed final drop: the rounded square docks at 200 ms.** With PX4's descent cap lifted (line 12/12, final 0.5 m 2.2 → 1.5 s) a faster final drop alone did not help the sine (coop 2/6). Timing it to the pad's turning did: v2 (start when the turn is gentle or easing, offset < 8 cm, drone at the hold height; never pause) gave coop **3/6 on the rounded square** (every earlier 200 ms run 0/6) and **3/6 on the sine** with misses 11-15 cm; chase 0/6 on both; line 6/6 both. v1 had stalled mid-drop (0/6). One good square landing then came off the pad at the next corner. Code PR merged to `main`. |
| 2026-10-06 | docking | **D3 stages A and B flown, and a rig flaw found.** Stage B passed: chase baseline unchanged with the coop code present (line 6/6, sine 0/6), chase never heard the broadcast (12/12), coop at 0 ms docks the rounded square 3/6. Stage A (200 ms): coop sine 3/6, rounded square 0/6; chase rounded square 0/6. A turn gate on the descent failed and is off. **PX4's saved parameters had capped every D1-D3 descent at 0.2 m/s** (set during D0, never reset): results stand, descriptions corrected (`ISSUES.md` L1). The cooperation ladder (state → intent → negotiated landing) is written up as the plan beyond D3. |
| 2026-10-05 | docking | **D3 wired and first flown.** Coop's camera frames were being dropped after each broadcast (a third of them); fixed with a replay buffer, chase unchanged. Estimator and controller take a strategy; the controller refuses to arm unless the broadcast topic matches it. First paired look: coop's velocity estimate ~5x better but **both** arms failed on the sine, ~45 cm behind their own target in every turn. **Acceleration feedforward** fixed that: coop then docked **5/6** on the sine (chase: 0/6 in D2). |
| 2026-09-22 | docking | **The chase baseline was characterised past its gate speed, and its limit identified.** At 1.0 and 1.5 m/s it aborted every trial, so the moving-pad alignment rule and a stale-target bug were fixed first (both fair to coop, which shares the controller); it still docked 0/5 at both. Two new pad paths then separated the two possible causes: at 1.0 m/s the chase docks **6/6 on a straight line** (median 2.2 cm) and **0/6 on a sine wave**, sitting ~30 cm off the pad with ~27 cm of it sideways. **It can keep up; it cannot follow a turn** — which is the quantity D3's broadcast supplies. Pad paths re-checked 15/15 (M0). |
| 2026-09-21 | docking | **Gate D2 passed: the chase baseline lands on the moving pad, 20/20 at 0.5 m/s**, touchdown error median 2.1 cm, max 4.2 cm. Every landing came after the pad's first corner, where the chase overshoots ~25 cm and pauses its descent. The code repo's `docking/` is now organized one folder per gate. |
| 2026-09-15 | docking | **Weeks 1–2 done in sim.** At 1280×960 the tag pose holds to 2.5 m. The drone lands on the stationary pad by camera alone, 20/20 trials, touchdown error median 1.2 cm. |
| 2026-09-14 | docking | **M0 and H0 flown.** The pad robot drives its path (9/9 patterns/speeds); the drone hovers over the pad by camera alone (3/3 heights). D0 re-flown at 1280×960: usable range extends 1.25 m → 2.5 m. |
| 2026-09-10 | docking | **Gate D0 flown: FAIL, usable envelope 0–1.25 m against a 1.5 m gate.** The limit is sensing geometry (tag size vs. camera FOV/resolution), not the estimator. |
| 2026-09-10 | plan | **Senior project re-scoped to cooperative UAV–UGV docking** (RSCL Project 2). GPS-denied work becomes the research/mission layer. |

Full pre-pivot log (GPS-denied phase, 2026-06-02 through 2026-09-10):
[archive/GPS_DENIED_LOG.md](./archive/GPS_DENIED_LOG.md).
