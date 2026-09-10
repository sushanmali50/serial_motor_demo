# ROS 2 / Arduino Serial Motor Demo

A ROS 2 interface to differential-drive motor firmware over serial, with a Tkinter GUI for sending commands and viewing encoder feedback.

## Provenance

This repository contains tutorial-derived code with Josh Newans listed in its package metadata. It is retained as a learning/reference integration project. The existing attribution is preserved; the repository does not establish which parts were independently authored by Sushan Mali.

The corresponding firmware is linked by the original project: [Josh Newans' ros_arduino_bridge](https://github.com/joshnewans/ros_arduino_bridge), derived from [HB Robotics' bridge](https://github.com/hbrobotics/ros_arduino_bridge).

## Components

| Package | Role |
| --- | --- |
| `serial_motor_demo` | Python serial driver and Tkinter GUI |
| `serial_motor_demo_msgs` | Motor commands, measured velocities and encoder-count messages |

The driver sends carriage-return-terminated commands: `o` for PWM, `m` for counts per firmware loop, and `e` to read encoders. The Arduino firmware performs the low-level motor control.

## Build

Use a configured ROS 2 environment with `colcon` and `rosdep`, plus Python serial and Tkinter dependencies.

```bash
mkdir -p ~/serial_motor_ws/src
cd ~/serial_motor_ws/src
git clone https://github.com/sushanmali50/serial_motor_demo.git
cd ..
rosdep install --from-paths src --ignore-src -r -y
sudo apt install python3-serial python3-tk
colcon build --symlink-install
source install/setup.bash
```

Flash the compatible external Arduino firmware separately. Match serial permissions, baud rate, encoder counts, and firmware loop rate to your hardware.

## Run

The numeric parameters below are examples from the original README. Replace them with the actual values for your motors and firmware.

```bash
ros2 run serial_motor_demo driver --ros-args -p encoder_cpr:=3440 -p loop_rate:=30 -p serial_port:=/dev/ttyUSB0 -p baud_rate:=57600
```

In a second sourced terminal:

```bash
ros2 run serial_motor_demo gui
```

| Parameter | Meaning |
| --- | --- |
| `encoder_cpr` | Counts per revolution; must be positive |
| `loop_rate` | Arduino control-loop rate; must be positive |
| `serial_port` | Default `/dev/ttyUSB0` |
| `baud_rate` | Default 57600 |
| `serial_debug` | Log transmitted and received commands |

| Topic | Message | Units |
| --- | --- | --- |
| `motor_command` | `MotorCommand` | rad/s, or raw PWM when `is_pwm` is true |
| `motor_vels` | `MotorVels` | rad/s |
| `encoder_vals` | `EncoderVals` | Raw counts |

The GUI displays rev/s and converts to rad/s. Its PWM mode accepts -255 to 255.

## Known limitations

- Default encoder/loop-rate parameters are zero; running unchanged can cause division by zero.
- The original GUI can fail when switching to feedback mode with an empty speed-limit field.
- Serial-response validation, command timeout behavior and shutdown handling need further work.
- Exact tested ROS/firmware versions are not recorded, and hardware execution has not been revalidated during documentation cleanup.

## Original project TODOs

Encoder-reset service, PID-parameter update service, stability improvements, and further parameterization.
