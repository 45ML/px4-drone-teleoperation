# PX4 Drone Teleoperation with ROS 2 and Gazebo

A ROS 2 Humble workspace project for manually piloting a PX4 Software-in-the-Loop (SITL) quadcopter in Gazebo Sim. The workspace separates operator input from the PX4 offboard controller and provides keyboard and game-controller interfaces.

> **Scope:** simulation-focused project. This repository does not claim real-aircraft validation, autonomous navigation, obstacle avoidance, or a certified safety system.

## Components

- **px4_bringup** — launches PX4 SITL with the Gazebo `gz_x500` model and starts the Micro XRCE-DDS Agent.
- **px4_offboard** — ROS 2 nodes for offboard velocity control, keyboard teleoperation, and joystick teleoperation.
- **docs/** — architecture, operator mapping, and a repeatable validation checklist.

Both teleoperation front-ends publish to `/offboard_velocity_cmd`; the shared controller translates those requests into PX4 ROS 2 messages.

## Target environment

- Ubuntu 22.04 LTS
- ROS 2 Humble
- PX4 Autopilot SITL
- Gazebo Sim with the `gz_x500` model
- Micro XRCE-DDS Agent
- PX4-compatible `px4_msgs`
- QGroundControl, colcon and rosdep

Keep PX4 and `px4_msgs` versions compatible. Record the exact versions used for each tested run.

## Workspace layout

```text
ros2_ws/
└── src/
    ├── px4_bringup/
    ├── px4_offboard/
    └── px4_msgs/       # install separately; match PX4 version
```

## Setup

### 1. Source ROS 2 and install build tools

```bash
source /opt/ros/humble/setup.bash
sudo apt update
sudo apt install -y python3-colcon-common-extensions python3-rosdep
```

Initialize rosdep if this machine has not used it before:

```bash
sudo rosdep init
rosdep update
```

### 2. Install PX4 SITL and dependencies

Follow the official PX4 Ubuntu setup instructions. The supplied bring-up launch file expects the PX4 source checkout at `~/PX4-Autopilot` and starts the `gz_x500` SITL target.

### 3. Install the Micro XRCE-DDS Agent

Install the agent using its official instructions. The bring-up launch file invokes:

```bash
MicroXRCEAgent udp4 -p 8888
```

Ensure UDP port 8888 is not already occupied by another agent.

### 4. Add PX4 message definitions

Place a `px4_msgs` package compatible with the installed PX4 version in `ros2_ws/src/`.

### 5. Build the workspace

From `ros2_ws`:

```bash
source /opt/ros/humble/setup.bash
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash
```

## Run the simulation

### Terminal 1 — PX4, Gazebo and DDS bridge

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 launch px4_bringup minimal.launch.py
```

The current launch implementation opens GNOME Terminal windows and assumes the PX4 checkout is at `~/PX4-Autopilot`.

### Terminal 2 — choose one input interface

Keyboard:

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 launch px4_offboard keyboard_teleop.launch.py
```

Joystick:

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 launch px4_offboard joystick_teleop.launch.py
```

Run QGroundControl separately if you want to monitor SITL telemetry.

## ROS 2 / PX4 interfaces

| Interface | Purpose |
|---|---|
| `/offboard_velocity_cmd` (`geometry_msgs/Twist`) | Velocity request from a teleoperation front-end |
| `/arm_message` (`std_msgs/Bool`) | Arm/disarm request |
| `/fmu/in/offboard_control_mode` | PX4 offboard heartbeat |
| `/fmu/in/trajectory_setpoint` | Motion setpoints |
| `/fmu/in/vehicle_command` | PX4 mode and arming commands |
| `/fmu/out/vehicle_status_v1` | Vehicle status feedback |
| `/fmu/out/vehicle_attitude` | Attitude feedback |

## Verification checklist

1. Confirm Gazebo starts and the X500 vehicle is present.
2. Confirm the XRCE-DDS Agent listens on UDP port 8888.
3. Check that PX4 topics appear: `ros2 topic list | grep /fmu`.
4. Confirm QGroundControl connects to the SITL vehicle.
5. Start only one teleoperation front-end at a time.
6. Verify arming, low-speed movement, neutral/hover behaviour and disarming in simulation.

See [docs/architecture.md](docs/architecture.md) and [docs/testing-notes.md](docs/testing-notes.md).

## Limitations and safety

- This is a Software-in-the-Loop project, not a flight-ready system.
- No obstacle avoidance or geofencing is implemented.
- The bring-up launch assumes a fixed PX4 path and GNOME Terminal.
- Joystick axis/button indices vary by controller; verify them locally.
- Confirm vertical-velocity conventions and `px4_msgs` compatibility for the installed PX4 release.
- PX4 offboard operation requires a continuous setpoint stream; review the failsafe behaviour for your firmware version.

## Project integrity

The repository is an organized ROS 2/PX4 simulation project based on the supplied workspace materials. It does not claim to implement PX4, ROS 2, Gazebo, or a new autopilot. Credit those upstream projects when presenting or extending this work. Only report implementation changes and test results that you personally performed and verified.

## Upstream documentation

- [PX4 Autopilot](https://docs.px4.io/)
- [PX4 ROS 2 interface](https://docs.px4.io/main/en/ros2/)
- [ROS 2 Humble](https://docs.ros.org/en/humble/)
- [Gazebo](https://gazebosim.org/docs)
- [QGroundControl](https://docs.qgroundcontrol.com/)
- [Micro XRCE-DDS Agent](https://github.com/eProsima/Micro-XRCE-DDS-Agent)
