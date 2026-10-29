# Key papers: landing a drone on a moving ground vehicle

Three papers the docking design builds on, with a link to a freely available copy of each and
what each means for our work. The PDFs are not stored here: they are published, copyrighted
papers, so we link to the authors' and preprint copies instead of redistributing them.

| Paper | Read it | What it is to us |
|---|---|---|
| Borowczyk, Nguyen, Nguyen, Nguyen, Saussié, Le Ny, "Autonomous Landing of a Quadcopter on a High-Speed Ground Vehicle," *Journal of Guidance, Control, and Dynamics* 40(9), 2017 | [arXiv preprint](https://arxiv.org/abs/1611.07329) · [authors' PDF](https://www.professeurs.polymtl.ca/jerome.le-ny/docs/journals/2017_JGCD_MAVlanding.pdf) · [video](https://youtu.be/ILQqD2xQ4tg) | **The closest prior work to our coop strategy.** A phone on the car broadcasts its GPS and IMU data to the drone, fused with an AprilTag in one Kalman filter; landings up to 50 km/h on a mostly straight track |
| Falanga, Zanchettin, Simovic, Delmerico, Scaramuzza, "Vision-based Autonomous Quadrotor Landing on a Moving Platform," IEEE SSRR 2017 | [authors' PDF](https://rpg.ifi.uzh.ch/docs/SSRR17_Falanga.pdf) · [video](https://youtu.be/Tz5ubwoAfNE) | **The counterpart of our chase strategy.** No communication with the platform at all: onboard vision only, a constant-velocity platform model, and fast trajectory replanning |
| Baca, Stepan, Spurny, Hert, Penicka, Saska, Thomas, Loianno, Kumar, "Autonomous Landing on a Moving Vehicle with an Unmanned Aerial Vehicle," *Journal of Field Robotics* 36(5), 2019 | [publisher](https://onlinelibrary.wiley.com/doi/abs/10.1002/rob.21858) · [ResearchGate copy](https://www.researchgate.net/publication/330155244) · [videos](http://mrs.felk.cvut.cz/jfr2018landing) | **Why turns are the hard part.** The MBZIRC 2017 winners found a linear motion model fails in turns, switched to a car-like model with a map of the track, planned 8 s ahead with MPC, and landed only on straight sections |

### Four more, read October 28-29, 2026

| Source | What it shows | What it is to us |
|---|---|---|
| Wang, Wang, Wang, Meng, "Quadrotor Autonomous Landing on Moving Platform," [arXiv 2208.05201](https://arxiv.org/abs/2208.05201), 2022 | Vision (AruCo) and replanning; **straight paths only, 0.2-0.9 m/s**, 4 real flights, **no landing error reported** | It descends to 0.2 m, then lands quickly: the idea behind our lower hold (0.25 m), which took sine coop from 5/15 to 9/10 in simulation |
| "Design of a Quadrotor Drone Following and Landing System on a Mobile Platform," [juyeuav.com](https://juyeuav.com/design-of-a-quadrotor-drone-following-and-landing-system-on-a-mobile-platform/) (web article, no authors listed) | GPS far, UWB in the middle (~10 m to 2 m), camera close; **no landing error reported** | The staged-sensor idea behind our step 12 (fly farther, find the pad from a distance) |
| Mangsatabam, "Control Development for Autonomous Landing of Quadcopter on Moving Platform," MS thesis, Florida Institute of Technology, 2018 | PD control; the moving target is **simulation only**, tracking error up to ~0.6 m | States our problem: a plain controller lags a target that changes direction |
| Saavedra-Ruiz, Pinto-Vargas, Romero-Cano, "Monocular visual autonomous landing system for quadcopter drones using software in the loop," [arXiv 2108.06616](https://arxiv.org/abs/2108.06616), 2021 | **Static pad**; ~2-3 cm in simulation, ~13-16 cm on real hardware (5× worse) | Our static-pad result in simulation is as good (median 1.2 cm); a warning that real flights will be worse than simulated ones |

None of the four lands within 10 cm on a pad that is turning, which is what our sine and rounded-square cells test.



| | Borowczyk 2017 | Falanga 2017 | Baca 2019 | Ours |
|---|---|---|---|---|
| Vehicle shares its state | Yes: GPS position, speed, heading (1 Hz), IMU (25 Hz) | No | No (uses a track map instead) | Chase: no. Coop: speed, heading, turn rate at 20 Hz, never position, with modelled latency, noise and dropout |
| Pad estimator | Linear Kalman filter, constant acceleration | EKF, constant speed and heading | UKF, car-like model with curvature | Linear Kalman filter; coop switches to a coordinated turn using the broadcast turn rate |
| Steering | Proportional navigation, then PD | Min-jerk replanning at 50 Hz | MPC (8 s horizon) + nonlinear controller | Position setpoint with velocity and (coop) acceleration feedforward |
| Evidence reported | Successful landings at 30–50 km/h, no trial counts | One run plotted per case | 54% of 22 early trials; 3 of 3 competition landings | 20 paired trials per cell, confidence intervals, McNemar tests |

**The gap our study fills:** none of the three compares sharing against not sharing under the
same conditions, measures how link latency affects the result, or reports success rates over
repeated paired trials. That comparison is the D3 result.

A fuller, illustrated comparison (pipelines and Kalman filters side by side) was written up on
2026-10-05; the summary above carries its conclusions.
