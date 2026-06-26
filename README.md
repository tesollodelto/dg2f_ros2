# DG2F ROS 2

![ROS 2 Humble](https://img.shields.io/badge/ROS_2-Humble-blue?logo=ros)
![ROS 2 Jazzy](https://img.shields.io/badge/ROS_2-Jazzy-blue?logo=ros)

ROS 2 packages for the **Delto Gripper DG-2F-M** (2-finger, 6 actuated DOF).

## Packages

| Package | Description |
|---|---|
| `dg2f_description` | URDF model, meshes, and RViz display launch |
| `dg2f_driver` | ros2_control hardware driver, controller configs, and launch files |

## Dependencies

This repository requires the shared common packages to build:

```bash
# Clone into your ROS 2 workspace src directory
git clone https://github.com/tesollodelto/dg_hardware.git
git clone https://github.com/tesollodelto/dg_tcp_comm.git
```

- [`delto_hardware`](https://github.com/tesollodelto/dg_hardware) — Unified ros2_control hardware interface (model `0x2F02`)
- [`delto_tcp_comm`](https://github.com/tesollodelto/dg_tcp_comm) — TCP communication library

## Build

```bash
cd ~/ros2_ws
colcon build --packages-select dg2f_description dg2f_driver delto_hardware
source install/setup.bash
```

## ⚠️ Before You Control

The `dg2f_driver` (ros2_control) operates in **Developer Mode** over a custom Ethernet protocol.
Set the gripper to **Developer Mode** with the communication mode set to **EtherNET** before launching a hardware driver (check the switch/LED settings in your DG-2F product manual).

> The DG-2F-M is an **M-series** gripper: motor direction is firmware-dependent — `delto_hardware` applies the legacy direction below firmware **v2.8** and the revised direction at v2.8+ (handled automatically).

## Launch

```bash
# RViz display (no device)
ros2 launch dg2f_description dg2f_display.launch.py

# JointTrajectoryController driver
ros2 launch dg2f_driver dg2f_driver.launch.py delto_ip:=169.254.186.72

# Effort / PID (individual) / PID (all-in-one)
ros2 launch dg2f_driver dg2f_effort_controller.launch.py delto_ip:=169.254.186.72
ros2 launch dg2f_driver dg2f_pid_controller.launch.py delto_ip:=169.254.186.72
ros2 launch dg2f_driver dg2f_pid_all_controller.launch.py delto_ip:=169.254.186.72

# Mock hardware (no device)
ros2 launch dg2f_driver dg2f_mock.launch.py
```

See [`dg2f_driver/README.md`](dg2f_driver/README.md) for controller details, topics, and test scripts.

## License
BSD-3-Clause
