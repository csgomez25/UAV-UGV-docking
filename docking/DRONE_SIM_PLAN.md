# Drone and Simulation Plan

*Docking team · RSCL@CPP Project 2 · agreed October 13, 2026*

The plan for the drone and its simulation through mid-December. It is written for the whole team: what the drone side is aiming for, how a landing is judged, what gets built in what order, and what the UGV, electronics and docking-mechanism work can count on. The overall status, the UGV and the competitions are in [STATUS_AND_PLAN.md](./STATUS_AND_PLAN.md). Everything here is simulation (PX4 autopilot and Gazebo); nothing claims GPS-denied flight.

## Goal

Show, in simulation, that the cooperative drone lands on a moving pad at **1.0 and 1.5 m/s** on several different paths at least **75% of the time (15 of 20 landings)**, where the camera-only drone cannot follow a turn.

- **Cooperative ("coop"):** the drone uses its camera and the UGV's broadcast of its speed, heading and turn rate.
- **Camera-only ("chase"):** the drone uses its camera alone. This is the baseline.

Whether 1.5 m/s can be reached on sharp corners is a research question. If it cannot, that is reported as a result, not hidden.

**Target: the drone side of the simulation essentially complete by mid-December 2026.** The senior design project is due in late April or early May 2027.

## Where it stands

First scored test, 1.0 m/s, 20 paired landings per path:

| Path | Camera-only | Cooperative |
|---|---|---|
| Straight line | 19 of 19 | 20 of 20 |
| Constant turning (sine) | 0 of 20 | 11 of 20 |
| Sharp corners (rounded square) | 0 of 20 | 7 of 20 |

Cooperation clearly helps on turns, but it is not yet at 75%. The largest single cause of failed cooperative landings is the drone touching down on target and then sliding as the pad turns under it, before the motors stop.

## How a landing is judged

| Measure | Meaning |
|---|---|
| **Docked (the headline)** | The drone touched within 10 cm of the pad centre **and** stayed on the deck |
| Touchdown error | How far from the centre it first touched |
| Slide | How far it moved after touching, and the sideways force needed to hold it in place. This is the number the latch or electromagnet design needs |
| Time to dock | From the pad starting to drive until touchdown |
| After an aborted landing | The drone gets one more full attempt. Landing on the moving pad counts as docked, noted as "after a second attempt". If the UGV has to stop for the drone to land, that is a failure that was recovered. Landing on the ground is a failure |

"Reliable" means at least 15 of 20, judged separately for each path and each strategy. Every limit is fixed before the flights it grades and is never moved to make a result pass.

## Test setup

- **Radio link:** a generic link with a 200 ms delay in each direction. This changes only if better parts are approved.
- **Start:** the pad is already moving when the drone is ready. The drone starts 1 to 3 m away, with no hint of where the pad is. Once the drone finds and lines up over the pad every time, the start moves out to 3 to 5 m.
- **Paths:** a straight line, a constant turn, sharp corners, and a random path that stands in for a UGV steering around obstacles.
- **Trials:** 20 paired landings for a scored test (both strategies from the same start). A 40-pair final run if time allows.

## Levels of cooperation

| Level | What the UGV shares | Status |
|---|---|---|
| State | Its speed, heading and turn rate right now | Built |
| Preview | The turn it is about to make, half a second ahead | Planned this fall |
| Landing window | It eases its turns for the few seconds the drone is touching down | Only if the first two are not enough at 1.5 m/s |

The preview means the UGV decides each turn half a second before making it. That small delay is the cost of cooperating and will be reported.

## Steps

