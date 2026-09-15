# Drone Docking on a Moving UGV — First Project Meeting Packet

**Project 2 · RSCL@CPP · Role: Autonomy · Fall 2026**
Built to the guide's 6 bring-items and the 10 No-Excuses Rules.

Companions: [`ONE_PAGER.md`](./ONE_PAGER.md) (the same plan on one page, with what each week
reuses) · [`REQUIREMENTS_AND_THEORY.md`](./REQUIREMENTS_AND_THEORY.md) (parts, ground truth,
theory).

---

## 1. Scientific project title (measurable claim, not a toy — Rule 10)
**Cooperative UAV–UGV Docking: Reducing Relative Landing Error Under Platform Motion and Disturbance**

## 2. Hypothesis (one sentence)
A cooperative controller in which the UGV broadcasts its velocity/heading and the UAV fuses that with visual relative pose achieves **lower touchdown error and higher docking success at higher relative speeds** than a UAV-only chase controller that treats the platform as a passive target.

## 3. Two primary metrics (with units — Rule 2)
1. **Touchdown position error** — cm (mean ± spread), per control strategy.
2. **Docking success rate** — % of attempts within pad tolerance, as a function of **platform speed (m/s)**.

*Supporting:* max reliable relative speed (m/s), settling time (s), disturbance tolerance, mode/estimator failure rate (%).

## 4. Six-week minimum prototype plan (simulation — no hardware ordered yet; Rules 1 & 3)
| Week | Task | Output |
|------|------|--------|
| 1 | UGV-sized pad with AprilTag in SITL; downward camera on sim UAV; tag detection → relative pose | Working relative-pose read |
| 2 | Repeatable **stationary-pad** autonomous landing | Baseline landing works |
| 3 | Pad moving at one fixed low speed; **UAV-only chase** controller | Comparison baseline |
| 4 | Logged runs: stationary + one moving speed, exact touchdown truth | **Baseline dataset (Rule 9)** |
| 5 | **Cooperative** controller: UGV velocity broadcast (realistic latency/noise) fused with visual pose, used as feedforward | Research core running |
| 6 | 20-trial matrix: 2 strategies × ≥2 platform speeds | **First plot worth discussing (Rule 9)** |

**Fall is a simulation study, and is reported as one.** Spring hardware tests whether the result
transfers; it does not replace it.

**Progress at the time of the meeting:** Week 1 is built and measured. The pad's pose from
the downward camera is accurate from touchdown to 1.25 m (≥ 98% of frames, ≤ 1.1 cm and
≤ 0.7° at p95), scored against simulator truth over 2,617 frames. Above 1.25 m the tags are
too small for the simulated 640 × 480, 100° camera, which makes the lens a design decision
to settle before D1.

**Update, 2026-09-15:** Weeks 1–2 are done in sim. At 1280 × 960 the tag pose holds to
**2.5 m**. The drone lands on the stationary pad by camera alone in **20/20** trials, each
started 0.8–1.2 m off the pad (touchdown error median 1.2 cm, max 2.7 cm, tolerance 10 cm fixed
beforehand; a first run failed 19/20 on a touchdown-detection bug, kept on record). Next is
Week 3: the pad moving, UAV-only chase.

**Full arc (real timeline):** Fall = sim research → **December presentation** (mission + stats). Jan–Feb = build minimal hardware, gated on the airframe flying repeatably before any docking attempt. **Mar–Apr = physical docking that reproduces the sim numbers.** C-UASC design submission May 1; flight Jun 4–6, 2027.

## 5. Primary + backup competition (Rule 7 — exactly one each)
- **Primary — C-UASC 2027** (California physical fly-off; rewards docking, CV, autonomous mission systems). Registration opens **Nov 1, 2026**, closes **Feb 1, 2027**; design May 1; flight **Jun 4–6, 2027**. Confirm CPP eligibility with organizers.
- **Backup — NASA Gateways to Blue Skies 2027** (concept; NOI **Oct 12, 2026**, proposal + video **Feb 22, 2027**). Reuses the same data as a cooperative air-ground inspection concept — **survives even if hardware slips**, because sim data alone carries it.

## 6. One risk that could kill it + de-risk
**Risk:** the "moving platform" collapses into a static-pad demo (the guide's *DO NOT DO THIS*), or the airframe isn't flight-ready by spring, leaving only sim.
**De-risk:** hold platform speed **non-zero through touchdown** and log the actual contact velocity as proof the target was moving; require the cooperative strategy to win *at speed*. Do the full study in **sim first**, so a hardware slip still yields a complete, clearly-labelled result and the Blue Skies backup remains viable.

---

## Standing practices (fold these in from day one)

**Lab notebook (Rule 4)** — every test logs:
`date | hardware version | software commit | test conditions | result | failure`

**Repeat, don't cherry-pick (Rule 5):** report the 20-trial matrix as distributions and name the failure cases — never one successful run.

**Scoring vs. research split (Rule 6):** the flight + landing *base* must be reliable; the cooperative fusion is the *experimental* research module. Keep them separable so a research failure doesn't ground the competition platform. The same split governs GPS: the scored flight runs **with GPS**; GPS-denied navigation is the research/mission layer and never a dependency of the docking result (docking takes relative pose from the fiducial).

**Safety gate (Rule 8):** propeller / flight tests require safety approval and a written procedure **before** the first spin — applies when hardware flight begins in spring, not to Fall sim work.

---

### 10 Rules — self-check
1 ✔ question written, no hardware ordered · 2 ✔ two metrics with units · 3 ✔ minimum sim experiment first · 4 ✔ notebook format above · 5 ✔ 20-trial distributions + failures · 6 ✔ scoring/research split named · 7 ✔ one primary + one backup · 8 ✔ safety gate before props · 9 ✔ Wk4 dataset, Wk6 plot · 10 ✔ title is an engineering claim

*As of Sep 10, 2026 — official competition pages are the final authority; verify rules and eligibility before spending money or traveling.*
