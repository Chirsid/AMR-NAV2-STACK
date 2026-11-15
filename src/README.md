## Overview

Mechabot has a Xacro robot model with wheels, a lidar, an IMU and a camera. The project includes a small-house environment and a QR maze, with launch files that combine simulation, control and higher-level ROS nodes. The source also contains experimental scripts for marker detection, maze solving, docking, battery-triggered behavior and waypoint navigation. These scripts describe implemented approaches; their reliability has not been independently verified.

### Architecture

Gazebo simulates the robot and sensors. ROS–Gazebo bridges provide clock, laser, IMU and camera data. `ros2_control`, joystick and `twist_mux` configurations handle movement; AMCL and Nav2 configurations support localization and navigation. SLAM Toolbox has a separate launch configuration.

| Package | Purpose |
| --- | --- |
| `mechabot_description` | Robot Xacro/URDF, meshes, worlds, models, RViz and Gazebo launch |
| `mechabot_controller` | Controllers, velocity multiplexing and joystick configuration |
| `mechabot_bringup` | Composite house and maze launches |
| `mechabot_mapping` | SLAM configuration and saved maps |
| `mechabot_localization` | AMCL, map server and localization visualization |
| `mechabot_navigation` | Nav2 configuration, behavior tree and waypoint script |
| `mechabot_scripts` | Sensor, marker, maze and docking Python nodes |

The packages use `ament_cmake` and `ament_python`, with Python launch and node code, XML/Xacro, YAML and Gazebo assets. Notable interfaces include `/scan`, `/imu/out`, `/camera/image_raw`, `/clock` and `/cmd_vel`.

## Repository layout

```text
src/
├── README.md
├── media/                       # original recording and frame from footage
├── mechabot_bringup/            # composite launches
├── mechabot_controller/         # ros2_control and joystick
├── mechabot_description/        # robot, worlds, models, meshes and RViz
├── mechabot_localization/       # AMCL
├── mechabot_mapping/            # SLAM and maps
├── mechabot_navigation/         # Nav2
└── mechabot_scripts/            # Python nodes
```

## Requirements and dependencies

Ubuntu with ROS 2 Humble, a working graphical session and compatible Gazebo/ROS–Gazebo integration are needed for the GUI simulation. The package manifests declare ROS dependencies including `ros_gz_sim`, `ros_gz_bridge`, `ign_ros2_control`, `ros2_control`, Nav2, SLAM Toolbox, RViz, Xacro, joystick and `twist_mux`. The code also uses `ros_gz_image`, Python OpenCV, NumPy, `cv_bridge` and TF transformations; not every runtime dependency is declared in the manifests. Check your Gazebo and bridge versions if plugins fail to load. A joystick is needed for the joystick controls.

## Installation and build

Install ROS 2 Humble and the `colcon` and `rosdep` tools for your Ubuntu installation first. Then:

```bash
mkdir -p ~/mechabot_portfolio_ws/src
cd ~/mechabot_portfolio_ws/src
git clone https://github.com/Chirsid/AMR-NAV2-STACK.git
cd ~/mechabot_portfolio_ws
source /opt/ros/humble/setup.bash
rosdep update
rosdep install --from-paths src --ignore-src -r -y --rosdistro humble
colcon build
source install/setup.bash
```

`rosdep` may report missing or unresolved dependencies, particularly those absent from the package manifests; install the corresponding Humble packages before launching. The repository's nested `src` is discovered recursively by colcon.

## Launch

In a sourced terminal, start the small-house simulation, controller, joystick, localization, RViz and navigation:

```bash
ros2 launch mechabot_bringup simulated_robot.launch.py
```

Start the QR maze world, controller, joystick and maze solver:

```bash
ros2 launch mechabot_bringup qr_maze.launch.py
```

For the Gazebo robot and bridges alone, choose `small_house`, `qr_maze` or `empty`:

```bash
ros2 launch mechabot_description gazebo.launch.py world_name:=small_house
```

`mechabot_description/display.launch.py` displays the robot; `mechabot_controller/controller.launch.py` and `joystick.launch.py` start control components; `mechabot_mapping/slam.launch.py`, `mechabot_localization/global_localization.launch.py` and `mechabot_navigation/navigation.launch.py` launch their respective stacks. The house bringup selects localization; its SLAM include is commented out.

## Troubleshooting

- Source both Humble and the workspace in each new terminal. Run `ros2 pkg list | grep mechabot` to check discovery.
- If a package or bridge is missing, inspect the launch output and install the matching ROS/Gazebo dependency. `rosdep` only installs declared dependencies.
- Gazebo and RViz need a working display and graphics drivers. For missing models, inspect `GZ_SIM_RESOURCE_PATH` and the launch output.
- For an immobile robot, check controller startup, joystick availability and velocity topics. For navigation, check the map, TF tree, simulated clock and initial pose.
- The manifests contain placeholder license declarations; redistribution rights for the project and bundled third-party assets require review. No project license is asserted here.

## Future improvements

Document and declare all runtime dependencies and asset licenses; add repeatable automated integration tests, launch validation and configuration for current supported ROS/Gazebo versions; improve package metadata and document experimental behavior and control procedures.
