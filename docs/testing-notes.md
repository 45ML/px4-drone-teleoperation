# Testing notes

## Validation status

The supplied project report describes a successful SITL/Gazebo launch, PX4 topic visibility through the XRCE-DDS bridge, and a QGroundControl connection. Those are project-reported outcomes; this repository has not been executed in the current environment, so they are not independently verified here.

## Repeatable simulation checklist

| Check | Expected observation | Status |
|---|---|---|
| PX4 SITL startup | PX4 SITL starts without build/runtime error | Not independently verified |
| Gazebo startup | X500 vehicle appears | Not independently verified |
| XRCE-DDS Agent | Agent listens on UDP 8888 | Not independently verified |
| ROS 2 bridge | PX4 `/fmu` topics appear | Not independently verified |
| QGroundControl | SITL telemetry is visible | Not independently verified |
| Keyboard input | Commands respond in simulation | Not independently verified |
| Joystick input | Axes/buttons match the selected mapping | Not independently verified |
| Disarm | Vehicle receives disarm request | Not independently verified |

## Items to check

- The bring-up launch assumes `~/PX4-Autopilot` and GNOME Terminal.
- Joystick mapping depends on controller model and driver.
- Verify vertical-velocity sign conventions for the installed PX4 release.
- Match `px4_msgs` definitions to the PX4 firmware version.
- The project materials describe simulation-only validation and no obstacle avoidance.

## Experiment log template

For each run, record:

- Date and machine/software versions
- PX4 commit/tag and `px4_msgs` version
- Controller model and axis/button mapping
- Launch command and terminal output
- Arming/offboard transition outcome
- Commanded movement and observed response
- Failsafe/disarm behaviour
- Screenshots or recordings produced during your own run
