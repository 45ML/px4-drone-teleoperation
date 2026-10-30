# System architecture and operator controls

## Data flow

```text
Keyboard ─┐
          ├─> ROS 2 teleoperation node ─> /offboard_velocity_cmd ─┐
Joystick ─┘                                                      │
                                                                 v
                                                     Offboard velocity controller
                                                                 │
                                      /fmu/in/offboard_control_mode
                                      /fmu/in/trajectory_setpoint
                                      /fmu/in/vehicle_command
                                                                 │
                                                                 v
                                                        PX4 SITL (x500)
                                                                 │
                                                           Gazebo Sim

PX4 SITL <── Micro XRCE-DDS Agent (UDP 8888) ──> ROS 2
QGroundControl monitors the SITL vehicle over MAVLink.
```

Keyboard and joystick are alternative front-ends. They publish to a shared command topic; the offboard controller handles PX4 message publication.

## Joystick mapping

The project materials describe this intended mapping. Actual axis and button indices depend on the game controller and Linux joystick driver; inspect `joy_config.yaml` and `teleop_joystick.py`.

| Input | Intended action |
|---|---|
| Left stick left/right | Yaw |
| Left stick up/down | Climb/descent |
| Right stick up/down | Forward/backward |
| Right stick left/right | Lateral movement |
| A | Arm |
| X | Toggle auto-hover |
| Y | Disarm/emergency stop |

## Keyboard mapping

| Key | Intended action |
|---|---|
| Space | Toggle arm/disarm |
| W / S | Climb/descent throttle |
| Arrow keys | Forward/backward/left/right |
| A / D | Yaw left/right |
| H | Request hover / neutral motion |

## Offboard start sequence

The controller streams initial setpoints before requesting offboard mode and arming. PX4 requires a continuing setpoint stream while in offboard mode. Loss of that stream can cause PX4 to exit offboard mode; consult the documentation for the installed firmware release.

## Control limits

The project materials describe controller-side velocity limits, a 50 Hz setpoint stream and no-input hover behaviour. Confirm the actual values and timeout in `velocity_offboard_control.py` before relying on or modifying them.
