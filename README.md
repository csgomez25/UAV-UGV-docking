# Cooperative UAV–UGV Docking

**RSCL@CPP Project 2, Cal Poly Pomona.** A drone that lands on a ground robot **while the robot is
still moving**, by having the two vehicles cooperate: the UGV broadcasts how it is moving, and the
drone fuses that with what its camera sees of the landing pad, instead of chasing a passive target.
This repository is the team's shared planning home: status, plan, competitions, roles and the lab
notebook. The code and every result live in the code repository (see [Repositories](#repositories)).

> 📌 **Start here:** [`docking/STATUS_AND_PLAN.md`](./docking/STATUS_AND_PLAN.md), the shared
> overview of the question, approach, results, competitions, the UGV, the timeline and the
> decisions in front of the team.

## Status

> **Docking status (2026-10-07): D3 has its first scored result. On both turning paths, coop
> docks where chase never does, and the difference is beyond chance: 11/20 vs 0/20 on the sine,
> 7/20 vs 0/20 on the rounded square, 1.0 m/s, 200 ms link. All in sim. This is the moving
> baseline later changes are measured against, not the final D3 result.**
>
> | Gate | Result | Date |
> |---|---|---|
> | M0 — the pad robot drives its path | ✅ 9/9, then ✅ **15/15** with line and sine paths × 5 speeds, position p95 ≤ 2.4 cm | 09-14 / 09-22 |
> | D0 — the tag gives a relative pose | ❌ 0–1.25 m at 640×480 → ✅ **0–2.5 m at 1280×960**, p95 ≤ 2.3 cm, ≤ 1.1° | 09-10 / 09-14 |
> | H0 — hold over the pad by camera alone | ✅ 3/3 heights, hold p95 ≤ 2.8 cm | 09-15 |
> | D1 — land on the static pad, 20 trials | ✅ **20/20**, touchdown error median 1.2 cm, max 2.7 cm (tolerance 10 cm) | 09-15 |
> | D2 — chase baseline, land on the moving pad, 20 trials | ✅ **20/20 at 0.5 m/s**, median 2.1 cm, max 4.2 cm | 09-21 |
> | D3 — chase vs coop | 🔄 **first scored cells: coop 11/20 vs chase 0/20 (sine), 7/20 vs 0/20 (rounded square), 20/20 vs 19/19 (line control)** at 1.0 m/s | 10-08 |
>
> **D3 baseline, all three paths** (runs `20261007_135910` and `20261008_142024`, 20 chase/coop pairs per cell from the same seeded starts,
> fresh seed never used for tuning, 200 ms noiseless link, 1.0 m/s):
>
> | Path | Chase docked | Coop docked | Paired: coop-only / chase-only | Exact McNemar p |
> |---|---|---|---|---|
> | sine (always turning, smoothly) | 0/20 (95% CI 0–16%) | **11/20** (34–74%) | 11 / 0 | **0.001** |
> | rounded square (a sudden corner every 5.6 s) | 0/20 (0–16%) | **7/20** (18–57%) | 7 / 0 | **0.016** |
> | line (control: no turns) | 19/19 valid (83–100%) | **20/20** (84–100%) | 0 / 0 | — (no discordant pairs) |
>
> In 40 pairs, chase never docked when coop failed. Coop touched down on all 20 sine trials
> (median 8.2 cm off centre) and on 15 square trials (median 5.9 cm). **Its largest failure mode:
> landing within tolerance, then skidding 3-10 cm in the ~0.35 s before the motors disarm** as the
> pad turns beneath it, ending a centimetre over the deck edge (5 of 22 failures; on the sine,
> 15/20 landings were within 10 cm at contact). The line control, flown to the same standard,
> docks chase 19/19 and coop 20/20.
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
> **What still limits it** (from the baseline's failures, diagnosed 2026-10-08). Five coop landings
> were within tolerance at contact, then skidded 3-10 cm in the third of a second before disarm,
> while still armed with near-hover thrust, as the pad turned (correlation 0.87 with the pad's
> turning); after disarm they did not move. Cutting thrust at contact is the fix, and a hardware
> requirement too. One more landing was a real loss: touchdown was detected 2.5 s late. Five sine landings missed by 0.6-2.3 cm: the final drop takes
> ~2.1 s, longer than the sine's quiet windows. On the square, 5 coop trials never touched and 6
> landed 17-22 cm off: its corners are still hard at a 200 ms link.
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

## How it works

![System pipeline: the camera path is shared by both strategies; the UGV broadcast is coop-only](docking/img/status_pipeline.png)

Both strategies see the pad with the same downward camera and the same 5-tag AprilTag bundle.
**Coop** also receives the UGV's speed, heading and turn rate over a degraded radio link (200 ms
late by default, never position), which corrects its estimate of the pad's velocity and lets the
controller anticipate turns. Mechanism in full: `DOCKING.md` in the code repository.

## Plan and timeline

![Timeline, October 2026 to June 2027](docking/img/status_timeline.png)

Simulation first: the full chase-vs-coop comparison runs in PX4 software-in-the-loop with Gazebo,
where ground truth is exact, and is presented in December. Hardware follows in the spring and
tests whether the result transfers. The Blue Skies proposal depends only on the simulation
results, so a hardware delay cannot block it.

### Gates (each a defensible deliverable)

| Gate | Deliverable | When | % |
|---|---|---|---|
| **D0** | Tag relative pose in sim, checked against truth | Week 1 | ✅ **flown 2026-09-14, PASS** at 1280×960, 0–2.5 m |
| **D1** | Repeatable stationary-pad landing in sim | Week 2 | ✅ **flown 2026-09-15, 20/20** |
| **D2** | UAV-only chase landing on a moving pad — the baseline | Weeks 3–4 | ✅ **flown 2026-09-21, 20/20 at 0.5 m/s**; characterised to 1.5 m/s and on 3 path shapes, 09-22 |
| **D3** | Cooperative vs. chase, 20-trial matrix → **the Fall result** (December) | Weeks 5–6 | 🔄 **~60%**: everything built and checked; **first scored cells at 1.0 m/s: coop 11/20 vs chase 0/20 (sine, p 0.001), 7/20 vs 0/20 (rounded square, p 0.016)**. Line control: both arms dock (chase 19/19, coop 20/20). Still to fly: 1.5 m/s, latency sweep, chase fairness arm |
| **D4** | Hardware: airframe flies repeatably; UGV holds a repeatable speed | Jan–Feb | 0% |
| **D5** | Physical docking reproduces the sim numbers (the hardware result; the airframe also serves C-UASC) | Mar–Apr | 0% |
| *stretch* | GPS-denied approach phase — the Blue Skies version, needs Gate A | — | — |

D3 alone is a complete senior project. Each later rung adds to it; none is required for
it to stand.

### Do next

1. ⬜ **Share [`docking/STATUS_AND_PLAN.md`](./docking/STATUS_AND_PLAN.md) with the whole team** and agree on who takes which area ([ROLES.md](./ROLES.md)).
2. ✅ **D2 done 2026-09-21** — the chase baseline lands on the moving pad, 20/20 at 0.5 m/s.
3. 🔄 **D3 in progress.** Built: `ugv_broadcaster`, the coop estimator, the controller's strategy
   guard, acceleration feedforward, the timed final drop, `d3_score.py` (+ its check, written
   first) and `run_d3.py` (with checkpoints and resume); runners set and record PX4's parameters.
   Stage B passed. **First scored cells flown: the moving baseline** (above). Next, each measured
   against the baseline on the same cells: (a) cut thrust at contact, to stop the skid before disarm (coop's largest
   failure mode); (b) shorten the final drop toward ~1.5 s; (c) a chase arm that estimates turn rate from the
   camera, so the comparison is fair; then the latency sweep (stage C) and 1.5 m/s (stage D).
4. ✅ **Write the evaluator's test before the first number.** Done for every gate through D3
   (`check_d3_score.py`: 13 trial cases + 8 pairing/cell checks).
   ⬜ **Beyond D3 (plan, `DOCKING.md` §4.3):** L2 intent (the UGV shares its planned turn rate a
   short time ahead) and L3 negotiated landing (the UGV holds a straight on request).
5. ⬜ **Blue Skies notice of intent by Oct 12**, the primary competition (non-binding; it keeps the entry open).
6. ⬜ **C-UASC registration opens Nov 1** (closes Feb 1): decide whether to enter the airframe; every pilot needs AMA membership.

## Competitions

Chosen from the RSCL list after reading each competition's 2027 rules (October 8, 2026). Detail
and the full comparison: [`docking/STATUS_AND_PLAN.md`](./docking/STATUS_AND_PLAN.md#competitions).

| Role | Competition | Why | Key dates |
|---|---|---|---|
| **Primary** | **NASA Gateways to Blue Skies 2027**, "InfraAir: Aviation for Infrastructure Inspection" | A concept competition with no hardware requirement; cooperative docking for corridor inspection (a drone that docks on a moving ground vehicle to recharge or hand off data) is the entry itself, and the simulation results support it | Notice of intent **Oct 12, 2026** · proposal + video **Feb 22, 2027** · finalist forum May 2027 |
| **Flight platform** | **C-UASC 2027** (Cal State LA) | Flight tasks (waypoints, delivery, recovery, target localization); no docking task, but precision landing transfers to delivery and recovery, and docking fits its design and innovation award | Registration Nov 1, 2026 – Feb 1, 2027 · design May 1 · flight June 4–6, 2027 |

Following the RSCL guide: one primary and one backup, and competition scoring kept separate from
the research contribution (Rules 6 and 7).

## Team

The docking project is a team effort. The areas of work, the interfaces between them, and how
to join one: [ROLES.md](./ROLES.md).

## Document map

| Doc | What it's for |
|---|---|
| [docking/STATUS_AND_PLAN.md](./docking/STATUS_AND_PLAN.md) | ⭐ **Start here.** The shared status and plan |
| [docking/ONE_PAGER.md](./docking/ONE_PAGER.md) | The project on one page: title, hypothesis, metrics, schedule, the Week 6 data table |
| [docking/FIRST_MEETING_PACKET.md](./docking/FIRST_MEETING_PACKET.md) | The RSCL guide's bring-items and the 10-rule self-check |
| [docking/REQUIREMENTS_AND_THEORY.md](./docking/REQUIREMENTS_AND_THEORY.md) | Parts, ground truth, the theory checklist, the simulation gaps |
| [docking/REFERENCES.md](./docking/REFERENCES.md) | The three key papers, with open links and how they compare to our design |
| [ROLES.md](./ROLES.md) | Areas of work, the data flow, and the interface contracts between them |
| [ROADMAP.md](./ROADMAP.md) | The learning roadmap and curated resources |
| [BUILD.md](./BUILD.md) | Airframe, sensor and compute decisions, BOM, weight math, risks |
| [GPS_DENIED.md](./GPS_DENIED.md) | The GPS-denied research layer: its status, gates, phases and next steps |
| [archive/](./archive) | Pre-pivot summer-era logs |

**And in the code repo**, which moves faster than this one and wins on mechanism:

| Doc | What it's for |
|---|---|
| `DOCKING.md` | 🛬 **The docking mechanism** — what PX4 already provides, the four sim gaps, the node and topic plan, gates D0–D5 (plus M0 and H0), with results |
| `docking/README.md` | 🛬 **The docking runbook** — every gate's commands, nodes and topics |
| `docking/gates/` | 🛬 **One folder per gate** (`d0`, `m0`, `h0`, `d1`, `d2`, `d3`): its scorer, runner, plot, launch file and `results/`. `gates/README.md` is the index |
| `HANDOFF.md` | 🔁 **Picking the project up cold.** Environment invariants to check *every session*, what works vs. what is structurally broken, the measured budget, and where to pick up. Read §1–§2 first |
| `TOUR.md` | 🧭 **New here.** Every node and script in a paragraph or less |
| `ISSUES.md` | 🐛 Every bug long-form, grouped by the **8 shapes** that keep recurring, plus the open list ranked by cost |

## Progress log

| Date | Track | What happened |
|---|---|---|
| 2026-10-08 | docking | **Line control and the skid diagnosis.** The line, flown to the baseline's standard (20 pairs, seed 2), docks chase 19/19 valid and coop 20/20: without turns, sharing adds nothing, as predicted. The baseline's "off the deck" failures were diagnosed from its records: five skidded 3-10 cm in the ~0.35 s between contact and disarm as the pad turned, and ended a centimetre over the deck edge; one was a touchdown detected 2.5 s late. The planning repo became the team's shared starting point (status and plan, key references). |
| 2026-10-07 | docking | **First scored D3 cells, now the moving baseline.** 20 chase/coop pairs per cell at 1.0 m/s, 200 ms, on a fresh seed: **sine coop 11/20 vs chase 0/20 (McNemar p 0.001); rounded square coop 7/20 vs 0/20 (p 0.016)**; both cells valid (80/80 flights). Chase never docked when coop failed. Coop's largest failure: landing within tolerance, then skidding off-centre before disarm (diagnosed 10-08). The runner gained checkpoints and `--resume`, tested by killing a run mid-flight. |
| 2026-10-07 | docking | **Timed final drop: the rounded square docks at 200 ms.** With PX4's descent cap lifted (line 12/12, final 0.5 m 2.2 → 1.5 s) a faster final drop alone did not help the sine (coop 2/6). Timing it to the pad's turning did: v2 (start when the turn is gentle or easing, offset < 8 cm, drone at the hold height; never pause) gave coop **3/6 on the rounded square** (every earlier 200 ms run 0/6) and **3/6 on the sine** with misses 11-15 cm; chase 0/6 on both; line 6/6 both. v1 had stalled mid-drop (0/6). One good square landing then came off the pad (not diagnosed for that run; a similar baseline case was a late disarm). Code PR merged to `main`. |
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

## Background: the GPS-denied research layer

The project began in summer 2026 as a GPS-denied drone. That work continues as the research and
mission layer on top of docking (inspecting under structures without GPS, then docking on a moving
ground robot to recharge), off the critical path: docking takes its relative pose from the tag, so
it never waits on it. The scored docking flights use GPS, and nothing yet estimates pose without
it, so "GPS-denied docking" is not claimed. Status, gates and next steps:
[GPS_DENIED.md](./GPS_DENIED.md).

## Repositories

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

*This repository was renamed from `GPS-Denied-Drone` to `UAV-UGV-docking` on 2026-09-15, and the code
repository to `UAV-UGV-docking-CodeStack` the same day; old URLs redirect.*
