# AI-Powered-Autonomous-Mobile-Robot

# AI-Powered Autonomous Mobile Robot — development roadmap

## Part I. Repository assessment

**Your repository contains one ROS 2 package and a partial visual model. The first milestone is to make that existing model installable, correct its ground clearance, and complete its kinematic tree.** You can do all three before introducing Gazebo. Doing them in this order will let you distinguish a modeling error from a physics, controller, or navigation error later.

### 1. What is actually in the repository

| Status | Finding | Evidence |
|---|---|---|
| **Already implemented** | The repository root has a colcon workspace layout. The only package under `src` is `robot_description`. It uses `ament_cmake`, package format 3, version `0.0.0`, and an MIT license declaration. | [Package manifest](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/src/robot_description/package.xml) |
| **Already implemented** | The Xacro robot is named `autonomous_mobile_robot`. It defines a blue chassis box, three chassis dimension properties, a black left wheel, and one continuous joint. | [Robot model](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/src/robot_description/urdf/robot.urdf.xacro:2) |
| **Already implemented** | The chassis measures **0.40 × 0.30 × 0.15 m**. The left wheel has radius **0.05 m**, width **0.03 m**, and center **(0, 0.165, 0) m** relative to `chassis`. The joint axis is **+Y**. The wheel visual has a roll of \(\pi/2\), placing its cylinder axis along the axle line. | [Chassis dimensions](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/src/robot_description/urdf/robot.urdf.xacro:6) · [Wheel and joint](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/src/robot_description/urdf/robot.urdf.xacro:22) |
| **Partially implemented** | The package builds its ament metadata. A historical build has return code `0` and empty stderr. Its installed share directory contains package metadata and environment hooks. **It contains no `urdf` directory.** | [CMake configuration](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/src/robot_description/CMakeLists.txt:26) · [Historical build result](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/build/robot_description/colcon_build.rc) |
| **Partially implemented** | The manifest declares `ament_cmake`, `ament_lint_auto`, and `ament_lint_common`. It has no runtime dependency on `xacro`. CMake enables lint discovery and skips the copyright and cpplint checks. There is no behavioral test of the model. | [Manifest](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/src/robot_description/package.xml) · [Test configuration](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/src/robot_description/CMakeLists.txt:14) |
| **Not implemented** | No right wheel, caster, `base_footprint`, `base_link`, sensor mounts, collisions, masses, inertia tensors, reusable wheel macro, or actuator interfaces. The only links are `chassis` and `left-wheel-link`. The only joint is `left-wheel-joint`. | [Complete Xacro source](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/src/robot_description/urdf/robot.urdf.xacro) |
| **Not implemented** | No project launch files, RViz configuration, Gazebo worlds, bridge YAML, controller YAML, ROS nodes, navigation configuration, maps, perception code, missions, or simulation tests. The package’s C++ source and include directories are empty. The other four proposed ROS packages do not yet exist. | [Workspace source directory](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/src) |
| **Already implemented** | The repository and package contain license files. The README contains the project title. The root ignore file covers `/build/`, `/install/`, and `/log/`. Local branches include `main` and `feature/robot-description`. The checked-out feature branch and its locally recorded `origin` tracking ref point to `e094b62`. | [README](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/README.md) · [Ignore rules](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/.gitignore) |
| **Requires verification** | Fresh build, actual Xacro expansion, URDF parsing, ROS discovery, RViz rendering, Gazebo rendering and physics, and live remote Git state. The untracked `.vscode/` directory has a setting pointing CMake at this package through an absolute path under your home directory. | [VS Code setting](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/.vscode/settings.json) |

**Two engineering findings change the immediate plan.**

1. **The chassis would hit the floor before the wheel.** Both centers currently have \(z=0\). The chassis bottom is at \(-0.15/2=-0.075\) m. The wheel bottom is at \(-0.05\) m. The chassis extends **25 mm below** the wheel’s contact plane. Raising the entire robot at spawn only delays contact. You need to change the chassis-to-axle geometry.
2. **A successful package build currently says nothing about whether an installed launch can find the robot.** CMake ends with `ament_package()` and has no resource installation rule. It also has no build target that expands the Xacro. Test both the source model and the installed model.

The existing lateral wheel placement is a good starting point. Half the chassis width plus half the wheel width is \(0.15+0.015=0.165\) m. Mirroring it gives a nominal **0.330 m wheel-center separation** and a **0.360 m total outside width**. Those are different measurements. The controller needs the former. Collision checking must account for the latter.

**Inspection scope.** I used shell commands for the initial read-only file inventory and Git status before reading the attachment’s instruction against command execution. I then used direct file reads and official documentation. I did not build, launch, test, install, edit project files, or make a commit. All commands below are for your later use.

### 2. Environment and compatibility as of 22 November 2026

There are three separate questions here: whether upstream supports your platform, whether packages are installed on this machine, and whether they work together at runtime.

