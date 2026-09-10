# Drone Docking on a Moving UGV — First-Meeting One-Pager

**Program:** Hybrid Autonomous Robotics · **Role:** Autonomy · **Term:** Fall 2026

Companions: [`FIRST_MEETING_PACKET.md`](./FIRST_MEETING_PACKET.md) (the six bring-items and the
10-rule self-check) · [`REQUIREMENTS_AND_THEORY.md`](./REQUIREMENTS_AND_THEORY.md) (parts,
ground truth, theory). All three share one title, hypothesis, metric set, schedule and
competition pair — if they ever disagree, that is a bug in the docs.

---

## Project title (as a scientific claim)
**Cooperative UAV–UGV Docking: Reducing Relative Landing Error Under Platform Motion and Disturbance**

## Hypothesis (one sentence)
A cooperative controller in which the UGV broadcasts its velocity/heading and the UAV fuses that with visual relative pose achieves **lower touchdown error and higher docking success at higher relative speeds** than a UAV-only chase controller that treats the platform as a passive target.

## Two primary metrics (with units)
1. **Touchdown position error** — **cm** (mean ± spread over the trial matrix), per control strategy.
2. **Docking success rate** — **% of attempts** that land within the pad tolerance, as a function of **platform speed (m/s)**.

*Supporting measurements:* max reliable relative speed (m/s), settling time (s), wind/disturbance tolerance, mode/estimator failure rate (%).

## Where the result comes from
**Fall is a simulation study, and is reported as one.** The full chase-vs-cooperative trial
matrix runs in PX4 SITL + Gazebo, where ground truth is exact. Spring hardware exists to test
whether that result **transfers** — it reproduces the sim numbers on a real airframe and UGV,
it does not replace them. Every plot says which of the two it is.

## Six-week minimum prototype (simulation — mapped to what I already have)
| Week | Task | Output | Reuses from my work |
|------|------|--------|---------------------|
| 1 | UGV-sized pad with AprilTag in the moving-platform world; downward camera on the sim UAV; tag detection → relative pose | Working relative-pose read | PX4/ROS 2 Jazzy SITL stack, Gazebo model-overlay mechanism, TF bridge |
| 2 | Repeatable **stationary-pad** autonomous landing | Baseline landing works | Offboard manager (heartbeat, arming, sim-time guard) |
| 3 | Pad moving at **one fixed low speed**; **UAV-only chase** controller | Comparison baseline | — |
| 4 | Logged runs: stationary + one moving speed, exact touchdown truth | **Baseline dataset** | Trial-sweep harness |
| 5 | **Cooperative** controller: UGV velocity broadcast (with realistic latency/noise) fused with visual relative pose, used as feedforward | The research core | Multi-camera perception, sensor-fusion experience |
| 6 | **20-trial matrix**: 2 strategies × ≥2 platform speeds | **First plot worth discussing** | Drift-budget sweep methodology already demonstrated |

**Full arc:** Weeks 1–6 sim → **December presentation** (mission + statistics). Jan–Feb: build
minimal hardware, gated on the airframe **flying repeatably** before any docking attempt.
**Mar–Apr:** physical docking that reproduces the sim numbers. C-UASC design submission May 1;
flight Jun 4–6, 2027.

## Definition of done
At least **two control strategies** (UAV-only chase vs. cooperative) compared on the **same trajectory**, reported as **landing-error distributions** across the trial matrix — not one successful video.

## Data table expected by Week 6
| Strategy | Platform speed (m/s) | Touchdown error (cm) | Success rate (%) | Settling time (s) |
|----------|----------------------|----------------------|------------------|-------------------|
| UAV-only chase | low / high | — | — | — |
| Cooperative (velocity broadcast + fusion) | low / high | — | — | — |

## Competition target + backup
- **Primary — C-UASC 2027 (California).** Physical fly-off; rewards docking, computer vision, and autonomous mission systems. Registration opens **Nov 1, 2026**, closes **Feb 1, 2027**; design submission May 1; flight event **Jun 4–6, 2027**. Confirm CPP eligibility with organizers early.
- **Backup — NASA Gateways to Blue Skies 2027.** NOI **Oct 12, 2026**, proposal + video **Feb 22, 2027**. Docking reframes as cooperative air-ground inspection (drone lands on a moving ground unit to recharge/hand off mid-inspection). Sim data alone carries it, so it **survives a hardware slip**.
- *Not targets:* RoboNation SUAS 2027 is a watch item — log autonomous flights so the airframe is mature if registration opens. IGVC 2027 (Jun 4–8) overlaps C-UASC. DASC 2026 (Sep 15) is out: it scores nav + object-ID, not docking, and needs an already-flying airframe.

## GPS policy
The scored flight runs **with GPS** — reliable, boring, points on the board. GPS-denied
navigation is the research/mission layer (the Blue Skies inspection story), never a dependency
of the docking result: docking takes relative pose from the fiducial, so it stands whether or
not the VIO work is finished.

## One risk that could kill the project + how I de-risk it
**Risk:** the UGV effectively stops at the moment of landing, reducing the problem to a static-pad demo (the guide's explicit *DO NOT DO THIS*) — or the physical airframe isn't flight-ready in spring, leaving only sim results.
**De-risk:** (1) hold platform speed non-zero through touchdown and log the platform's actual velocity at contact as evidence the target was moving; require the cooperative strategy to win *at speed*, not at rest. (2) Run the complete study in sim first, so a hardware slip still leaves a finished, clearly-labelled sim result and the Blue Skies backup intact — hardware then tests transfer rather than carrying the whole project.

## Open items to confirm before committing
- **UGV hardware:** is the AVL ground platform available to this project, or is a rover a purchase?
- **Ground truth:** does the lab have motion capture, or is it the overhead-camera option?

---
*Reality check (as of Sep 10, 2026): verify all competition rules and eligibility on the official pages before committing money or travel.*
