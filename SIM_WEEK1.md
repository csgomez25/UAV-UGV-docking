# Week-1 Simulation Setup Checklist

> Goal for the week: a simulated X500 takes off and flies a waypoint **autonomously via offboard control**, with a depth camera streaming into ROS 2, and an occupancy map building from that depth. No hardware, no crash risk.
>
> Companion doc: [BUILD.md](./BUILD.md). Target stack: PX4 SITL + Gazebo + ROS 2 **Jazzy** + Isaac ROS (cuVSLAM/nvblox).
>
> ⚠️ **These are starting-point commands.** Package names, repo paths, and versions drift — cross-check each against the official PX4 / ROS 2 / Isaac ROS docs (linked at the bottom) as you go. Don't paste blindly.

---

## Progress (as of 2026-08-13) — Days 1–5 and 7 done; **Day 6 is the whole remaining project**

> **Read the gate as three, not one.** "Days done / 7" hides the fact that Day 6 is a
> research problem and the rest were integration problems. Current split: Estimate ❌ ~30%
> · Closed-loop-flight-without-GPS ✅ 100% · Survives-drift ⬜ ~5%. See
> [README.md](./README.md#the-three-gates--read-this-before-any-percentage).

| Day | Milestone | Status | Evidence / blocker |
|---|---|---|---|
| 0 | Machine check | ✅ | Ubuntu 24.04, RTX A3000 **6 GB**, 16 cores, 31 GB RAM, 333 GB free |
| 1 | ROS 2 Jazzy | ✅ | `/opt/ros/jazzy` |
| 2 | PX4 SITL + Gazebo, manual flight | ✅ | gz-harmonic 8.12 + `~/PX4-Autopilot` builds and flies |
| 3 | Offboard waypoint from ROS 2 | ✅ | `~/ws_px4` builds `px4_msgs` + `px4_ros_com` + `gps_denied_autonomy` |
| 4 | Depth camera into ROS 2 | ✅ **2026-07-30** | 4 topics at rate, in flight, real TF — `DEPTH_SIM.md` §2, `results/depth_bridge.png` |
| 5 | Occupancy map from depth | ✅ **2026-07-30** | `octomap_server` on `/depth_camera/points`; 3/3 obstacles mapped at 21–37× map density, open ground **and** the area behind the aircraft 0.0% occupied — `MAPPING.md`, `results/octomap_day5.png`, `results/octomap_3view.png` |
| 6 | VIO + drift number | ❌ **three candidates measured, none viable** | Registration fixed and real ground truth now exists — but `icp_odometry` is closed on geometry (32.4%), `rgbd_odometry` closed both ways (0.033 m for 14 s then a latch, or 11–35% of path), and OpenVINS produces no pose. `VIO.md`, `results/gate_a/`, `ISSUES.md` §I2 |
| 7 | Close the loop | ✅ **2026-07-31** | two autonomous legs planned on the **perceived** map (`/projected_map`), arriving 0.03 m / 0.12 m from goal; leg 2 flew 35.2 m for a 16.3 m straight line, routing around a wall the aircraft had mapped itself. Pose is still PX4's own sim state. `PHASE1_GATE.md` |
| — | **Fly with GNSS fusion off** | ✅ **2026-08-01** | not a SIM_WEEK1 day — Phase-3 work pulled forward. Armed, flew a 5 m square to 3.16 m and landed with `EKF2_GPS_CTRL=0`, on a pose PX4 did not compute. Est vs truth **max 0.199 m**, 1160/1160 armed samples valid. **The pose is simulator truth, not an estimator.** `CLOSED_LOOP.md` |
| — | **Error budget** | ✅ **2026-08-02** | 20 runs, four knobs. yaw **0.5 °/s** (hard wall) · position **~1.5 m accumulated** · latency 200 ms · noise no ceiling below 0.6 m. `CLOSED_LOOP.md` §8 |

**What's actually done:** Days 1–5 fully, and Day 7's *own code* — `planner_node`,
`offboard_manager`, `astar`, `fake_world` all built, and the closed loop flew
end-to-end in SITL (`~/ws_px4/src/gps_denied_autonomy/SITL_FLIGHT.md`).

**Day 4 closed on 2026-07-30.** `/depth_camera/image_raw` (28.8 Hz),
`/camera_info` (29.5 Hz), `/depth_camera/points` (24.5 Hz) and `/imu` (241 Hz) all
deliver, captured *during* an autonomous square with `px4_tf_publisher` supplying a
real moving `map -> base_link -> camera_link`. Runbook + numbers:
`~/ws_px4/src/gps_denied_autonomy/DEPTH_SIM.md`.

**Day 5 closed on 2026-07-30.** `octomap_server` builds a live map from
`/depth_camera/points`: three deliberately asymmetric obstacles all map to their true
positions, open ground inside the flight square comes back **0.0% occupied**, and the
sim holds ~500 MB for the whole run. Details, including two silent-failure traps
(wrong octomap parameter spellings; a sim-time/wall-time clock mismatch that drops
every cloud), are in `MAPPING.md`.

> **⚠️ Day-6 result must not be quoted — and the first explanation for it was wrong.**
> The run produced `ATE 4.32 m, final drift 2.16 m (3.38%)`, which looks respectable
> and is not. I first blamed the aircraft (a 16 m excursion at 16.4 m/s ⇒ "tree
> collision"). Chasing it ruled that out — the nearest tree is 4.38 m *outside* the
> square — and found two better answers, either of which alone voids the number:
>
> 1. **The reference is not ground truth.** `/fmu/out/vehicle_odometry` is **EKF2's
>    estimate**; PX4 exports no groundtruth topic over uXRCE-DDS at all. The comparison
>    was estimator-vs-estimator, so an EKF position jump reads as estimator "error".
>    Path length varied **36–64 m across runs of the identical commanded 5 m square**,
>    while `offboard_manager` logged orderly waypoint progression throughout.
> 2. **ICP never registered anything.** Every frame logs `ratio = 0.000000` at ~0.1 ms
>    per update, where real ICP on this cloud takes tens of ms. `/odom` is not a tracked
>    trajectory, so no statistic derived from it means anything.
>
> Also ruled out with evidence: NED/FRD frame mixing (all NED), timebase artifacts,
> a starved cloud (197 955 points at 27.5 Hz), and bad normals. Full write-up:
> `VIO.md` §3b.
>
> **Both blockers are now closed (2026-08-01/03), and the second one only half the way
> it reads above.** Ground truth arrives over DDS via a two-line `dds_topics.yaml` patch
> — **no `PosePublisher` overlay was needed.** The premise "PX4 exports no groundtruth"
> was right; the inference "so the data does not exist" was wrong. `GZBridge` has been
> filling four uORB groundtruth topics in NED the whole time; stock PX4 just never
> published them. And `ratio = 0.000000` was a **latch**, not an inability to register:
> one failed frame clears the velocity model, the next guess is null, `RegistrationIcp`
> refuses a null guess, which clears the velocity again — and `Odom/ResetCountdown`
> defaults to *never reset*, so nothing breaks the cycle. **One bad frame wedged odometry
> permanently.** `=1` took registration 4.3% → 80.3%, null-guess failures 622 → 0.
>
> ⚠️ **A harder lesson landed on top of it (2026-08-02/03): the instrument was wrong
> six different ways.** Every one had the same shape — *it reported a good number for a
> bad run.* It PASSed a flight that never armed; its yaw fit ran entirely on pre-takeoff
> samples so `atan2(0,0)` returned `+0.0°` on every run ever taken; it then applied that
> offset **backwards**; it fitted over the whole flight and rotated two real runs ~180°
> to flatter them; its headline used final drift, which a returning square drives to
> zero; and `lost/null: 0` was never true, because rtabmap signals a dropout with an
> **all-zero pose, not a NaN**, and `isfinite()` scored every one as "the aircraft is at
> the origin." **No Gate A number from before 2026-08-03 is comparable with one after.**
> The fix that matters is `check_vio_score.py` — ten synthetic flights with answers known
> by construction, ~0.1 s, no ROS and no aircraft. `ISSUES.md` §E.

**The swap is done (2026-07-31) — what's left is Day 6.** `planner_node` now consumes
`/projected_map`, and the mission flew: two autonomous legs on the perceived map, 0.03 m
and 0.12 m arrival error, with a genuine detour around a mapped wall. Write-up:
`PHASE1_GATE.md`.

> The remap was one line, as predicted. What cost the time was everything a *perceived*
> map does that a synthetic one never did — the goal is normally outside the observed
> grid, nothing publishes `/current_pose` in the real stack, inflation swallows the
> aircraft's own cell, and octomap publishes ~25× faster than a plan takes. All four
> failed silently. Also worth carrying forward: with `use_sim_time` and no `/clock`,
> ROS time never advances and **node timers never fire at all**.

So the only incomplete day is **Day 6**, and with it the thing that makes this a
GPS-denied stack rather than a mapping demo: every flight above runs on PX4's own
perfect sim pose.

> **Updated 2026-08-13 — and the framing above needs one correction.** "Every flight
> above runs on PX4's own perfect sim pose" is no longer true of *all* of them. On
> 2026-08-01 the aircraft flew a full square with **`EKF2_GPS_CTRL=0`**, on a pose fed in
> from outside the autopilot — so the closed-loop half of GPS-denied flight is done and
> measured. What that pose is, is **simulator truth, not an estimator**, and saying so in
> the same breath is what keeps the claim honest. Day 6 remains the open half: three
> candidates measured, none good enough to fly on.

> **✅ Day-5 defect found and fixed the same day.** The first working map had ~18% of
> the cells *behind* the aircraft occupied, where nothing exists. Plotting the occupied
> **voxel centres** in plan + two elevations (`viz_octomap3d.py`) identified them
> instantly as a thin sheet at **Up ≈ 0.2 m** — ground returns that survived the plane
> filter, not phantom structure. `/projected_map` is 2D, so a ground return and a 4 m
> tower look identical in it; the 3-view render is what made the difference.
> Raising `occupancy_min_z` 0.25 → 0.45 took the artifacts to **0.0%** and *sharpened*
> obstacle contrast from 4.6–9.5× to **21–37×**. Cost: obstacles under ~0.45 m are
> invisible — fine for a drone at 3 m, wrong for a ground robot.
>
> Still worth chasing separately: the free space carved is a near-complete **disc**,
> which a vehicle holding the commanded `yaw = 0` could not produce. That was the
> *exposure* mechanism, not the cause, but it suggests PX4 isn't tracking commanded
> yaw — a real bug in its own right.

> **Days 5–6 decided (BUILD.md §0.6):** `octomap_server` for occupancy in sim; Isaac
> ROS (nvblox + cuVSLAM) stays the *flight* stack on the Jetson. Isaac ROS ships only
> as NVIDIA Docker containers, and standing that up means Docker + the NVIDIA Container
> Toolkit (neither installed) plus a GPU mapper on 6 GB of VRAM — days of yak-shaving
> on tooling that gets discarded when the Jetson arrives. Read §0.6 for what this
> costs: Phase-1 tuning numbers will **not** transfer to nvblox, and this does not
> count as "nvblox validated."
>
> **Day-6 front-end decided (2026-07-30): `rtabmap_ros`.** Installed and available as
> `rtabmap_odom icp_odometry`. Chosen because it is apt-installable with no Docker, runs
> on the depth cloud we already publish, and needs no RGB — which matters, since the
> overlay model deletes the RGB camera to stop the 180 MB/s leak. It must run with
> **loop closure OFF**: rtabmap is SLAM, and loop closure corrects exactly the drift
> Day 6 exists to measure. DPVO stays where it earns its place, on the research track;
> the OpenVINS/EuRoC ladder stays the offline career deliverable.

> **Two blockers found during Day 4 — both now fixed (2026-07-30).**
> 1. ✅ **Invalid TF tree.** `depth_bridge.launch.py` defaulted `static_tf:=true`,
>    publishing `map -> camera_link` while `px4_tf_publisher` publishes
>    `base_link -> camera_link`. Both ran together, giving `camera_link` two parents —
>    and near the origin it looked fine. The default is now `false`, so bridge +
>    `px4_tf_publisher` is correct with no extra flags. Don't re-enable it.
> 2. ✅ **Gazebo's RAM leak — root-caused to the unused RGB camera.** `gz sim` grew at
>    a flat **180 MB/s** and was OOM-killed twice (15:16, 15:25 — the second took the
>    desktop session with it). It is the OakD-Lite's **1920×1080 RGB camera at 30 Hz**,
>    matching `1920×1080×3×30 = 178 MiB/s`. The **depth** camera does not leak at all.
>    Fixed with an overlay model that deletes the RGB sensor; the sim now holds ~490 MB
>    indefinitely and Day 4 still passes. **Requires one `export`** — see
>    `gps_denied_autonomy/sim/README.md` and `DEPTH_SIM.md` §4b.
>
> Ruled out along the way: `always_on=0` (doesn't gate rendering on subscribers) and
> the Gazebo version (system 8.14.0 leaks identically to the ROS-vendored 8.11.0 —
> note `.bashrc` makes 8.11.0 the default via `GZ_CONFIG_PATH`).

---

## Day 0 — Machine check & the one branch that decides your week

Run these and write down the answers:

```bash
lsb_release -a            # must be Ubuntu 24.04 (Noble) for Jazzy tier-1
nvidia-smi                # NVIDIA GPU present? note the answer
df -h ~                   # need ~40+ GB free (PX4 + ROS + Gazebo + containers)
nproc && free -h          # SITL + Gazebo + RViz is heavy; 8+ cores / 16+ GB ideal
```

**Confirmed: dev box has an NVIDIA RTX A3000 (Ampere, CUDA) → you are on the FULL GPU path.** cuVSLAM + nvblox run locally in sim. Notes for the A3000:

- Isaac ROS runs in **Docker** + the **NVIDIA Container Toolkit** — install that on Day 5 before pulling the Isaac ROS container.
- **VRAM watch:** the A3000 laptop GPU has 6 GB (some SKUs 12 GB). Gazebo rendering + cuVSLAM + nvblox together can pressure 6 GB. If you hit OOM, drop the sim camera resolution and the nvblox voxel resolution (e.g. 5 cm → 10 cm). Not a blocker, just a knob.
- Ampere (compute capability 8.6) is well within Isaac ROS's supported range.

*(CPU-fallback path removed — not needed with the A3000.)*

---

## Day 1 — ROS 2 Jazzy

```bash
# Locale
sudo apt update && sudo apt install -y locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8

# Enable universe + add the ROS 2 apt source
sudo apt install -y software-properties-common curl
sudo add-apt-repository universe
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" \
  | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

sudo apt update
sudo apt install -y ros-jazzy-desktop ros-dev-tools
```

Add to `~/.bashrc` (or source per-shell):
```bash
source /opt/ros/jazzy/setup.bash
```

**✅ Success check:** in two terminals, `ros2 run demo_nodes_cpp talker` and `ros2 run demo_nodes_py listener` exchange messages.

---

## Day 2 — PX4 SITL + Gazebo, fly it manually first

```bash
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
bash ./PX4-Autopilot/Tools/setup/ubuntu.sh      # installs sim deps; reboot after
cd PX4-Autopilot
make px4_sitl gz_x500                            # plain X500 in Gazebo
```

Install **QGroundControl** (AppImage) separately and launch it — it should auto-connect to SITL on UDP.

**✅ Success check:** in QGroundControl, arm and command takeoff; the X500 lifts off in Gazebo. (Or in the `pxh>` console: `commander takeoff`.) This proves SITL + Gazebo + GCS before any ROS or autonomy.

---

## Day 3 — Bridge PX4 ↔ ROS 2 and fly an offboard waypoint

**3a. Micro XRCE-DDS Agent** (the PX4↔ROS 2 link):
```bash
git clone https://github.com/eProsima/Micro-XRCE-DDS-Agent.git
cd Micro-XRCE-DDS-Agent && mkdir build && cd build
cmake .. && make && sudo make install && sudo ldconfig /usr/local/lib/
MicroXRCEAgent udp4 -p 8888                       # leave running
```
Restart SITL (`make px4_sitl gz_x500`); the PX4 client connects to the agent automatically.

**3b. ROS 2 messages + examples:**
```bash
mkdir -p ~/ws_px4/src && cd ~/ws_px4/src
git clone https://github.com/PX4/px4_msgs.git
git clone https://github.com/PX4/px4_ros_com.git
cd ~/ws_px4
source /opt/ros/jazzy/setup.bash
colcon build && source install/setup.bash
```

**✅ Success check (incremental):**
1. `ros2 topic list` shows `/fmu/out/...` topics → bridge works.
2. Run the offboard control example from `px4_ros_com` (or the PX4 ROS 2 offboard tutorial node) → the vehicle arms, switches to Offboard mode, and holds/flies a setpoint **commanded from ROS 2**.

This is the single most important milestone of the week: **ROS 2 is now flying the drone.** Everything else feeds setpoints into this.

---

## Day 4 — Depth camera into ROS 2 — ✅ **done 2026-07-30**

> Full runbook, verified topic names and results:
> **`~/ws_px4/src/gps_denied_autonomy/DEPTH_SIM.md`**. What follows is the summary.

PX4 ships an X500 variant with a depth camera (an OakD-Lite):
```bash
cd PX4-Autopilot
HEADLESS=1 make px4_sitl gz_x500_depth   # HEADLESS: the GUI + sensor rendering
                                         # does not fit 6 GB of VRAM
```

Bridge Gazebo's camera topics into ROS 2 (Jazzy pairs with Gazebo Harmonic via
`ros_gz`, installed at `1.0.22-1noble`):
```bash
sudo apt install -y ros-jazzy-ros-gz
ros2 launch gps_denied_autonomy depth_bridge.launch.py
ros2 run gps_denied_autonomy px4_tf_publisher    # the real map->base_link->camera_link
```

**Three things this cost time to learn, recorded so Day 5 doesn't repeat them:**

- **`gz topic -l` first, always.** Gazebo auto-scopes any sensor topic without an
  explicit `<topic>` tag by `world/model/link/sensor`, so the paths change if the world
  or model is renamed. This is the same class of bug as the Day-3 topic drift.
- **There is no camera-rigid IMU.** A `camera_imu` topic *name* appears in
  `gz topic -l` with nothing publishing on it. The only live IMU is the flight
  controller's on `base_link`. Fine in sim (the camera joint is fixed, extrinsic known
  exactly) — but on hardware the camera–IMU calibration is real work, and this sim
  passing does not retire it.
- **The point cloud is x-forward, not ROS's z-forward optical convention.** Assume
  wrong and Day 5's map is silently rotated 90°.

**✅ Success check — met.** All four topics deliver at rate, captured *during* an
autonomous square, with `max |first-last| = 17.2 m` across 409 frames proving the
stream tracks the world. Evidence: `results/depth_bridge.png`.

Two things the original check asked for that were **not** done, and why:
- *RViz2 visualisation* → replaced with a headless PNG checker. RViz alongside SITL's
  sensor rendering is the VRAM combination that OOM-killed the sim on Day 3. Same
  evidence, less budget. A laptop constraint, not a design choice.
- *"place an object in the Gazebo world"* → **still outstanding.** Every capture so far
  is over an empty ground plane, so nothing is within 3 m. Near-field depth is where a
  real D435i is trustworthy and where obstacle avoidance lives. Do this on Day 5.

---

## Day 5 — Occupancy map from depth — ✅ **done 2026-07-30**

> Full runbook, results and the two silent-failure traps:
> **`gps_denied_autonomy/MAPPING.md`**. Summary below.

> **Revised 2026-07-30: `octomap_server`, not nvblox.** Full rationale and accepted
> costs in [BUILD.md §0.6](./BUILD.md). nvblox stays the *flight* stack on the Jetson;
> this substitution is for the laptop sim only. The original nvblox route is kept below
> for Phase 2.

Everything octomap needs is already published by the Day-4 bridge:

| octomap input | supplied by |
|---|---|
| `PointCloud2` | `/depth_camera/points` |
| `map -> camera_link` TF | `px4_tf_publisher` (don't re-enable `static_tf` — see above) |
| `use_sim_time` | `/clock` |

```bash
python3 gps_denied_autonomy/sim/spawn_obstacles.py   # geometry to actually map
ros2 launch gps_denied_autonomy octomap.launch.py    # tuned params, see MAPPING.md
ros2 run gps_denied_autonomy px4_tf_publisher --ros-args -p use_sim_time:=true
python3 gps_denied_autonomy/check_octomap.py results/octomap_day5.png 20
```

**Two traps here, both of which fail *silently*** — nodes run, topics exist,
`ros2 topic hz` looks healthy, and the map stays empty:

1. **`use_sim_time:=true` on `px4_tf_publisher`.** The cloud is stamped in sim time
   (~68 s) while the TF publisher defaults to wall clock (~1.78e9), so octomap drops
   every cloud as *"earlier than all the data in the transform cache."*
2. **octomap's parameter names.** It ignores unknown ones without warning. The
   plausible-looking `filter_ground` and `pointcloud_min_z` are **wrong** — the real
   names are `filter_ground_plane` and `point_cloud_min_z`. Get them wrong and the
   ground plane is mapped as one solid obstacle. `ros2 param list /octomap_server`
   against a running node is the only reliable check.

`octomap_server` publishes **`/projected_map`** as a `nav_msgs/OccupancyGrid` — the
exact type `planner_node` already consumes. That is the whole point: `fake_world`'s
synthetic grid unplugs and a perceived map plugs in **with no planner changes**, which
is the Phase-1 gate.

**Both Day-4 blockers are cleared**, which is what makes this practical: the TF tree is
correct, and the sim now holds a flat ~490 MB instead of dying in minutes — so a long
mapping run is finally possible. Just remember the `GZ_SIM_RESOURCE_PATH` export, or
the RGB leak comes back.

**✅ Success check — met 2026-07-30.** Three asymmetric obstacles spawned north of the
flight square all map to their true positions (contrast 4.6–9.5× vs. the map as a
whole), open ground inside the square comes back **0.0% occupied**, and gz holds
492 → 501 MB across the run. Evidence: `results/octomap_day5.png`.

> Counting occupied cells near an obstacle is *not* a real check — on a map that is
> 90% occupied (what a broken ground filter produces) every window has hits and
> everything "passes". `check_octomap.py` also requires a **control patch** of open
> ground to come back free, and normalises each obstacle's density by what its own
> geometry allows. That is what caught the ground-filter bug.

<details>
<summary>Original nvblox route — deferred to Phase 2 on the Jetson</summary>

**GPU path (Isaac ROS nvblox):** run via NVIDIA's `isaac_ros_common` Docker container (simplest, avoids host dependency hell). Feed it the bridged depth + camera_info + a pose source.
```bash
# Use the Isaac ROS Docker container per NVIDIA's "Getting Started" + nvblox quickstart.
# Inside the container, launch nvblox subscribed to the sim depth topics.
```

Configure nvblox to also publish a **2D ESDF/distance-map slice at flight altitude** (`nav_msgs/OccupancyGrid` or `nvblox_msgs/DistanceMapSlice`) — that 2D slice is what the Day-7 planner consumes.

**✅ Success check:** the 3D voxel map *and* a 2D slice appear in RViz, and an obstacle you drop into the Gazebo world shows up as occupied/high-cost cells.
</details>

---

## Day 6 — VIO (rtabmap `icp_odometry`, not cuVSLAM)

> **Decided 2026-07-30: `rtabmap_ros`, installed.** cuVSLAM has the same Docker/VRAM
> problem that moved Day 5 to octomap (BUILD.md §0.6). `rtabmap_odom icp_odometry` is
> apt-installable, needs no Docker, and runs on the depth cloud already being published
> — which matters because the overlay model **deletes the RGB camera** to stop the
> 180 MB/s leak, so no feature-based front-end has an image to work with.
>
> **Run it with loop closure OFF.** rtabmap is a SLAM system; loop closure corrects
> precisely the drift this day exists to measure. A drift number from a loop-closing
> SLAM system is not a VIO drift number.
>
> **Use the forest world.** ICP on a flat plane with three boxes is *degenerate* — the
> ground constrains only 3 DOF and the estimate slides freely along it. Trees constrain
> all directions. This is not cosmetic; it is the difference between a drift number and
> a slow slide into nonsense.
>
> Whatever the front-end, it publishes odometry behind the same `map -> base_link` TF
> that `px4_tf_publisher` supplies today — that node is the interface contract.
>
> Note the sim's IMU is the **body** IMU on `base_link`, not a camera-rigid one
> (see Day 4). Usable here because the camera joint is fixed and the extrinsic exact.
> On hardware that calibration is real work; sim passing does not retire it.

Launch the candidate estimator on the sim RGB/depth + IMU; confirm it publishes an odometry estimate on `/odom`. Compare its track against PX4 sim ground-truth pose over a 30–60 s flight and write down a first **drift number**. This is your baseline before the sensor ever flies.

> Wiring note: the estimator's odometry is what feeds PX4 EKF2 on hardware (via `VehicleVisualOdometry`/`VISION_POSITION_ESTIMATE`). In **sim**, PX4 already has perfect state, so don't fight it — let SITL use its own state for flight control, and run the estimator **in parallel** purely to validate the pipeline and quantify drift. The hardware swap comes later on the bench.

### Updated 2026-08-13 — this day is now a harness, not a command

Everything below is superseded by **`sweep_gate_a.py`** in the code repo, which is the
supported path: it flies a candidate across several worlds, N runs each, gates the camera
tilt/FOV *before* measuring, tears down between runs, keeps a log tail per failure, and
refuses to score a run whose mission never completed or whose `/clock` stalled because
Gazebo died. Run it with **absolute paths**.

```bash
python3 -u sweep_gate_a.py --worlds forest --runs 3                  # rgbd (closed)
python3 -u sweep_gate_a.py --worlds forest --runs 3 --imu            # + gravity alignment
python3 -u sweep_gate_a.py --worlds forest --runs 3 --estimator openvins
python3 check_vio_score.py          # the scoring core: no ROS, no sim, ~0.1 s
```

**Six things that will waste your afternoon, on top of the gotchas at the bottom of this file:**

1. **A NEW launch file needs `colcon build`.** Editing an existing one does not — the package "builds in place" for Python *nodes* only, and launch files are installed via `data_files`. That asymmetry is what makes it surprising.
2. **`lost/null: 0` used to be a lie.** rtabmap signals a lost pose with an **all-zero pose**, not a NaN. If you point a new consumer at `/odom`, that is the trap.
3. **Read `coverage`, not ATE.** On a mission that returns to its origin, an estimate that barely moves scores a *good* ATE for doing nothing.
4. **Check the build, never the parameter list.** `rtabmap --params | grep OdomOpenVINS` returns 50 parameters on a binary built `WITH_OPENVINS=false`. Use `rtabmap --version`. **A configuration surface is not a capability** — same shape as `camera_imu` appearing in `gz topic -l` with nothing publishing on it.
5. **`~/ws_px4/src/open_vins` is a patched clone, not a submodule** — 4 lines of Jazzy header migration (`image_transport`, `tf2_geometry_msgs`, `cv_bridge` all went `.h` → `.hpp` and were removed in Jazzy) plus a hard `libceres-dev` dependency. A fresh clone silently breaks it.
6. **Numbers from before 2026-08-03 are not comparable with numbers after.**

> **`ov_eval` is deliberately not built.** It is OpenVINS's own trajectory scorer, and
> scoring a candidate with a tool that shares its assumptions is exactly the class of bug
> this project keeps paying for. `eval_vio_drift.py` stays the instrument.

**✅ Success check:** the estimator publishes continuous odometry, `check_vio_score.py` is green, and the run meets the measured budget — `coverage ≥ 0.8`, ≤ ~1.5 m accumulated position error, ≤ 0.5 °/s yaw drift. **A drift number alone is not the check** — three candidates have now produced one and none is a pose you can fly on.

---

## Day 7 — Close the loop (detailed)

### Architecture — three small nodes plus nvblox

```
                 nvblox (Day 5)                          your code
 depth ─▶ ┌──────────────────────┐   /map (2D slice)  ┌───────────────┐  waypoints  ┌────────────────────┐
          │ nvblox: 3D voxels +   │ ──OccupancyGrid──▶ │ planner_node  │ ──Path────▶ │ offboard_manager   │ ──▶ PX4
 pose  ─▶ │ 2D ESDF/cost slice    │                    │ (A* on grid)  │             │ (TrajectorySetpoint)│   (Day 3 link)
          └──────────────────────┘                     └───────────────┘             └────────────────────┘
                                                              ▲                              ▲
                                                         goal pose                   /fmu/out/vehicle_local_position
```

You write **two** nodes; nvblox and the Day-3 bridge already exist:
1. `planner_node` — turns the 2D occupancy slice + a goal into a collision-free `nav_msgs/Path`.
2. `offboard_manager` — arms, holds Offboard mode, and streams the path as PX4 setpoints.

**Stay 2D for week 1.** Fly at a fixed altitude and plan on nvblox's 2D slice. Full 3D planning is a later upgrade (see end).

### Frame discipline (this is where loops die)
- PX4 local position & `TrajectorySetpoint` are **NED** (x=North, y=East, **z=Down**, so 2 m altitude = `z = -2.0`).
- ROS / nvblox / RViz are **ENU** (z=Up).
- Pick **one** planning frame (use the ENU `map` frame nvblox publishes), plan there, then convert to NED only at the moment you fill `TrajectorySetpoint`. `px4_ros_com` has helpers; if you hand-roll it: `north=y_enu, east=x_enu, down=-z_enu`.

### Node A — `planner_node` (skeleton, rclpy)
```python
# subscribes: /nvblox_node/static_map_slice  (nav_msgs/OccupancyGrid)
#             /goal_pose                      (geometry_msgs/PoseStamped, set from RViz "2D Goal Pose")
#             current pose from TF (map -> base_link)
# publishes : /planned_path                   (nav_msgs/Path, in map/ENU frame)
class PlannerNode(Node):
    def on_map(self, grid):           self.grid = grid           # store latest occupancy slice
    def on_goal(self, goal):          self.goal = goal; self.replan()
    def replan(self):
        start = self.current_xy_from_tf()
        # 1. inflate occupied cells by drone radius + margin (e.g. 0.35 m) -> safety buffer
        # 2. A* over free cells (8-connected) from start cell to goal cell
        # 3. simplify path (line-of-sight shortcut), publish as nav_msgs/Path
        ...
```
Start with stock A* on the inflated grid — don't write minimum-snap yet. **Obstacle inflation is the single most important line:** inflate by drone radius + a margin so the *center-point* path keeps the whole airframe clear. Re-run `replan()` on every new map message so newly-seen obstacles trigger replanning (that's your "mid-flight replanning").

### Node B — `offboard_manager` (skeleton, rclpy + px4_msgs)
```python
# pub: /fmu/in/offboard_control_mode (OffboardControlMode)  @ >=10 Hz  (position=True)
#      /fmu/in/trajectory_setpoint   (TrajectorySetpoint)   @ >=10 Hz
#      /fmu/in/vehicle_command       (VehicleCommand)        (arm, set Offboard)
# sub: /fmu/out/vehicle_local_position (VehicleLocalPosition)  -> current NED pos
#      /planned_path                   (nav_msgs/Path)
class OffboardManager(Node):
    def tick(self):  # 50 ms timer = 20 Hz
        self.pub_offboard_mode()                      # MUST keep streaming or PX4 exits Offboard
        wp = self.current_waypoint()                  # next path point, converted ENU->NED
        self.pub_trajectory_setpoint(x, y, z=-ALT, yaw=face_travel_dir)
        if self.reached(wp, tol=0.25): self.advance()  # 0.25 m acceptance radius
    # startup sequence: stream setpoints for ~1 s FIRST, THEN arm, THEN switch to Offboard
```
**Order matters:** publish setpoints *before* commanding Offboard, or PX4 rejects the mode switch. Keep `offboard_control_mode` flowing at ≥2 Hz the entire flight (use 10–20 Hz).

### Build & run order (terminals)
```bash
# T1: Micro-XRCE agent
MicroXRCEAgent udp4 -p 8888
# T2: PX4 SITL with depth cam
cd PX4-Autopilot && make px4_sitl gz_x500_depth
# T3: ros_gz bridges (depth + camera_info)   [Day 4]
# T4: Isaac ROS container -> cuVSLAM + nvblox  [Days 5-6]
# T5: your nodes
cd ~/ws_px4 && colcon build && source install/setup.bash
ros2 launch my_autonomy bringup.launch.py
# T6: RViz2 — add OccupancyGrid, Path, TF, PointCloud2; use "2D Goal Pose" to set the goal
```

### Test procedure
1. Empty world: confirm the drone flies start → goal in a straight line and holds. (Validates B alone.)
2. Drop **one box** in Gazebo between start and goal. Confirm nvblox marks it occupied and the planner routes around it.
3. Move the box *while flying* → confirm replanning kicks in.
4. Record per run: reached goal? closest-approach distance to the obstacle? replan latency? Note every failure mode (fell out of Offboard, clipped the obstacle, path oscillated, etc.).

### ✅ Week-1 done when
Simulated X500 autonomously flies start → goal, avoiding one inserted obstacle, with the map built live from depth and cuVSLAM producing a validated odometry track. That's the §6 Step-1 gate in BUILD.md — **only after this do you start buying hardware.**

> **Status 2026-08-13: the avoidance half is met, the odometry half is not.** The
> aircraft flies start → goal around an obstacle **on a map it built itself** (Day 7,
> 2026-07-31), and separately it flies with **GNSS fusion switched off** on an externally
> supplied pose (2026-08-01). What is still missing is the "validated odometry track":
> three candidates measured, two closed on evidence, the third produces no pose.
>
> So when this gate is called, state plainly what was validated. **"Architecture proven
> with octomap + a PX4-supplied pose, plus closed-loop flight on an external pose that is
> simulator truth"** is honest and defensible. **"nvblox and cuVSLAM validated"** is not,
> and neither is **"GPS-denied navigation."**

### Upgrade path (don't do these in week 1)
- **3D planning:** plan over the nvblox ESDF in 3D (e.g. a sampling planner, or `ego-planner` / MAVROS-style local planners) instead of a 2D slice.
- **Smooth trajectories:** replace raw waypoints with minimum-snap / polynomial trajectories for smooth, fast flight.
- **Nav2 route:** nvblox ships a Nav2 costmap plugin (`nvblox_nav2`); usable if you adapt Nav2's 2D output to multirotor setpoints — heavier, more "standard," optional.

---

## Gotchas to expect

- **Gazebo version mismatch** is the #1 time-sink: PX4 main + ROS 2 Jazzy expect **Gazebo Harmonic**. Don't mix in old `gazebo-classic`.
- **Two ROS distros on one machine** (if you ever added Humble) → only ever source one `setup.bash` per shell.
- **Isaac ROS = Docker.** Don't try to apt-install it onto a bare host; use NVIDIA's container. It needs the NVIDIA Container Toolkit.
- **Offboard mode safety timeout:** PX4 drops out of Offboard if setpoints stop arriving at a high enough rate (~>2 Hz). If the drone "falls out" of autonomy, check your publish rate first.
- **Frames/TF:** PX4 is NED, ROS is ENU. The px4_ros_com helpers handle conversions — respect them or your waypoints go sideways.
- **Gazebo *sensor* frames are x-forward**, while ROS's optical convention is z-forward. Confirmed on the depth cloud, Day 4. Different axis trap from the NED/ENU one above, and it bites the mapper rather than the controller.
- **One child frame, one parent.** Running two publishers that both parent `camera_link` produces an invalid TF tree that tf2 resolves nondeterministically — and near the origin it looks *fine*. Hit on Day 4; see DEPTH_SIM.md §4a.
- **Point the camera where the information is.** Mounted level at 3 m, half the depth frame is sky and grazing-incidence ground returns defeat octomap's plane filter — 18.4% of open ground came back *occupied*. A 20° downward tilt takes that to zero and lifts frame utilisation 52% → 78%. Tilt beats flying lower, because lowering the aircraft does not move the horizon.
- **A sensor's mounting angle lives in two places.** The model (what the sim renders) and the TF (what the stack believes). Disagree and every point is rotated about the camera, which looks like odometry drift rather than a mounting error.
- **Watch system RAM, not just VRAM.** `gz sim` with the depth airframe grew at 180 MB/s and was OOM-killed twice on 2026-07-30, the second time taking the desktop session down. The 6 GB VRAM limit is a *separate* constraint with a confusingly similar symptom. Root cause was the unused 1920×1080 RGB camera; fixed by an overlay model (`DEPTH_SIM.md` §4b).
- **An unused sensor still costs you everything.** Nothing subscribed to that RGB topic, and `always_on=0` didn't stop it either — Gazebo renders declared sensors regardless. If you don't need a sensor, delete it from the model rather than leaving it unsubscribed.
- **You have two Gazebo installs.** `.bashrc` sources ROS, which sets `GZ_CONFIG_PATH` to the ROS-vendored **8.11.0**; apt has **8.14.0**. Check `gz sim --version` before blaming (or reporting) a version-specific bug.
- **Don't redirect the `pxh>` console to an uncapped file.** Two SITL logs reached 7.8 GB, almost entirely terminal escape codes. Filter it at the source rather than discarding it — that log is the only record of *why* a run failed.
- **`use_sim_time` is all-or-nothing across every node.** `offboard_manager` was the one node in the flight stack not on sim time, and it is the one commanding the aircraft: at RTF 0.52 the sequencer gave the aircraft half the sim-time it needed per waypoint, and **three flights out of eight ran away, one by 55 m.** Watch the arming delay — under ~0.25 s between `commanded ARM + OFFBOARD` and `ARMED + OFFBOARD confirmed` is healthy; 31 s means real-time factor has collapsed and the flight is garbage. Also sanity-check truth path length: a 5 m square is ~29–35 m, and 91 m means the aircraft ran away.
- **A diagnostic must not depend on the thing it diagnoses.** The most reused idea in this project. A heartbeat on a node timer is silent in exactly the case (`use_sim_time` with no `/clock`) it exists to report — put it on a `STEADY_TIME` clock.
- **QoS is two independent policies, and DDS reports only the first mismatch.** Fixing one and re-testing looks exactly like the fix failing. Never subscribe with default QoS to anything you did not publish, and remember `ros2 topic hz` creates its *own* compatible subscriber, so it shows a healthy topic while your callback never fires once.
- **NVIDIA kernel modules are prebuilt per kernel version and a kernel upgrade does not carry them over.** `nvidia-smi` failed, Gazebo silently fell back to the Intel iGPU, and **segfaulted mid-sweep** — while RTF read ~1.0 and every arming delay looked healthy. Install the `-generic-hwe-24.04` metapackage so it keeps tracking. And note **"the driver is loaded" and "Gazebo is using it" are different claims**: with `prime-select on-demand` you need the render offload, and only a `gz` process appearing in `nvidia-smi --query-compute-apps` answers the second.
- **This laptop is thermally constrained.** Measured mid-sweep: CPU package 100 °C, GPU 92 °C against a 95 °C slowdown threshold. Tear down between runs and never leave SITL running — a forgotten Gazebo at 245% CPU once made a *driver* fault look like a *cooling* fault.
- **`pgrep`/`pkill` without `-f` matches process names**, so `pgrep -a foo.py` finds nothing while `foo.py` is running and looks exactly like a crash. But `-f` then matches *your own shell's* command line — `pkill -f offboard_manager` killed the shell it was typed into. Prefer `pkill -x`, or filter out `$$`/`$PPID`.
- **Buffered stdout looks like a hang.** A redirected sweep prints nothing for 40 minutes and then everything at once. The results JSON is the reliable progress signal; use `python3 -u` for the console.

---

## Official docs to cross-check against
- PX4 ROS 2 User Guide: https://docs.px4.io/main/en/ros2/user_guide
- PX4 uXRCE-DDS bridge: https://docs.px4.io/main/en/middleware/uxrce_dds
- PX4 Gazebo SITL: https://docs.px4.io/main/en/sim_gazebo_gz/
- ROS 2 Jazzy install: https://docs.ros.org/en/jazzy/Installation.html
- Isaac ROS Getting Started (Docker): https://nvidia-isaac-ros.github.io/getting_started/index.html
- Isaac ROS Visual SLAM (cuVSLAM): https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_visual_slam/index.html
- Isaac ROS Nvblox: https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/index.html