| Component | Upstream compatibility and local evidence | What still needs a runtime check |
|---|---|---|
| **Ubuntu and ROS 2** | The host reports **Ubuntu 26.04.1 LTS**, with `amd64` system packages. `/opt/ros/lyrical` exists. The package database lists `ros-lyrical-desktop`, ROS CLI, C++ and Python demos, and ROS development tools. Ubuntu 26.04 `amd64` and `arm64` are both Tier 1 Lyrical targets. [Lyrical platform matrix](https://github.com/ros2/ros2_documentation/blob/lyrical/source/Releases/lyrical/supported-platforms.rst) | Source the intended underlay and exchange a message between two terminals. |
| **Build tools** | Installed records include Python 3.14.3, CMake 4.2.3, GCC/G++ 15-series packages, `build-essential`, colcon extensions, rosdep, pytest, and VS Code. Your package’s CMake minimum is 3.20. Lyrical specifies C++20, C17, and Python 3.12–3.14. [Lyrical language requirements](https://github.com/ros2/ros2_documentation/blob/lyrical/source/Releases/lyrical/supported-platforms.rst) | Check the executable your shell selects. A virtual environment or a previously sourced overlay can select a different Python or package. |
| **Model and visualization** | `xacro`, `check_urdf`, RViz2, `robot_state_publisher`, both joint-state publisher variants, `tf2_ros`, and `tf2_tools` are present in the Lyrical installation. `check_urdf` is under `/opt/ros/lyrical/bin`. [Robot State Publisher](https://github.com/ros/robot_state_publisher) | Expand and parse the current model, then verify RViz and the graphics driver. |
| **Gazebo** | The supported pairing is **Gazebo Jetty**, Gazebo Sim major **10**. Local packages include `ros_gz`, `ros_gz_sim`, `ros_gz_bridge`, `ros_gz_image`, simulator demos, and Gazebo vendor libraries. The installed simulator launch file defaults to major 10. [Gazebo pairing](https://gazebosim.org/docs/jetty/ros_installation/) · [Installed launch file](/opt/ros/lyrical/share/ros_gz_sim/launch/gz_sim.launch.py) | Load a world, advance simulated time, render a GPU sensor, and verify its ROS bridge. Each exercises a different subsystem. |
| **Control** | `gz_ros2_control`, `controller_manager`, `joint_state_broadcaster`, and `diff_drive_controller` have Lyrical releases. I did not find these packages in the inspected Lyrical installation. [Gazebo control adapter](https://control.ros.org/lyrical/doc/gz_ros2_control/doc/index.html) · [Differential-drive controller](https://control.ros.org/lyrical/doc/ros2_controllers/diff_drive_controller/doc/userdoc.html) | Check apt candidates before Phase 4, then discover the hardware interfaces and activate the controllers. |
| **Estimation and navigation** | `robot_localization`, `slam_toolbox`, `navigation2`, `nav2_bringup`, and Nav2’s AMCL and map-server packages have Lyrical releases. I did not find the principal packages in the inspected installation. [Lyrical build farm](https://repo.ros2.org/status_page/ros_lyrical_default.html) | Check candidates from your configured repositories. After installation, inspect the installed launch arguments and parameter files. |
| **Sensors and vision** | Gazebo sensor vendor packages, ROS bridge examples, `cv_bridge`, `image_transport`, `vision_msgs`, `rqt_image_view`, and Python OpenCV are present. The installed bridge demos include camera, GPU LiDAR, IMU, and RGB-D examples. [Installed demos](/opt/ros/lyrical/share/ros_gz_sim_demos/launch) · [Jetty sensors](https://gazebosim.org/docs/jetty/sensors/) | Verify image encoding, optical frame, scan orientation, clock, rates, and graphics performance on this host. |
| **Apple Silicon later** | Ubuntu 26.04 **inside an ARM64 Linux environment** is a Tier 1 target. Native macOS ARM64 has no Lyrical support tier in the published matrix. The build farm lists ARM64 binaries for core robotics packages. The official ROS container manifest includes multiarchitecture Lyrical images. [Platform matrix](https://github.com/ros2/ros2_documentation/blob/lyrical/source/Releases/lyrical/supported-platforms.rst) · [ARM64 builds](https://repo.ros2.org/status_page/ros_lyrical_armv8.html) · [Official container manifest](https://github.com/docker-library/official-images/blob/master/library/ros) | Check the guest’s graphics acceleration, DDS discovery, and any ML runtime separately. A successful ARM64 build does not establish usable camera frame rates. |

**Use native Ubuntu for the first end-to-end robot.** You already have most of the environment, and it gives the shortest diagnostic path for RViz and Gazebo. Docker becomes valuable for clean builds and headless tests. An ARM64 Ubuntu VM gives you a complete Linux desktop on a Mac, with graphics performance dependent on the hypervisor. Docker Desktop on macOS also crosses a Linux VM boundary. Test discovery between processes inside one environment before introducing host-to-container networking. Headless Gazebo still needs a functioning rendering backend for an RGB camera or GPU LiDAR.

---

## Part II. Proposed architecture

### 3. Keep the five proposed packages

The five packages are appropriate. Create each when its first deliverable needs it. The paths in this table are **proposed**, except for the existing `robot_description` package and its current files.

| Package | Responsibility and first files | Dependency direction |
|---|---|---|
| `robot_description` | Physical dimensions, link and joint names, masses, collision shapes, sensor mounts. Keep `urdf/robot.urdf.xacro` as the public entry point. Add `urdf/wheel.xacro`, `launch/display.launch.py`, and `rviz/display.rviz` when used. | Independent of Gazebo, navigation, and perception. The display launch may depend on Xacro, RViz, state publishers, and ROS launch. |
| `robot_gazebo` | `worlds/lab.sdf`, `urdf/sim.urdf.xacro`, `urdf/control.xacro`, `urdf/sensors.xacro`, `config/bridge.yaml`, `config/controllers.yaml`, `launch/sim.launch.py`. Owns simulated physics, actuation, sensor generation, and bridging. | Its simulation Xacro includes the physical description and adds simulator-specific elements. This avoids making the description package depend back on Gazebo. |
| `robot_navigation` | `config/ekf.yaml`, `config/slam.yaml`, `config/nav2.yaml`, `maps/`, and launches for estimation, mapping, localization, and navigation. An educational encoder integrator can live in `tools/` here. | Consumes standard odometry, IMU, scan, clock, and TF contracts. It can later consume the same contracts from physical drivers. |
| `robot_perception` | Python package with camera conversion, classical baseline, learned detector adapter, parameter files, evaluation fixtures, and detection tests. | Consumes camera messages. Publishes detections. It does not need to know how wheels or Nav2 work. |
| `robot_bringup` | System-level launch composition and the first small mission client. Examples include `launch/mission.launch.py`, `config/inspection_sites.yaml`, and `robot_bringup/inspection_mission.py`. | Depends on the components it starts. Lower-level packages must remain launchable independently. |

A separate control package becomes worthwhile when the same controller configuration is shared with physical hardware. A separate mission package becomes worthwhile when several missions share infrastructure. Neither split buys much for the first simulated robot.

### 4. Mechanical and frame design

Use REP-103 body axes throughout: **+X forward, +Y left, +Z up**, metres, seconds, radians, kilograms. Positive yaw is counterclockwise when viewed from above. Use the following as a **worked design proposal**, with the dimensions already in the repository preserved where indicated. [REP-103](https://github.com/ros-infrastructure/rep/blob/master/rep-0103.rst)

| Item | Proposed convention | Consequence you can test |
|---|---|---|
| `base_footprint` | Ground-plane frame directly below the drive-axle midpoint. Use a planar, level-floor robot for the MVP. | Navigation can express a footprint in a plane at \(z=0\). |
| `base_link` | Body-fixed frame at the drive-axle midpoint, **0.05 m above** `base_footprint`. A fixed joint joins them. Put chassis geometry and inertia on `base_link`. | Both wheel joint centers have \(x=0,z=0\) in `base_link`. No extra physical chassis link is necessary. |
| Chassis center | Example \((-0.035,0,+0.050)\) m in `base_link`. Apply the same offset to the box visual, collision, and center of mass for a uniform chassis. | Bottom clearance becomes \(0.050+0.050-0.075=0.025\) m. The rearward offset puts load toward a rear caster. |
| Drive wheels | Radius 0.05 m and width 0.03 m from the existing wheel. Centers at \(y=\pm0.165\) m. Both continuous joint axes point +Y. | Equal positive wheel speeds drive forward. Right faster than left gives positive yaw. |
| Passive support | Start with a **low-friction spherical caster approximation**, fixed to the rear of the body. Example radius 0.025 m and center \((-0.16,0,-0.025)\) m in `base_link`. | Its lowest point lies in the wheel contact plane. The projection of the combined center of mass should lie inside the triangle formed by the two wheels and rear support. |
| Sensor frames | `laser_link` above the chassis with clear horizontal visibility, `imu_link` body-aligned, `camera_link` forward-facing, and `camera_optical_frame` with optical axes. Record measured mount offsets in Xacro. | A wall in front appears at scan angle zero and in front of the camera. Positive body yaw gives positive IMU angular velocity about Z. |

The fixed footprint-to-body transform is a **planar approximation**. A robot pitching on ramps would require a different treatment of the ground projection. The sphere is also a deliberate simplification. A later steering caster needs a swivel joint, wheel joint, trail, and additional contact tuning.

### 5. Final TF tree and transform ownership

```text
map
└── odom                         SLAM Toolbox while mapping OR AMCL while localizing
    └── base_footprint           wheel controller initially; planar EKF after fusion
        └── base_link            fixed URDF joint
            ├── left_wheel_link  continuous joint, driven by measured joint position
            ├── right_wheel_link continuous joint, driven by measured joint position
            ├── caster_link      fixed for the spherical approximation
            ├── laser_link       fixed mount
            ├── imu_link         fixed mount
            └── camera_link      fixed mount
                └── camera_optical_frame   fixed optical convention
```

`robot_state_publisher` reads the URDF and computes the transforms below `base_footprint`. It publishes fixed joints on `/tf_static` and movable joints on `/tf` when it receives positions in `/joint_states`. The GUI joint-state publisher supplies those positions only in the display exercise. The controller’s joint-state broadcaster supplies them in simulation. A fixed camera mounting transform and a fixed camera-to-optical transform have different purposes. For a forward-facing body-aligned camera, the latter can use roll/pitch/yaw \((-\pi/2,0,-\pi/2)\). [Robot State Publisher](https://github.com/ros/robot_state_publisher) · [Optical frame convention](https://github.com/ros-infrastructure/rep/blob/master/rep-0103.rst)

`odom` gives locally continuous motion and gradually drifts. `map` gives a globally useful reference which may correct abruptly after a loop closure or localization update. The global estimator can compute

\[
{}^{map}T_{odom} =
{}^{map}T_{base}\left({}^{odom}T_{base}\right)^{-1},
\]

where \({}^{a}T_b\) transforms coordinates from frame \(b\) into frame \(a\), and `base` denotes the same robot frame in both estimates. This lets a map correction coexist with smooth local motion. Assign **one publisher per TF edge in each operating mode**. [REP-105](https://github.com/ros-infrastructure/rep/blob/master/rep-0105.rst)

### 6. ROS communication contracts

Names below are proposed project conventions unless they are upstream names. Start with one un-namespaced robot. A topic carries a stream, a service performs a short request/reply, and an action supports a longer operation with feedback, cancellation, and a terminal result.

| Producer → consumer | Interface | Purpose and checks |
|---|---|---|
| Description launch → `robot_state_publisher` → RViz, spawner, control integration | `robot_description` parameter containing expanded URDF. Default publisher also offers `/robot_description`, `std_msgs/msg/String`, transient-local. | Use the same expanded simulation model for spawning and the simulation state publisher. A late subscriber must request compatible durability to retrieve a previously published description. |
| GUI publisher **or** `joint_state_broadcaster` → state publisher | `/joint_states`, `sensor_msgs/msg/JointState` | Match positions to joint **names**. Array ordering is an implementation detail. The state publisher computes wheel TF. |
| Gazebo → `ros_gz_bridge` → simulated ROS nodes | `/clock`, `rosgraph_msgs/msg/Clock` | One clock publisher. Set `use_sim_time:=true` for timestamping, TF consumers, controllers as applicable, RViz, and perception in simulation. |
| Gazebo LiDAR → bridge → SLAM, AMCL, costmaps, collision monitor | `/scan`, `sensor_msgs/msg/LaserScan` | Header frame `laser_link`, time at acquisition, ranges in metres, angles in radians. Consumers need sensor-compatible QoS and TF at that timestamp. |
| Gazebo IMU → bridge → EKF | `/imu/data`, `sensor_msgs/msg/Imu` | Header frame `imu_link`. Initially fuse yaw rate. Verify units, direction, covariance, and which measurements the simulated device supplies. |
| Gazebo camera → bridge → perception and RViz | `/camera/image_raw`, `sensor_msgs/msg/Image`, and `/camera/camera_info`, `sensor_msgs/msg/CameraInfo` | Image encoding, calibration, timestamp, dimensions, and optical frame must agree. |
| Teleop in manual mode, or Nav2’s final velocity stage in autonomous mode → `base_controller` | `/base_controller/cmd_vel`, `geometry_msgs/msg/TwistStamped` | Lyrical’s differential-drive controller reads forward `linear.x` and yaw `angular.z`. Use fresh stamps and a command timeout. |
| `base_controller` → EKF or odometry evaluator | `/base_controller/odom`, `nav_msgs/msg/Odometry` | Raw wheel-feedback estimate. Pose is expressed in `header.frame_id`; twist in `child_frame_id`. |
| EKF → Nav2 and TF | `/odometry/filtered`, `nav_msgs/msg/Odometry`; `odom → base_footprint` | Once enabled, set `enable_odom_tf: false` on the wheel controller. Point every relevant Nav2 odometry consumer at the chosen topic. |
| SLAM Toolbox or map server → RViz, AMCL, global costmap as appropriate | `/map`, `nav_msgs/msg/OccupancyGrid` | SLAM produces the map during mapping. The map server serves a saved grid during localization. Check resolution, origin, dimensions, and durability. |
| RViz → AMCL → operator and TF | `/initialpose` and `/amcl_pose`, `geometry_msgs/msg/PoseWithCovarianceStamped` | The initial pose seeds a distribution in `map`. AMCL estimates pose from odometric motion and scan agreement. |
| Mission or RViz → Nav2 | `/navigate_to_pose`, `nav2_msgs/action/NavigateToPose` | Goal in `map`, feedback during motion, terminal success/abort/cancel. Sending a goal and reaching it are separate events. |
| BT navigator → planner/controller servers | `ComputePathToPose` and `FollowPath` actions from `nav2_msgs`; the planned route is a `nav_msgs/msg/Path` | The planner finds a route. The controller repeatedly chooses feasible motion as observations change. |
| Nav2 controller/behavior servers → velocity smoother → collision monitor → base | `cmd_vel_nav` → `cmd_vel_smoothed` → `/base_controller/cmd_vel` | Wire the last output explicitly. Inspect endpoint types and publishers at each hop. Run manual control and navigation in mutually exclusive modes. |
| Perception → mission | `/perception/detections`, `vision_msgs/msg/Detection2DArray` | Each observation retains the source image stamp and frame, a box, and class hypotheses. A 2D box alone gives no metric range. |
| Mission → operator | Proposed `/mission/status`, `std_msgs/msg/String`, plus a run result saved by the mission | Report the site, recognized class, observation time, action result, and termination reason. A custom message can wait until there is a second programmatic consumer. |

The controller manager’s services load/configure/switch controllers. Lifecycle services configure and activate SLAM/Nav2 components. The map saver takes a grid snapshot. These are useful service boundaries. Streaming wheel commands through services or implementing navigation as a blocking service would make timing and cancellation harder. [Lyrical drive interfaces](https://control.ros.org/lyrical/doc/ros2_controllers/diff_drive_controller/doc/userdoc.html) · [Nav2 launch wiring](https://github.com/ros-navigation/navigation2/blob/1.5.2/nav2_bringup/launch/navigation_launch.py) · [Vision interfaces](https://github.com/ros-perception/vision_msgs/tree/ros2/vision_msgs/msg)

---

## Part III. Level 1 — high-level roadmap

I would swap the original **sensor simulation** and **fused state estimation** phases. Raw wheel odometry already appears when the drive controller works. Simulating and validating the IMU before fusing it gives you two independently testable inputs. I would also start testing in Phase 0 and reserve Phase 13 for repeated trials and optimization.

Times are **student working hours**, with learning and implementation/debugging counted separately. The completion tests below are proposed engineering targets for a small indoor simulation. Measure a baseline before tightening them.

| Phase | Depends on | Outcome | Learn | Implement and debug | Difficulty |
|---|---|---|---:|---:|---|
| 0. Environment and repository | Existing workspace | Reproducible baseline, package discovery, working ROS communication | 2–4 h | 2–4 h | Low |
| 1. Robot description | 0 | Installable, mechanically coherent, physics-ready model | 5–8 h | 6–10 h | Medium |
| 2. RViz and TF | 1 | Inspectable frame tree and independently movable wheels | 3–5 h | 3–6 h | Medium |
| 3. Gazebo | 1–2 | Stable spawn, ground contacts, one simulation clock | 4–7 h | 5–9 h | Medium |
| 4. Drive and teleoperation | 3 | Measured wheel actuation, raw odometry, command timeout | 5–8 h | 6–10 h | Medium–high |
| 5. Sensor simulation | 2–4 | Validated LiDAR and IMU; RGB camera available for full project | 4–7 h | 6–11 h | Medium |
| 6. Odometry and fusion | 4–5 | Independently checked wheel integration and planar EKF | 6–10 h | 7–12 h | High |
| 7. SLAM | 4–5 and working odom TF | Reusable occupancy grid of a measured environment | 4–7 h | 5–9 h | Medium–high |
| 8. Known-map localization | 7 | Repeatable AMCL initialization and pose tracking | 4–7 h | 4–8 h | Medium |
| 9. Autonomous navigation | 4, 5, 8 | Goals, obstacle response, cancellation and repeatable routes | 7–12 h | 10–18 h | High |
| 10. Classical vision | RGB part of 5 | Measured image-processing baseline and detection contract | 4–7 h | 6–10 h | Medium |
| 11. Learned perception | 10 | Evaluated learned detector using the same contract | 6–12 h | 10–20 h | High |
| 12. Autonomous mission | 9–11 | Inspect sites, recognize a target, report outcome, handle failure | 4–7 h | 7–13 h | High |
| 13. Testing and optimization | Tests accumulate from 0; full mission for final evaluation | Reproducible metrics, failure cases, and performance budget | 4–7 h | 8–15 h | High |
| 14. Documentation and demo | 9 for MVP demo, 12–13 for full demo | Independently reproducible project and portfolio demonstration | 2–4 h | 5–9 h | Medium |

**MVP.** Two driven wheels, passive support, stable Gazebo physics, teleop, feedback odometry, a real simulated LiDAR stream, mapping, saved-map localization, and Nav2 reaching a goal around an obstacle. Wheel-only odometry can support the first MVP if it passes the motion tests. The EKF is a valuable learning milestone and the intended full configuration. RGB, depth, and ML are outside the MVP’s dependency chain.

**Full project.** Add the IMU/EKF comparison, RGB perception, a measured classical baseline, a learned recognition capability, an inspection mission, failure handling, and reproducible evaluation. At ten hours per week, budget approximately **15–28 weeks** for the full plan, with integration uncertainty concentrated in physics, navigation, and model evaluation.

---

## Part IV. Level 2 — detailed milestone plan

### How to use the commands

The examples assume Bash. Set these variables in each new development terminal. The letters are shell conveniences for the exact future directories.

```bash
export WS="$HOME/Projects/AI-Powered-Autonomous-Mobile-Robot"
export D="$WS/src/robot_description"
export G="$WS/src/robot_gazebo"
export N="$WS/src/robot_navigation"
export P="$WS/src/robot_perception"
export B="$WS/src/robot_bringup"
export MODEL="$D/urdf/robot.urdf.xacro"
cd "$WS"
source /opt/ros/lyrical/setup.bash
```

For **builds**, start with a fresh terminal sourced to the underlay. For **running** your packages, additionally run `source "$WS/install/local_setup.bash"` after a successful build. This adds your workspace’s installed packages to discovery.

The following conventions apply to every task:

- **Editing a named file** means opening it with `code <path>` and making the specified change yourself. `mkdir -p <directory>` creates a required parent directory. Project launch names, Python entry points, tests, and configuration filenames introduced below are **deliverables you will author**. They become callable after installation.
- **Building an affected package** means `colcon build --symlink-install --packages-up-to PACKAGE` from `$WS`, substituting its exact package name. `--packages-up-to` includes workspace dependencies. `--symlink-install` makes many resource edits visible without a copy. Rebuild when adding a resource, changing installation rules, or adding a Python entry point.
- **Validating the model** means `xacro "$MODEL" -o /tmp/amr_model.urdf` followed by `check_urdf /tmp/amr_model.urdf`. The former expands properties and macros. The latter parses the generated robot tree. Both must succeed. A parser cannot detect an upside-down camera, incorrect mass, or insufficient ground clearance.
- **`ros2 run PACKAGE EXECUTABLE`** starts one installed executable. **`ros2 launch PACKAGE FILE`** starts a described group. `--ros-args -p name:=value` sets a ROS parameter; `-r old:=new` remaps an interface. A launch argument such as `use_sim_time:=true` takes effect only when that launch declares or forwards it.
- **`ros2 topic info -v`** inspects endpoint names, types, and QoS. **`echo --once`** samples one message. **`hz`** measures rate observed by that subscriber. With a slow simulator, compare message-stamp intervals as well as wall-clock receipt rate. Use separate terminals for persistent processes and stop the previous mode before starting a competing one.

### Phase 0 — establish the baseline

**Objective and concepts.** Learn how the underlay, overlay, ament index, and DDS graph fit together. Package discovery, successful compilation, and a running node are three separate observations. Use the installed ROS demos to isolate middleware problems from project problems. [Lyrical installation guide](https://docs.ros.org/en/lyrical/Get-Started/Installation/Ubuntu-Install-Debs.html) · [Lyrical tutorials](https://github.com/ros2/ros2_documentation/tree/lyrical/source/Tutorials)

| Task | Work and verification |
|---|---|
| **0.1.1 — Identify the platform** | Workspace-level, no file edit. Run `cat /etc/os-release`, `dpkg --print-architecture`, `printenv ROS_DISTRO`, and `command -v ros2 xacro check_urdf colcon`. Expect Resolute, `amd64`, `lyrical`, and resolvable tools. An empty distro or missing `check_urdf` points first to the sourced environment. |
| **0.1.2 — Reproduce the package baseline** | Run `colcon list` and `colcon build --symlink-install --packages-select robot_description`. `--packages-select` isolates the existing package. In a runtime terminal, use `ros2 pkg prefix --share robot_description`. Expect one workspace source package and a share path under this workspace’s `install`. A path in another workspace means overlay contamination. |
| **0.1.3 — Verify communication** | Run `ros2 run demo_nodes_cpp talker` and `ros2 run demo_nodes_py listener` in separate sourced terminals. In a third use `ros2 node list` and `ros2 topic info /chatter -v`. Expect a string publisher, subscriber, and increasing message counter. Compare `ROS_DOMAIN_ID`, `RMW_IMPLEMENTATION`, and terminal sourcing if discovery fails. |
| **0.2.1 — Record the development contract** | Update `README.md` with OS, distro, workspace root, sourcing order, build command, and baseline limitations. Review `.vscode/settings.json`; use `${workspaceFolder}/src/robot_description` if you want to share the current single-package setting. Use colcon from the root for multi-package builds. `git status --short` should show only intended source/editor/documentation changes. |
| **0.2.2 — Establish dependency checks** | Run `rosdep check --from-paths src --ignore-src --rosdistro lyrical`. It resolves external manifest keys for this distro while ignoring workspace packages. If the rosdep cache is uninitialized, follow the ROS installation guide’s one-time initialization and `rosdep update`. A satisfied manifest currently covers only the dependencies that manifest declares. |

**Completion:** a fresh terminal finds your package, one build completes, and the listener receives five successive messages. **Git suggestion:** `feature/workspace-baseline`, commit `Document and verify the Lyrical workspace baseline`. **Learn before moving on:** identify which paths come from `/opt/ros` and which come from `install`. **Optional:** repeat the CLI demo in an ARM64 container after the native baseline works.

### Phase 1 — finish the robot description

**Objective and concepts.** A link has its own coordinate frame. A joint places its child relative to its parent. A visual describes appearance, a collision describes contact geometry, and an inertial element describes mass distribution. Xacro generates the URDF consumed by the rest of ROS. [ROS URDF examples](https://github.com/ros/urdf_tutorial/tree/ros2) · [Xacro](https://github.com/ros/xacro)

For a uniform box of mass \(m\), length \(L\) along X, width \(W\) along Y, and height \(H\) along Z, the principal moments about its center are

\[
I_{xx}=m(W^2+H^2)/12,\quad I_{yy}=m(L^2+H^2)/12,\quad I_{zz}=m(L^2+W^2)/12.
\]

For a solid wheel cylinder of radius \(r\), axial width \(w\), and axle along Y, \(I_{yy}=mr^2/2\) and \(I_{xx}=I_{zz}=m(3r^2+w^2)/12\). For a solid sphere all three moments are \(2mr^2/5\). Units are kg·m². Express those components in the **inertial frame** you actually declare.

| Task | Work and verification |
|---|---|
| **1.1.1 — Preserve a baseline** | Expand the original model to `/tmp/amr_before.urdf` and parse it. Expect `chassis` as root and `left-wheel-link` as its only child. If expansion fails, fix that before editing geometry. This separates an original defect from a later regression. |
| **1.1.2 — Install the description** | In `$D/CMakeLists.txt`, add `install(DIRECTORY urdf DESTINATION share/${PROJECT_NAME})` before `ament_package()`. Add `<exec_depend>xacro</exec_depend>` in `$D/package.xml`. Build `robot_description`. Run `xacro "$(ros2 pkg prefix --share robot_description)/urdf/robot.urdf.xacro" -o /tmp/amr_installed.urdf` and parse it. An absent file points to the install rule or wrong overlay. |
| **1.1.3 — Define the base and clearance** | In `$MODEL`, introduce `base_footprint`, `base_link`, and their fixed joint. Transfer the chassis visual to `base_link` and apply the offset from Part II. Parameterize wheel radius, width, track, and chassis center. Validate the model. Calculate clearance on paper and compare it with the generated origins. |
| **1.2.1 — Make and mirror the wheels** | Create `$D/urdf/wheel.xacro` with a macro taking a side/name and signed lateral offset. Generate `left_wheel_joint` and `right_wheel_joint` with corresponding `_link` children. Apply the cylinder rotation to geometry, leave both joint axes at `0 1 0`, and validate. Expect two continuous joints at opposite Y coordinates. Reversing one joint axis will cause trouble in Phase 4. |
| **1.2.2 — Add passive support and collisions** | Add the fixed caster approximation. Put primitive collisions on chassis, wheels, and support with matching physical origins. Validate and run `gz sdf -p /tmp/amr_model.urdf` to inspect Gazebo’s conversion. Check the three support points, wheel clearance, and overall footprint. Fixed-link lumping can merge physical elements in SDF while the ROS TF frames still exist. |
| **1.2.3 — Give physical bodies mass and inertia** | Use explicit educational mass assumptions, for example 4 kg for the chassis, 0.20 kg per drive wheel, and 0.08 kg for the support. Document them in `README.md`. Place each inertial origin at its center of mass. Leave purely conceptual frames massless. Validate and inspect the converted SDF for both drive joints, positive masses, and plausible inertias. A uniform example chassis gives \(I_{xx}=0.0375\) kg·m². |

**Completion:** one root, two continuous wheel joints, three intended ground supports, positive 25 mm chassis clearance for the worked dimensions, and consistent installed/source expansions. **Git:** continue `feature/robot-description`, commit `Complete the differential-drive geometry and inertial model`. Document a dimensioned side and top view. **Optional:** replace the spherical support with a true swivel caster after the simpler model drives reliably.

### Phase 2 — visualize and verify TF

**Objective and concepts.** A homogeneous transform is \(T=\begin{bmatrix}R&t\\0&1\end{bmatrix}\), where \(R\) is a 3D rotation and \(t\) a translation. For a child point \(p_c\), \(p_p=Rp_c+t\) gives its parent-frame coordinates. A wheel angle changes its orientation. The fixed chassis-to-axle distance stays constant. RViz lets you test this without contact forces. [Robot State Publisher](https://github.com/ros/robot_state_publisher) · [TF conventions](https://github.com/ros-infrastructure/rep/blob/master/rep-0105.rst)

| Task | Work and verification |
|---|---|
| **2.1.1 — Launch the state publisher** | Create `$D/launch/display.launch.py`. Resolve the Xacro from the installed package share, expand it, pass its XML string to `robot_state_publisher` as `robot_description`, and use wall time for display. Add the launch directory to CMake installation and declare the dependencies your launch imports/starts. Build. Run `ros2 launch robot_description display.launch.py` and `ros2 param get /robot_state_publisher robot_description`. Expect expanded XML. Literal `${…}` expressions mean the launch passed unexpanded Xacro. |
| **2.1.2 — Supply joint positions** | Add `joint_state_publisher_gui` to the display launch. Inspect `ros2 topic echo /joint_states --once`. Move one slider and observe that joint’s position changing. A wheel missing from the GUI suggests a fixed joint, a stale installed description, or a name mismatch. |
| **2.2.1 — Configure RViz** | Add RViz2 to the launch. On the first run choose fixed frame `base_footprint`, add `RobotModel` using `/robot_description`, and add `TF` and `Grid`. Save `$D/rviz/display.rviz`, install `rviz/`, and have later launches load the file. Both wheels and support should appear at their predicted positions. The TF axes make wheel rotation visible even on a featureless cylinder. |
| **2.2.2 — Check individual transforms** | Run `ros2 run tf2_ros tf2_echo base_footprint base_link` and `ros2 run tf2_ros tf2_echo base_link left_wheel_link`. Expect translations `(0,0,0.05)` and `(0,0.165,0)` for the proposed geometry. Move the slider and inspect the second rotation. Run `(cd /tmp && ros2 run tf2_tools view_frames)` for a tree diagram. If fixed edges work and wheel edges fail, inspect `/joint_states` before editing the URDF. |

**Completion:** RViz opens from the installed launch, the model has no missing-transform error, and moving either wheel changes only its intended joint. **Git:** `feature/description-display`, commit `Add repeatable RViz and TF verification`. Document one expected transform and one screenshot. **Optional:** calculate a transformed wheel-surface point by hand and compare it with TF.

### Phase 3 — spawn a passive robot in Gazebo Jetty

**Objective and concepts.** Gazebo integrates motion under gravity and contact constraints. It needs a collision ground plane, valid dynamic bodies, and a consistent time step. RViz can display a mechanically impossible robot perfectly. This phase is the first physical test. [Jetty worlds](https://gazebosim.org/docs/jetty/sdf_worlds/) · [Model spawning](https://gazebosim.org/docs/jetty/ros2_spawn_model/) · [URDF-to-SDF conversion](https://sdformat.org/tutorials/specification/sdformat_urdf_extensions/)

| Task | Work and verification |
|---|---|
| **3.1.1 — Create the simulation package and world** | Run `ros2 pkg create --destination-directory "$WS/src" --build-type ament_cmake --license MIT robot_gazebo`. Add `$G/worlds/lab.sdf` with world name `lab`, gravity, a static plane with collision and visual geometry, lighting, and measured box walls. Install `worlds/`. Include Jetty’s `Physics`, `UserCommands`, and `SceneBroadcaster` world systems explicitly when defining the system list. Build and run `ros2 launch ros_gz_sim gz_sim.launch.py gz_args:="-r $G/worlds/lab.sdf"`. `-r` starts simulation running. `gz service -l` should include `/world/lab/create`. |
| **3.1.2 — Establish model conversion and spawning** | Create `$G/urdf/sim.urdf.xacro` to include the description and later hold simulator extensions. Expand it to `/tmp/amr_sim.urdf`. Inspect `gz sdf -p /tmp/amr_sim.urdf`. In the running world use `ros2 launch ros_gz_sim gz_spawn_model.launch.py world:=lab file:=/tmp/amr_sim.urdf entity_name:=robot z:=0.02`. The 2 cm offset assumes the proposed ground-frame root. Expect exactly one robot to settle onto its supports. Missing bodies point first to inertias or conversion. |
| **3.2.1 — Establish simulation time** | Create `$G/config/bridge.yaml` with one Gazebo-to-ROS clock mapping. To isolate it before writing the YAML, use `ros2 run ros_gz_bridge parameter_bridge '/clock@rosgraph_msgs/msg/Clock[gz.msgs.Clock'`. The `[` selects Gazebo-to-ROS. `ros2 topic echo /clock --once` should advance between observations and stop advancing when paused. If empty, inspect `gz topic -l` for a world-scoped clock and map that Gazebo name to ROS `/clock`. |
| **3.2.2 — Assemble and test the simulation launch** | Create `$G/launch/sim.launch.py`. Start the world, one clock bridge, and one state publisher with the simulation Xacro and `use_sim_time=true`. Spawn once when the create service and description are available. Remove the separate diagnostic clock process. Install `launch/`, `config/`, `urdf/`, `worlds/`. Run `ros2 launch robot_gazebo sim.launch.py`. Observe 30 simulated seconds at rest. Inspect collisions if it sinks, jumps, or tips. Change only one of spawn height, geometry, inertia, time step, or friction per trial. |

**Completion:** three clean launches each produce one robot, one advancing ROS clock, and a stable body with the chassis off the floor. **Git:** `feature/gazebo-lab`, commit `Spawn the passive robot in a measured Jetty world`. Document the world dimensions, spawn pose, physics settings, and observed settle behavior. **Optional:** compare two time steps while holding model and contact settings constant.

### Phase 4 — differential drive, feedback, and teleoperation

**Objective and concepts.** Under rolling without slip, define wheel radius \(r\), center separation \(b\), wheel angular velocities \(\dot\phi_L,\dot\phi_R\), body forward speed \(v\), and yaw rate \(\omega\). Then

\[
v=\frac r2(\dot\phi_R+\dot\phi_L),\qquad
\omega=\frac r b(\dot\phi_R-\dot\phi_L),\qquad
\dot\phi_{L,R}=\frac{v\mp b\omega/2}{r}.
\]

At \(r=0.05\), \(b=0.33\), a straight command of \(0.10\) m/s requires **2 rad/s on both wheels**. A pure left turn of \(0.50\) rad/s requires **−1.65 rad/s left and +1.65 rad/s right**. Write these predictions down before the first motion test. [Lyrical drive controller](https://control.ros.org/lyrical/doc/ros2_controllers/diff_drive_controller/doc/userdoc.html) · [Gazebo control integration](https://control.ros.org/lyrical/doc/gz_ros2_control/doc/index.html)

| Task | Work and verification |
|---|---|
| **4.1.1 — Resolve control dependencies** | Add runtime dependencies in `$G/package.xml` for `gz_ros2_control`, `controller_manager`, `joint_state_broadcaster`, and `diff_drive_controller`. Include `ros2controlcli` for the development CLI in your environment instructions. `apt-cache policy ros-lyrical-gz-ros2-control ros-lyrical-ros2-control ros-lyrical-ros2-controllers ros-lyrical-ros2controlcli` shows candidates. Once available, install the needed packages or use `rosdep install --from-paths src --ignore-src --rosdistro lyrical -y` for declared dependencies. `ros2 pkg prefix gz_ros2_control` and `ros2 control --help` verify discovery. |
| **4.1.2 — Expose simulated wheel interfaces** | In `$G/urdf/control.xacro`, create one `ros2_control` system with hardware class `gz_ros2_control/GazeboSimSystem`. On each existing drive joint export a velocity command interface and position/velocity state interfaces. Load `libgz_ros2_control-system.so` with class `gz_ros2_control::GazeboSimROS2ControlPlugin` and its controller YAML path. Include it from the simulation Xacro. The plugin creates the controller manager. Relaunch and run `ros2 control list_hardware_interfaces`. Expect both wheel velocity command interfaces. A second controller manager or an additional Gazebo DiffDrive system would compete for the same robot. |
| **4.1.3 — Configure and activate controllers** | In `$G/config/controllers.yaml`, declare `joint_state_broadcaster/JointStateBroadcaster` and `diff_drive_controller/DiffDriveController`, naming the latter `base_controller`. Set actual left/right joint names, `wheel_radius: 0.05`, `wheel_separation: 0.33`, `position_feedback: true`, `open_loop: false`, `base_frame_id: base_footprint`, `odom_frame_id: odom`, and initially `enable_odom_tf: true`. Use a finite timeout such as 0.5 s. Spawn with `ros2 run controller_manager spawner joint_state_broadcaster -c /controller_manager` then `ros2 run controller_manager spawner base_controller -c /controller_manager`. Integrate these into launch after the manual test. `ros2 control list_controllers` should show both active. |
| **4.2.1 — Test bounded motion** | Execute the forward test below; repeat with `angular: {z: 0.5}` in place of `linear: {x: 0.1}`. Observe `/joint_states`, `/base_controller/odom`, and movement in the world. Wrong yaw sign points to swapped wheels or axis signs. Correct wheel speeds with poor body motion point to collision/friction. Commands received without movement suggest stale stamps, unclaimed interfaces, or paused physics. |
| **4.2.2 — Add teleoperation** | Run `ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args -r cmd_vel:=/base_controller/cmd_vel -p stamped:=true -p use_sim_time:=true`. Use the key bindings printed by the node. `ros2 topic info /base_controller/cmd_vel -v` must show compatible `TwistStamped` endpoints. Stop input and measure the command watchdog and physical stopping time separately. |

```bash
ros2 topic pub --use-sim-time --rate 10 --times 20 \
  /base_controller/cmd_vel geometry_msgs/msg/TwistStamped \
  '{header: {stamp: now}, twist: {linear: {x: 0.1}}}'
```

This sends 20 fresh-stamped commands at 10 Hz. Acceleration limits, the timeout interval, and stopping dynamics affect final distance. Compare **steady wheel speed** against the calculation and **actual traveled distance** against integrated feedback.

**Completion:** straight, reverse, and both turn directions behave correctly; steady wheel speeds are within an initial 10% target; the controller stops commanding motion after input expires. **Git:** `feature/differential-drive`, commit `Drive simulated wheels from stamped velocity commands`. Document signs, dimensions, timeout, and measured results. **Optional:** model motor lag and compare a proportional wheel-speed loop \(u=K_p(\dot\phi_{desired}-\dot\phi_{measured})\) with the ideal simulator interface.

### Phase 5 — simulate and validate each sensor

**Objective and concepts.** A sensor is a measurement model with a frame, timestamp, field of view, rate, noise, and transport policy. A frame existing in TF proves only that the mount exists. A message arriving proves transport. A controlled physical observation tests the measurement itself. [Jetty sensors](https://gazebosim.org/docs/jetty/sensors/) · [Version-matched bridge types and configuration](https://github.com/gazebosim/ros_gz/blob/3.0.10/ros_gz_bridge/README.md)

| Task | Work and verification |
|---|---|
| **5.1.1 — Establish mounts and world systems** | Add fixed mounts to `$MODEL` or an included description file. Add `$G/urdf/sensors.xacro` for sensor definitions. In `lab.sdf` add the Jetty `Sensors` system using Ogre2 for rendering sensors and the `Imu` system for the IMU. Validate the expanded simulation Xacro with `gz sdf -p /tmp/amr_sim.urdf` after regenerating it. Verify mounts using `ros2 run tf2_ros tf2_echo base_link laser_link`. If TF works but Gazebo has no sensor, inspect converted SDF and loaded world systems. |
| **5.1.2 — Bring up LiDAR alone** | Define a horizontal `gpu_lidar`, e.g. 360 samples, one vertical layer, 10 Hz, and a documented useful range. Choose an unobstructed height. Set the reported frame to `laser_link`. Add a `GZ_TO_ROS` bridge from `gz.msgs.LaserScan` to `sensor_msgs/msg/LaserScan` on `/scan`. Run `gz topic -l`, then `ros2 topic echo /scan --once --qos-reliability best_effort`. Place the laser origin 1 m from a perpendicular wall; the forward beam should read about 1 m. Determine its index using `angle_min` and `angle_increment`. |
| **5.1.3 — Bring up IMU alone** | Add a body-aligned IMU, e.g. 50 Hz, with an explicit `imu_link` frame. Bridge `gz.msgs.IMU` to `sensor_msgs/msg/Imu` on `/imu/data`. `ros2 topic echo /imu/data --once --qos-reliability best_effort` should show near-zero angular rate at rest and positive Z rate in a left turn. Inspect reported accelerations and gravity convention before enabling any acceleration fusion. |
| **5.2.1 — Add RGB and calibration** | Add a forward camera, initially 640×480 at 10–15 Hz. Bridge `gz.msgs.Image` and `gz.msgs.CameraInfo` to their `sensor_msgs` equivalents on `/camera/image_raw` and `/camera/camera_info`. Both should report `camera_optical_frame`; apply any bridge frame override per affected topic. Run `ros2 run rqt_image_view rqt_image_view /camera/image_raw` and inspect `ros2 topic echo /camera/camera_info --once --qos-reliability best_effort`. Center a target physically and check the image center. |
| **5.2.2 — Measure rate and noise** | In `$G/urdf/sensors.xacro`, keep an ideal baseline and a documented modest-noise configuration. Calculate a stationary sample mean and standard deviation for wall range and gyro Z. Use `ros2 topic hz /scan` and `ros2 topic info /scan -v` to distinguish missing samples, QoS mismatch, and slow rendering. First validate changes with one sensor enabled. Enable all sensors and compare again. |

For bridge YAML, each entry can specify `gz_topic_name`, `ros_topic_name`, `gz_type_name`, `ros_type_name`, and `direction`. Use actual Gazebo topic names from `gz topic -l`. Give sensor subscribers a compatible sensor QoS profile. A reliable subscriber requesting data from a best-effort-only publisher can appear silent despite matching names and message types.

**Completion:** a known 1 m wall measures within an initial 5 cm budget, gyro sign passes the turn test, the camera sees the forward target, and delivered rates and header frames are recorded. **Git:** `feature/simulated-sensors`, commit `Validate lidar IMU and camera measurements in the lab`. Document a sensor specification table. **Optional:** add registered depth only when a task needs metric target position.

### Phase 6 — wheel odometry and state estimation

**Objective and concepts.** Encoders measure wheel rotation. Odometry integrates it under a contact model. Let \(\Delta\phi_L,\Delta\phi_R\) be wheel-angle increments and \((x,y,\theta)\) the previous planar pose:

\[
d_L=r\Delta\phi_L,\quad d_R=r\Delta\phi_R,\quad
d=(d_L+d_R)/2,\quad \delta=(d_R-d_L)/b,
\]
\[
x'=x+d\cos(\theta+\delta/2),\quad
y'=y+d\sin(\theta+\delta/2),\quad
\theta'=\theta+\delta .
\]

The midpoint expression approximates an arc over a short interval. The exact constant-curvature displacement adds a factor \(\sin(\delta/2)/(\delta/2)\) to the translational increment, with limiting value 1 at zero. Either method can be checked on straight, spin, and circular trajectories.

An EKF predicts state and covariance, then corrects them with measurements. In
\[
P^-=FPF^\top+Q,\qquad K=P^-H^\top(HP^-H^\top+R)^{-1},
\]
\(P\) is state uncertainty, \(F\) the local motion Jacobian, \(Q\) process uncertainty, \(H\) the measurement Jacobian, \(R\) measurement uncertainty, and \(K\) the gain weighting the measurement residual. Wheel speed and an independently simulated gyro offer complementary information. Encoder pose and encoder twist derived from the same wheel samples are correlated. [Upstream EKF parameters](https://github.com/cra-ros-pkg/robot_localization/blob/3.10.0/params/ekf.yaml)

| Task | Work and verification |
|---|---|
| **6.1.1 — Write a teaching integrator** | Create `robot_navigation` as an `ament_cmake` package using the Phase 3 package-create command with `robot_navigation` as the name. Write `$N/tools/encoder_odometry.py` with a pure integration function and `$N/test/test_encoder_odometry.py`. Run `python3 -m pytest "$N/test/test_encoder_odometry.py"`. Test zero increments, one full turn of both wheels, opposite increments, and asymmetric motion. One equal revolution should translate \(2\pi r \approx 0.31416\) m. |
| **6.1.2 — Compare real feedback and independent truth** | Extend the exercise to consume `/joint_states` by name, initialize on the first sample, handle nonpositive time intervals/reset, and log an educational trajectory. Keep it off the production TF tree. Add an evaluation-only truth stream from Gazebo world/model pose, e.g. Jetty’s `OdometryPublisher` bridged to `/evaluation/ground_truth`. Run `ros2 topic info /evaluation/ground_truth -v` and check its frame origin. Compare encoder integration, controller odometry, and world motion on a straight line and a square. Align starting frames and timestamps. [Jetty truth publisher implementation](https://github.com/gazebosim/gz-sim/blob/gz-sim10/src/systems/odometry_publisher/OdometryPublisher.cc) |
| **6.2.1 — Configure the planar filter** | Add `robot_localization` as a runtime dependency. Create `$N/config/ekf.yaml` with node key `ekf_filter_node`, `two_d_mode: true`, `world_frame: odom`, `odom_frame: odom`, `base_link_frame: base_footprint`, `publish_tf: true`, `odom0: /base_controller/odom`, and `imu0: /imu/data`. Start with wheel forward velocity and IMU Z angular velocity. In the 15-field selection order \(x,y,z,roll,pitch,yaw,v_x,v_y,v_z,\omega_x,\omega_y,\omega_z,a_x,a_y,a_z\), enable indices 6 and 11 respectively, counted from zero. Supply plausible nonzero variances for selected measurements. |
| **6.2.2 — Transfer TF ownership and verify** | Set `enable_odom_tf: false` in the fused controller profile, restart simulation, then run `ros2 run robot_localization ekf_node --ros-args -r __node:=ekf_filter_node --params-file "$N/config/ekf.yaml" -p use_sim_time:=true`. Inspect `/odometry/filtered`, `ros2 param get /ekf_filter_node two_d_mode`, and `ros2 run tf2_ros tf2_echo odom base_footprint`. If the filter stays silent, check first measurement arrival, frame connectivity, parameter node key, and simulated time. |
| **6.2.3 — Quantify fusion** | Record matched trials with ideal contacts, modest slip, gyro noise, and a short IMU interruption. Compare endpoint translation and wrapped yaw error with truth. Inspect covariance growth and recovery. If the filter is overconfident, revisit measurement variances and correlated inputs before changing filter frequency. Save results in `$N/README.md`. |

**Completion:** analytical unit tests pass; a 1 m straight-line run is within an initial 5 cm target on high-traction ground; fused odometry is continuous; exactly one component owns `odom → base_footprint`; and a table explains when fusion helps or hurts. **Git:** `feature/planar-estimation`, commit `Compare encoder odometry and planar sensor fusion`. **Optional:** estimate a gyro bias or compare midpoint and exact-arc integration at a low sample rate.

### Phase 7 — map an unknown world

**Objective and concepts.** LiDAR supplies local geometry. Odometry supplies a motion prior. SLAM estimates both trajectory and map. An occupancy grid can represent evidence as log odds \(L=\log[p/(1-p)]\), where \(p\) is the probability a cell is occupied. A beam adds free-space evidence along its ray and occupied evidence near a valid return. A pose graph adds constraints between poses; revisiting a recognizable place supplies a loop-closure constraint that can correct accumulated error. [SLAM Toolbox](https://github.com/SteveMacenski/slam_toolbox) · [Lyrical mapping and localization guide](https://docs.nav2.org/lyrical/configuration_and_development/first_time_robot_setup_guide/sensors/mapping_localization/)

| Task | Work and verification |
|---|---|
| **7.1.1 — Prove scan/TF readiness** | Use `ros2 topic hz /scan`, `ros2 run tf2_ros tf2_echo odom laser_link`, and an RViz `LaserScan` display fixed to `odom`. Drive slowly past a wall. Its points should remain approximately stationary. A wall rotating with the robot indicates a frame or timestamp problem. Resolve this before tuning SLAM. |
| **7.1.2 — Configure online mapping** | Install/declare `slam_toolbox`. Copy its installed `config/mapper_params_online_async.yaml` to `$N/config/slam.yaml`. Set `mode: mapping`, `map_frame: map`, `odom_frame: odom`, `base_frame: base_footprint`, `scan_topic: /scan`, a nonzero transform publication period, and e.g. `resolution: 0.05`. Run `ros2 launch slam_toolbox online_async_launch.py use_sim_time:=true slam_params_file:="$N/config/slam.yaml"`. `ros2 lifecycle get /slam_toolbox` should report active. [Upstream launch arguments](https://github.com/SteveMacenski/slam_toolbox/blob/2.10.0/launch/online_async_launch.py) |
| **7.2.1 — Make a controlled mapping run** | Teleoperate an overlapping loop through a room, doorway, and asymmetric corridor. Revisit the starting wall with comparable viewpoint. Display `/map` in `map`, then run `ros2 topic echo /map --once --field info --qos-durability transient_local`. Check cell resolution, nonzero dimensions, wall thickness, loop alignment, and a measured wall separation. Double walls can originate from bad wheel scale, scan timing, or fast rotation. |
| **7.2.2 — Save a reusable artifact** | Create `$N/maps`, then run `ros2 run nav2_map_server map_saver_cli -f "$N/maps/lab"`. Declare/install `nav2_map_server` by this milestone. `-f` selects the output basename. Open `$N/maps/lab.yaml` and its referenced image. Keep the pair together and install `maps/`. Verify resolution, origin, and image path. Saving a **pose graph** for later continuation is a separate SLAM Toolbox operation. |

**Completion:** the map reloads from its YAML/image pair, a known wall spacing agrees within about 0.10 m at 0.05 m resolution, and the loop has no persistent duplicate wall larger than two cells in the evaluation area. **Git:** `feature/lidar-mapping`, commit `Build and save a measured lidar map`. Document the mapping route, map frame, resolution, and source world revision. **Optional:** serialize and resume the pose graph.

### Phase 8 — localize in the saved map

**Objective and concepts.** AMCL maintains particles representing possible robot poses. Motion updates propagate them using odometric change and noise. Scan updates weight them by map agreement. Resampling concentrates particles in likely regions. In a symmetric corridor, different positions can explain similar scans. A well-chosen initial distribution and a path past distinctive geometry make the experiment interpretable. [Lyrical AMCL configuration](https://docs.nav2.org/lyrical/configuration_and_development/configuration_guide/others/configuring_amcl/)

| Task | Work and verification |
|---|---|
| **8.1.1 — Serve the map independently** | Create `$N/launch/localization.launch.py` with `nav2_map_server`’s `map_server`, `yaml_filename` set to the installed map, and simulation time. Add AMCL and a lifecycle manager responsible for just these two nodes. Set its `autostart` true and node order `map_server`, `amcl`. Stop mapping first. Build, launch `ros2 launch robot_navigation localization.launch.py`, and check `ros2 lifecycle get /map_server`. An inactive server can be discoverable while serving no usable map. |
| **8.1.2 — Set AMCL’s measurement contract** | In `$N/config/nav2.yaml`, set `global_frame_id: map`, `odom_frame_id: odom`, `base_frame_id: base_footprint`, `scan_topic: /scan`, differential motion model, and `tf_broadcast: true` under `amcl`. Run `ros2 lifecycle get /amcl` and inspect `/scan` plus the map. A map visible in RViz only proves the map server path. |
| **8.2.1 — Initialize and converge** | In RViz use **2D Pose Estimate** near the true pose and heading in the saved map. Inspect `ros2 topic echo /amcl_pose --once` and `ros2 run tf2_ros tf2_echo map base_footprint`. Drive a short route with a turn. The scan should align with walls and uncertainty should generally contract after informative motion. Confirm the map origin convention if the result appears mirrored or offset. |
| **8.2.2 — Test uncertainty and ownership** | Repeat from three known starts, including one deliberately offset initialization. Compare estimated and truth poses in the same coordinates; record convergence distance and time. Stop SLAM Toolbox in this mode. If `map → odom` jumps back and forth or TF reports competing data, inspect which global publishers are alive. |

**Completion:** from three documented starts, converge within a proposed 0.15 m and 10° after a short informative path, with the saved grid unchanged and one map transform authority. **Git:** `feature/map-localization`, commit `Localize the robot on a saved map with AMCL`. Document initial-pose procedure and failure in a symmetric area. **Optional:** compare AMCL with SLAM Toolbox localization mode on the same trials.

### Phase 9 — autonomous navigation and obstacle response

**Objective and concepts.** The global planner searches a traversability representation. In A*, \(f(n)=g(n)+h(n)\), where \(g\) is accumulated path cost and \(h\) an estimated remaining cost. Dijkstra uses \(h=0\). The local controller selects motion repeatedly using the current pose, route, robot limits, and nearby obstacles. The Lyrical Nav2 reference configuration includes NavFn and a differential-drive MPPI controller. MPPI samples candidate control sequences and evaluates predicted outcomes. It is a model-based controller and requires no learned detector or training dataset. [Lyrical reference parameters](https://github.com/ros-navigation/navigation2/blob/1.5.2/nav2_bringup/params/nav2_params.yaml)

| Task | Work and verification |
|---|---|
| **9.1.1 — Start from the installed Nav2 release** | Declare `nav2_bringup` and the navigation stack used by your launch. Inspect `ros2 launch nav2_bringup bringup_launch.py --show-args` and the installed `params/nav2_params.yaml`. Merge your AMCL settings into a project copy. Preserve required plugin sections and node keys. Set sim time, base frames, controller speed limits, and the `/odometry/filtered` topic wherever a server consumes odometry. |
| **9.1.2 — Configure both costmaps** | Set the global costmap frame to `map`, local rolling costmap frame to `odom`, and robot frame to `base_footprint`. Use static map data globally, LiDAR obstacle marking and raytrace clearing, and inflation. Give both costmaps the actual polygon. For the worked design a conservative rectangular starting outline is `[[0.165,0.18],[0.165,-0.18],[-0.235,-0.18],[-0.235,0.18]]` in metres. Recheck after sensor or bumper changes. In RViz verify the footprint encloses wheels and that obstacles appear and clear. [Nav2 footprint guide](https://docs.nav2.org/lyrical/configuration_and_development/first_time_robot_setup_guide/footprint/setup_footprint/) |
| **9.1.3 — Wire the velocity chain** | Set the collision monitor’s input to `cmd_vel_smoothed` and output to `/base_controller/cmd_vel`. Configure its scan source, robot frame, footprint/stop zone, and source timeout. Check the scan’s physical height against its height filters. Align smoother/controller limits with Phase 4 measurements. Use `ros2 topic info /base_controller/cmd_vel -v` to verify the final publisher. Stop keyboard teleop for this mode. |
| **9.2.1 — Start full bringup and send a measured goal** | Stop the standalone Phase 8 localization launch. Keep simulated hardware, sensors, and estimator running. Launch Nav2 with the command below. The top-level launch owns localization and lifecycle bringup for this mode. Initialize the pose, inspect `ros2 lifecycle get /bt_navigator`, and `ros2 action list -t`. Send the example goal only if `(1,0)` is free in your actual map; otherwise use a measured free coordinate. Check feedback and the final result. |
| **9.2.2 — Exercise replanning and failure** | Test a clear corridor, a static obstruction with a detour, a completely blocked route, a canceled goal, and an obstacle introduced after planning. Use RViz to inspect the global path, local costmap, and footprint. Check `ros2 topic echo /base_controller/cmd_vel --once` during a failed run to locate the first silent stage. A valid planner route with no actuator command points downstream of planning. Record contacts from simulation or an evaluator, including the robot’s sides during rotation. |

```bash
ros2 launch nav2_bringup bringup_launch.py \
  map:="$N/maps/lab.yaml" params_file:="$N/config/nav2.yaml" \
  use_sim_time:=true slam:=false autostart:=true \
  use_keepout_zones:=false use_speed_zones:=false
```

```bash
ros2 action send_goal /navigate_to_pose nav2_msgs/action/NavigateToPose \
  '{pose: {header: {frame_id: map}, pose: {position: {x: 1.0, y: 0.0}, orientation: {w: 1.0}}}}' \
  --feedback
```

The zone flags select a first environment without zone-mask maps. The identity quaternion gives zero yaw in the example goal. The action feedback shows progress; its terminal state tells the mission whether navigation succeeded. [Versioned bringup launch](https://github.com/ros-navigation/navigation2/blob/1.5.2/nav2_bringup/launch/bringup_launch.py) · [NavigateToPose definition](https://github.com/ros-navigation/navigation2/blob/1.5.2/nav2_msgs/action/NavigateToPose.action)

**Completion / MVP demonstration:** at least 9 of 10 reachable-goal trials succeed within a documented 0.15 m positional tolerance, with zero obstacle contacts across the trial set; an unreachable goal terminates under a bounded policy; cancellation brings the robot to rest. **Git:** `feature/nav2-navigation`, commit `Navigate mapped routes with lidar obstacle response`. Document goals, outcomes, measured footprint, speed limits, and one failed trial. **Optional:** compare Dijkstra and A* with the same costmap and controller.

### Phase 10 — classical computer vision

**Objective and concepts.** Start with a target you can label and an algorithm you can explain. A colored inspection marker is suitable. For a rectified pinhole camera,
\[
u=f_xX/Z+c_x,\quad v=f_yY/Z+c_y,
\]
where \((X,Y,Z)\) is an optical-frame point, \((u,v)\) its pixel location, \(f_x,f_y\) focal lengths in pixels, and \(c_x,c_y\) principal point. A pixel gives a ray. Metric depth still needs another observation or known geometry. [OpenCV thresholding tutorial](https://docs.opencv.org/4.x/da/d97/tutorial_threshold_inRange.html) · [Detection message definition](https://github.com/ros-perception/vision_msgs/blob/ros2/vision_msgs/msg/Detection2D.msg)

| Task | Work and verification |
|---|---|
| **10.1.1 — Define and collect the baseline** | Run `ros2 pkg create --destination-directory "$WS/src" --build-type ament_python --license MIT robot_perception`. Add target and distractor props to an inspection variant of the Gazebo world. Record positives, negatives, off-center examples, and partial occlusion using `ros2 bag record -o /tmp/amr_camera_baseline --topics /camera/image_raw /camera/camera_info /clock`. Use a new output path for each run. `ros2 bag info /tmp/amr_camera_baseline` should show both camera topics. |
| **10.1.2 — Write a pure detector** | In `$P/robot_perception/color_detector.py`, convert BGR to HSV, threshold, remove isolated noise, select contours by area/shape, and return pixel boxes. Keep OpenCV operations independent of ROS. In `$P/test/test_color_detector.py`, use positive, negative, and border-touching images. `python3 -m pytest "$P/test/test_color_detector.py"` should locate the target and produce an empty result for negatives. Inspect HSV hue wraparound if using red. |
| **10.2.1 — Wrap it in ROS** | Add a node and `setup.py` entry point named `target_detector`. Declare actual dependencies including `rclpy`, `sensor_msgs`, `cv_bridge`, and `vision_msgs`. Use `cv_bridge` with an explicit target encoding. Publish `/perception/detections` with source headers. Run `ros2 run robot_perception target_detector --ros-args -p use_sim_time:=true` and `ros2 interface show vision_msgs/msg/Detection2DArray`. Overlay predicted boxes on the source image for inspection. Empty frames should yield an empty array. |
| **10.2.2 — Measure failure boundaries** | In `$P/config/color.yaml`, expose thresholds and minimum area. Compare at least two lighting conditions, multiple distances, and a same-color distractor. Replay a recorded run in isolation with `ros2 bag play /tmp/amr_camera_baseline`. Record true detections, misses, false positives, and processing latency. If raw transport works but compressed transport fails, check image-transport plugin availability and subscriber transport choice separately. |

**Completion:** repeatable box coordinates on fixtures, correct source timestamps, no false detection on the defined blank scene, and measured precision/recall on held-out runs. **Git:** `feature/classical-perception`, commit `Detect and evaluate inspection markers from camera images`. Document target appearance and known failure conditions. **Optional:** recover the pose of a calibrated, known-size marker.

### Phase 11 — add a genuinely useful learned capability

**Objective and concepts.** Make the learning problem **appearance recognition under variation**. For example, distinguish the target inspection object from similarly colored distractors. This provides a measurable reason to try ML. Keep localization, kinematics, controller limits, and mission timeouts in the established robotics stack.

A detector learns parameters \(\theta\) mapping an image to class scores and boxes. Intersection over union is \(\mathrm{IoU}=|A\cap B|/|A\cup B|\) for predicted box \(A\) and labeled box \(B\). At a fixed matching threshold, precision is \(TP/(TP+FP)\) and recall \(TP/(TP+FN)\), where \(TP,FP,FN\) count matched predictions, spurious predictions, and missed targets. Use one-to-one matching so multiple predictions for the same target do not inflate success. [OpenCV DNN inference](https://docs.opencv.org/4.x/d6/d0f/group__dnn.html) · [COCO evaluation reference](https://github.com/cocodataset/cocoapi/blob/master/PythonAPI/pycocotools/cocoeval.py)

| Task | Work and verification |
|---|---|
| **11.1.1 — Specify a fair experiment** | Create `$P/evaluation/dataset_manifest.json` and `$P/evaluation/README.md` describing class definitions, annotated boxes, negative examples, world/run IDs, license, and splits. Split by run or world before sampling frames. Write and run `python3 -m pytest "$P/test/test_dataset_split.py"` to ensure run IDs are disjoint. Adjacent frames from one drive create misleadingly good test results if split randomly. |
| **11.1.2 — Verify one learned model offline** | Select a licensed model whose classes include your object, or fine-tune on your training split if the class is new. Record model source, checksum, tensor shape, color order, normalization, resize/letterbox transform, label order, and output decoding in `$P/evaluation/model.md`. Implement an adapter in `$P/robot_perception/learned_detector.py`. A CPU-compatible ONNX model through OpenCV DNN is one option; verify operator support on your installed build. Run `python3 -m pytest "$P/test/test_learned_detector.py"` on a labeled fixture before ROS integration. |
| **11.2.1 — Evaluate both methods on identical inputs** | Write `$P/evaluation/evaluate.py` with a documented interface. A suitable target command is `python3 "$P/evaluation/evaluate.py" --manifest "$P/evaluation/dataset_manifest.json" --split test`. Report precision/recall at IoU 0.5, errors by lighting and distance, and median/p95 latency. Choose confidence thresholds on validation data, then evaluate the held-out test split. If boxes are systematically displaced, reverse preprocessing geometry before retuning confidence. |
| **11.2.2 — Integrate through the existing contract** | Add a project parameter `backend` to `target_detector`. Run `ros2 run robot_perception target_detector --ros-args -p backend:=learned -p use_sim_time:=true`. Keep the same detection topic, message type, and timestamps. Bound queued frames so old detections cannot accumulate if inference is slow. Measure wall-time latency and simulation-time observation age separately. |

**Completion:** an independently reproducible comparison, at least one documented appearance condition where the model’s benefit can be measured, and an explicit decision on its use in the mission. A negative result is valid if the evaluation explains it. **Git:** `feature/learned-perception`, commit `Evaluate learned recognition against the classical baseline`. Document accuracy, runtime, model provenance, and how to reproduce inference. **Optional:** fine-tune with domain randomization and evaluate in an unseen world.

### Phase 12 — execute an autonomous inspection mission

**Objective and concepts.** Use a **finite-state machine** you write yourself and Nav2’s existing action interface. A good first mission visits surveyed observation sites, identifies a target, visits the associated safe inspection pose, verifies it again, and reports completion. This is achievable with RGB and LiDAR. It does not depend on inferring arbitrary metric range from an image box. [Nav2 Simple Commander](https://docs.nav2.org/lyrical/configuration_and_development/simple_commander_api/simple_commander_api/)

| Task | Work and verification |
|---|---|
| **12.1.1 — Author the mission geometry** | Create `robot_bringup` as `ament_python`. In `$B/config/inspection_sites.yaml`, define site IDs, map-frame search poses, approach poses, target class, standoff, and timeouts. Verify each pose manually with the Phase 9 action command. Test the footprint at the final heading. A target visible from a point does not establish that the base can safely occupy it. |
| **12.1.2 — Implement deterministic transitions** | In `$B/robot_bringup/inspection_mission.py`, implement `WAIT_READY → GO_TO_SITE → OBSERVE → APPROACH → VERIFY → REPORT`. Add explicit next-site, canceled, timed-out, and failed transitions. Require fresh observations after arrival, e.g. three consistent detections across distinct recent frames. Unit-test using a fake action client and simulated clock. Run `python3 -m pytest "$B/test/test_inspection_mission.py"`. |
| **12.2.1 — Connect navigation and perception** | Use `nav2_simple_commander` or an asynchronous `rclpy` action client. Wait for active navigation, send a `PoseStamped`, monitor feedback, and interpret the terminal result. Keep callbacks spinning during motion and observation. Cancel an outstanding goal before switching sites. Publish `/mission/status` and save a terminal record. Install the `inspection_mission` entry point and run `ros2 run robot_bringup inspection_mission --ros-args -p use_sim_time:=true` against an already running stack. |
| **12.2.2 — Provide one integrated launch and failure cases** | Create `$B/launch/mission.launch.py` to compose simulation, estimation, localization/navigation, perception, and the mission exactly once. Explicitly expose initial-pose procedure, map, and mission-site config. Run `ros2 launch robot_bringup mission.launch.py`. Test target present, absent, stale detection, blocked approach, and cancellation. Inspect `ros2 topic echo /mission/status`. Each trial must terminate within a configured bound with an accurate reason. |

**Completion:** three successful inspection runs, one target-absent run and one blocked-approach run correctly reported, no obstacle contact, and no active drive goal left after termination. **Git:** `feature/inspection-mission`, commit `Execute and report autonomous inspection missions`. Document the state diagram and transition conditions. **Optional:** add unknown-location approach using calibrated registered depth, TF at acquisition time, and a tested collision-free standoff pose.

### Phase 13 — test, measure, and optimize

**Objective and concepts.** Unit tests isolate mathematics. Launch tests check process and communication contracts. Scenario tests measure the robot’s behavior. A trajectory error metric only means something after matching timestamps and reference frames.

For \(n\) aligned samples with estimated and true planar positions \(\hat p_i,p_i\), compute
\[
\mathrm{RMSE}_{xy}=\sqrt{\frac1n\sum_i\|\hat p_i-p_i\|^2}.
\]
Wrap yaw residuals to \([-\pi,\pi)\). Report navigation success rate as successes divided by attempted trials, with counts for cancellation, collision, planner failure, and timeout. Real-time factor is elapsed simulated time divided by elapsed wall time. [ROS launch testing](https://github.com/ros2/launch/tree/lyrical/launch_testing) · [ROS bag tooling](https://github.com/ros2/rosbag2)

| Task | Work and verification |
|---|---|
| **13.1.1 — Register tests with the package build** | Add model expansion/tree checks to `robot_description`, control-geometry consistency checks to `robot_gazebo`, mathematical tests to `robot_navigation`, and detector/mission tests to their Python packages. Register CMake-package Python tests with `ament_cmake_pytest` or an appropriate ament test mechanism. Run `colcon test --packages-select robot_description robot_gazebo robot_navigation robot_perception robot_bringup` then `colcon test-result --verbose`. Verify that the expected tests actually ran; an empty test suite can also give a clean summary. |
| **13.1.2 — Add startup integration checks** | Create launch tests for installed model availability, advancing `/clock`, active controllers, nonempty joint/scan data, transform lookup, and Nav2 action readiness. Use bounded waits for observable conditions. Test a missing bridge or invalid wheel name to verify that the intended check fails with a useful diagnostic. Keep heavy rendering tests separate from quick headless tests. |
| **13.2.1 — Record and replay representative trials** | Record `ros2 bag record -o /tmp/amr_nav_trial_01 --topics /clock /tf /tf_static /scan /imu/data /joint_states /base_controller/odom /odometry/filtered /evaluation/ground_truth /base_controller/cmd_vel /mission/status`. Inspect `ros2 bag info /tmp/amr_nav_trial_01`. Use an isolated replay graph, a single clock source, and compatible static-TF durability. Record action outcomes and contacts in the scenario result as well. |
| **13.2.2 — Build an evaluation matrix** | In `$B/test/scenarios/`, define seeds, initial poses, goals, layout changes, and perturbations. Include clear, cluttered, narrow passage, blocked path, sensor dropout, distractor, and changed lighting. Write an evaluator for pose RMSE, goal error, elapsed time, shortest clearance, obstacle contacts, route length, inference age, and real-time factor. Version the thresholds with the scenarios. Compare repeated runs and report spread as well as mean. |
| **13.2.3 — Optimize a measured bottleneck** | If real-time factor drops, compare world-only, LiDAR-only, camera-enabled, and inference-enabled runs. Use `ros2 topic hz /camera/image_raw`, `ros2 topic hz /scan`, and logged callback durations. Change one of camera resolution/rate, costmap resolution, controller rate, or inference rate. Rerun the identical scenario set and retain the change only if behavior remains within acceptance bounds. |

**Completion:** a clean test report, at least ten repeatable navigation trials, a full mission failure matrix, and one before/after experiment tied to a measured bottleneck. **Git:** `feature/scenario-evaluation`, commit `Measure navigation perception and mission robustness`. Document hardware, model revisions, seeds, thresholds, and worst cases. **Optional:** containerize headless CI and run the same scenario suite on ARM64.

### Phase 14 — document and demonstrate the project

**Objective and concepts.** A reproducible demonstration lets a reader distinguish what the robot did autonomously, where its inputs came from, and how reliably it worked. Build the portfolio around one measured mission and one informative failure.

| Task | Work and verification |
|---|---|
| **14.1.1 — Write the entry path** | Expand `$WS/README.md` with requirements, dependency resolution, build, sourcing, display, manual simulation, mapping, localization, navigation, and mission commands. Give the order and terminal count for each mode. Run `ros2 launch robot_bringup mission.launch.py --show-args` and compare the output with the documentation. |
| **14.1.2 — Publish the engineering evidence** | Add `docs/architecture.md`, `docs/evaluation.md`, a TF diagram, package graph, and dimensioned robot sketch. Include command ownership, sensor specifications, action contract, map origin, mass assumptions, and the accuracy/runtime table. Use actual `view_frames` output and measured trials for the implemented revision. |
| **14.2.1 — Rehearse a clean reproduction** | In a fresh checkout or clean environment, follow only the README. Use `rosdep install --from-paths src --ignore-src --rosdistro lyrical -y`, `colcon build --symlink-install`, `source install/local_setup.bash`, and the documented launch. Check for accidental references to your home directory, uninstalled resources, missing models, or absolute texture paths. |
| **14.2.2 — Record the demonstration** | Capture Gazebo, RViz, the target detection, mission state, and final report. Show one obstacle response and one correctly handled failure. Include the map/world revision, goal, start conditions, runtime, and measured success rate in the caption. Have another reader reproduce one scenario from the documented revision. |

**Completion:** an independent reader can build, spawn, navigate, and run the mission from the documented inputs. **Git:** `docs/final-demonstration`, commit `Document and demonstrate the autonomous inspection robot`. **Optional:** provide a short student lab guide with an exercise to diagnose one broken transform.

---

## Part V. Level 3 — the next 16 actions, in order

These are the actions I would take **before working on Gazebo control, sensors, or Nav2**. They begin with the exact current repository state. Commands use the shell variables defined in Part IV.

| # | Exact action | Expected result and interpretation |
|---:|---|---|
| **1** | Open a fresh Bash terminal. Set `WS`, `D`, and `MODEL` as above. Run `cd "$WS"`, `cat /etc/os-release`, `dpkg --print-architecture`, and `source /opt/ros/lyrical/setup.bash`. Then run `printenv ROS_DISTRO` and `command -v ros2 colcon xacro check_urdf`. | Ubuntu 26.04, `amd64`, `lyrical`, four resolvable executables. If Xacro or `check_urdf` is missing from the shell, check sourcing before installing anything. |
| **2** | Inspect `git status --short --branch`, `colcon list`, and `code "$WS/.vscode/settings.json"`. Compare the workspace root with the configured CMake source directory. | Expect the feature branch, untracked `.vscode/` unless you have since changed it, and one source package. Decide whether to keep the editor setting personal or make it portable. It does not control colcon package discovery. |
| **3** | Run `colcon build --symlink-install --packages-select robot_description`. Open a second terminal, source `/opt/ros/lyrical/setup.bash` and `"$WS/install/local_setup.bash"` after setting `WS`, then run `ros2 pkg prefix --share robot_description`. | One completed package and a share directory under this workspace. If colcon reports conflicting previous build settings, use a fresh build base for diagnosis instead of deleting your existing workspace. |
| **4** | Run `ros2 run demo_nodes_cpp talker` in one terminal and `ros2 run demo_nodes_py listener` in another. Inspect `ros2 topic info /chatter -v` in a third. Stop the demos after five messages. | A working cross-language DDS exchange. If nodes cannot see one another, compare domain, middleware, and sourcing. This isolates environment problems from the robot package. |
| **5** | Run `xacro "$MODEL" -o /tmp/amr_before.urdf`, then `check_urdf /tmp/amr_before.urdf`. Open `/tmp/amr_before.urdf` and the [original Xacro](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/src/robot_description/urdf/robot.urdf.xacro) side by side. | The expanded file should contain numerical dimensions and the `chassis → left-wheel-link` tree. This is your first runtime verification of the repository assessment. Retain the temporary baseline for comparison. |
| **6** | Run `ls "$(ros2 pkg prefix --share robot_description)"`. Open [CMakeLists.txt](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/src/robot_description/CMakeLists.txt) and [package.xml](/home/enrico/Projects/AI-Powered-Autonomous-Mobile-Robot/src/robot_description/package.xml). Add the `urdf` installation rule before `ament_package()` and the `xacro` runtime dependency. | Before editing, the installed share should lack `urdf`. Understand the distinction between a build-system dependency, a runtime dependency, and a resource installation rule. You need the last two for your model. |
| **7** | Rebuild `robot_description`. In the sourced runtime terminal run `xacro "$(ros2 pkg prefix --share robot_description)/urdf/robot.urdf.xacro" -o /tmp/amr_from_share.urdf`, `check_urdf /tmp/amr_from_share.urdf`, and `diff -u /tmp/amr_before.urdf /tmp/amr_from_share.urdf`. | Both generated models should describe the same tree and numbers. Xacro may include a source-path comment differing between source and install paths; inspect the diff. A missing file or changed geometry is significant. |
| **8** | In `README.md`, draw or tabulate wheel radius 0.05, track 0.33, outside width 0.36, body dimensions, proposed axle height 0.05, chassis lift 0.05 relative to axle, and desired clearance 0.025, all in metres. Mark measured-from-source values and proposed values separately. | You can explain why the current body bottom is 25 mm too low and predict the corrected body bottom without launching a viewer. This becomes the dimensional reference for controllers and costmaps. |
| **9** | Open `"$MODEL"`. Introduce `base_footprint` as root and `base_link` as its fixed child at `xyz="0 0 0.05"`, or use the equivalent Xacro property. Transfer the original chassis visual to `base_link` with the proposed center `-0.035 0 0.05`. Reparent the existing wheel joint to `base_link`. Expand and parse after this edit alone. | Expect one root and a connected left wheel. If `check_urdf` reports two roots, the original `chassis` link or an obsolete parent reference likely remains. The project name and robot name can stay as they are. |
| **10** | Create `"$D/urdf/wheel.xacro"`. Move the left wheel definition into a macro. Keep visual roll at `pi/2`; set joint rotation to zero and axis to `0 1 0`. Invoke it once for the left side. For this refactor, use `left_wheel_link` and `left_wheel_joint` consistently. Expand and parse. | The tree still contains one wheel at `+0.165` m. Underscores are a deliberate project convention; the original hyphenated names were not by themselves proof of a parsing error. Isolating this step makes macro mistakes easy to find. |
| **11** | Add the right-side invocation at `-0.165` m. Run `xacro "$MODEL" -o /tmp/amr_model.urdf`, `check_urdf /tmp/amr_model.urdf`, and `rg -n 'wheel_(joint\|link)|axis xyz' /tmp/amr_model.urdf` or inspect the XML directly. | Two distinct continuous joints, opposite lateral positions, same axis sign, and 0.330 m center separation. Duplicate names mean the macro failed to use its side parameter everywhere. |
| **12** | Add `caster_link` and a fixed joint to `base_link`. Use the worked support radius and center from Part II. Add chassis, wheel, and caster collision elements, preserving the wheel cylinder’s rotation in its collision element. Regenerate and parse the URDF. | All three intended supports meet the same plane and the box clears it by 0.025 m. An unrotated wheel collision cylinder could look correct in RViz’s visual display and still be wrong in physics. |
| **13** | Add mass and inertial elements to physical bodies. Use the formulas in Phase 1 and document the assumed masses. Run `gz sdf -p /tmp/amr_model.urdf` **after regenerating** `/tmp/amr_model.urdf`. Inspect both wheel joints and the resulting body inertias. | Conversion retains meaningful physical bodies and both movable joints. Check the wheel’s axial moment is assigned to Y for the chosen inertial-frame orientation. Fixed-joint reduction can legitimately change the SDF link count. |
| **14** | Create `"$D/launch/display.launch.py"`. Use the installed package share to find the Xacro, then start `robot_state_publisher`, `joint_state_publisher_gui`, and RViz2. Declare the relevant runtime dependencies in `package.xml`, including `launch`, `launch_ros`, `xacro`, `robot_state_publisher`, `joint_state_publisher_gui`, `rviz2`, and `ament_index_python` if imported. Install `launch/`, rebuild, re-source, and run `ros2 launch robot_description display.launch.py`. | Three processes start. `ros2 node list` and `ros2 topic echo /joint_states --once` reveal the live graph. If using a launch `Command` substitution for Xacro, pass its result as a string-valued parameter. XML parsing as a YAML parameter is a launch-layer issue. |
| **15** | Set RViz fixed frame to `base_footprint`. Add `RobotModel`, `TF`, and `Grid`. Inspect visual and collision geometry separately. Move each slider. Run `ros2 run tf2_ros tf2_echo base_link left_wheel_link` and the equivalent command for `right_wheel_link`. Save `"$D/rviz/display.rviz"`, add `rviz/` to CMake installation, and launch with this configuration. | Both wheels rotate around their centers. Translations have opposite Y signs. The support and chassis stay fixed. If a wheel revolves around the chassis, its joint origin and visual origin have been confused. |
| **16** | Rebuild and launch once from a fresh runtime terminal. Run `ros2 param get /robot_state_publisher use_sim_time`, `ros2 topic info /joint_states -v`, and `(cd /tmp && ros2 run tf2_tools view_frames)`. Add the verified tree, dimensions, and launch command to `README.md`. Review `git diff --check` and `git status --short` before your own milestone commit. | Display mode uses wall time, exactly one intended node publishes joint states, both wheel transforms resolve, and the saved RViz configuration loads from the install tree. **This is the point to start the passive Gazebo world.** |

### Project practices worth adopting from the first milestone

- **Use Python where it improves iteration.** For this project that means launch files, offline kinematics exercises, evaluation, detector adapters, and mission logic. Reuse the C++ implementations in state publishing, `ros2_control`, SLAM Toolbox, and Nav2. Move your own hot path to C++ only after you measure it.
- **Keep parameters with their owning package.** Controller geometry belongs with control integration; sensor rate with the simulated sensor; detector threshold with perception; site poses with the mission. Where the same physical number must appear in two formats, derive it from one source or add a test comparing generated URDF and controller YAML.
- **Treat installed resources as the public interface.** CMake packages install resource directories under `share/${PROJECT_NAME}`. Python packages must include launch/config assets in `setup.py` `data_files` and register executables under `console_scripts`. A relative source path that works only from the repository root is a reproducibility defect.
- **Use diagnostics in physical order.** For a silent sensor, follow **Gazebo measurement → Gazebo topic → bridge → ROS endpoint/QoS → header → TF at acquisition → consumer**. For a stationary robot, follow **action or teleop → stamped velocity → active controller → claimed wheel interfaces → joint feedback → ground motion**. This is usually faster than tuning parameters.
- **Keep an experiment record in each milestone.** Record the hypothesis, world/map version, command, expected observation, actual observation, and one unresolved limitation. Use ROS log levels for state transitions and failures. Avoid logging every camera frame in the normal path.
- **Keep simulator truth for evaluation.** It is excellent for measuring drift, localization error, and collision clearance. Feeding it into the operational estimator or using it to identify a mission target would invalidate the corresponding experiment.

### Optional extensions after the full project

| Extension | Prerequisites and added complexity | Educational payoff |
|---|---|---|
| Semantic mapping | Reliable detections, metric object positions, time-aligned TF, repeated-observation association, and uncertainty handling | Learn how uncertain observations become persistent map entities. |
| Visual SLAM | Camera calibration, textured environments, feature tracking, motion initialization, scale handling for monocular cameras, and independent truth | Compare visual and LiDAR failure modes on the same route. |
| RGB-D or LiDAR-camera fusion | Extrinsic calibration, synchronization, registration, occlusion handling, and metric standoff validation | Progress from recognizing a target to approaching a previously unknown target location. |
| Reinforcement learning or learned local navigation | Strong classical baseline, reproducible resets, reward design, many trials, and held-out obstacle layouts | Quantify sample efficiency and generalization against Nav2. |
| Advanced planning | Fixed scenarios, cost definitions, measured footprint, and comparable goal tolerances | Separate path length, clearance, smoothness, and execution success. |
| Physical robot | Motor drivers, real encoders, calibrated dimensions, power, hardware interfaces, independent stop mechanism, and physical sensor calibration | Discover the simulation assumptions that matter most in real motion. |

The first portfolio-worthy result is already well defined: **one command launches your own installed robot description, and you can account for every body, joint, transform, and millimetre of clearance visible in RViz.** That gives you a sound reference for every later simulation and autonomy experiment.