| # | Step | Status |
|---|---|---|
| 1 | One command that runs every automated check, run automatically on every change | ✅ Done October 13 |
| 1.5 | Make the test runs reliable, and fix the bugs a full code review found | ✅ Done October 16 |
| 2 | Judge touchdown and slide separately, and re-score the existing results | ✅ Done October 16 |
| 3 | Split the landing logic from the autopilot interface, and write a one-page card for each subsystem | ✅ Done October 27 |
| 4 | Stop the slide after touchdown, move to the new start, and fly a new 1.0 m/s baseline | Code done October 28: slide fix, new start, filter re-tuned, 360 s flight limit, 3 tries per flight. Coop latency sweep done October 28 (in simulation, 15 flights per path per delay): docked 1-5 of 15 on the sine and 1-8 of 15 on the rounded square, more at longer delay. The cause is not the delay: the drone starts its 2 s final drop on a straight and lands in the next turn (93% of square landings at 25 ms, 57% at 200 ms; every sine landing), 10-19 cm off. Next: fix landing in turns, then the 1.0 m/s baseline |
| 5 | A shorter final drop onto the pad | Look done October 29 (in simulation): the drone now waits at 0.25 m above the pad instead of 0.5 m, and no longer climbs back up below 0.5 m in a corner. Sine: 9 of 10 docked, typical miss 5.7 cm (was 5 of 15, 12.6 cm). Rounded square: 5 of 10, no aborts; every miss started its drop at the end of a straight and landed in the next corner |
| 6 | The half-second preview from the UGV | Next. First use: only start the final drop if the UGV says no sharp turn is coming in the next ~1.2 s (the sine's gentle turns still allow it; a square corner does not) |
| 7 | The random path | |
| 8 | The 1.5 m/s tests (0.5 m/s only if 1.5 cannot be made to work) | |
| 9 | Recovery after an aborted landing: one more attempt, then ask the UGV to stop | |
| 10 | Gusts: a sideways push on the drone that varies in strength and direction, flown once the 75% mark is met | |
| 11 | A fairer camera-only baseline that estimates the turn from the camera, then the final run | |
| 12 | Fly farther and longer, then find the pad from a distance: GPS when far, UWB in the middle, the camera tags up close | Planned October 29 |

**About step 1.5.** Before building more, every part of the drone code was reviewed, and the bugs that could make a result wrong or stop a long test run were fixed, each with an automated check that proves it. A test run now survives a lost message to the autopilot, a simulator that will not shut down, or an interruption, and picks up where it stopped. A landing the drone refused to attempt because the test was set up wrong is no longer counted as a failed landing. None of the published results changed. The automated checks went from 12 to 18.

If time runs short, the 40-pair final goes first, then the gust flights, then the fairer baseline. Steps 2, 3, 4, 6 and the 1.0 and 1.5 m/s tests are never cut.

## What the rest of the team can count on

The drone side needs only three things from the UGV:

1. **Its true position**, in simulation only, to score landings.
2. **Its broadcast:** speed, heading and turn rate, and later the turn it is about to make.
3. **The deck's size.**

Anything behind those three can change without touching the drone side. That includes a UGV model with real wheel and deck physics, which can replace the current moving pad and be tested on the same paths.

Work the UGV, electronics and docking-mechanism areas can pick up:

- **UGV model:** a simulated UGV with real physics that provides the three items above.
- **Stop on request:** the UGV slowing to a stop when the drone asks, so an aborted landing can be recovered.
- **Announcing turns:** the UGV committing to each turn half a second ahead and broadcasting it.
- **Latch or electromagnet:** sized from the slide distance and holding force that step 2 reports.

Each subsystem has a short card (step 3, `SUBSYSTEMS.md` in the code repository) stating what it does, what goes in and out with units, how to test it alone, and its known limits, so pieces can be built and connected without reading each other's code.

## Not this fall

- The full cycle of taking off from the deck, flying out, returning and docking again (spring, on hardware).
- F Prime, NASA's flight software framework: being looked at as part of a possible NASA ORBIT entry.
- A Pixy2 camera on the UGV: its role is still to be confirmed.
- A second landing algorithm: a written survey of options only.
- Real wind in the simulator, as opposed to a sideways push.

## Dates

| Date | What |
|---|---|
| Nov 16, 2026 | NASA ORBIT registration closes (entry still being decided) |
| Mid-December 2026 | Drone side of the simulation essentially complete |
| Dec 14, 2026 | NASA ORBIT first overview due, if entered |
| Feb 22, 2027 | NASA Gateways to Blue Skies proposal and video |
| Late April or early May 2027 | Senior design project due |

The detailed version of this plan, with file names and test commands, is `DRONE_SIM_PLAN.md` in the code repository.
