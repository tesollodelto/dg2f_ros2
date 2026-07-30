# DG2F ROS 2

![ROS 2 Humble](https://img.shields.io/badge/ROS_2-Humble-blue?logo=ros)
![ROS 2 Jazzy](https://img.shields.io/badge/ROS_2-Jazzy-blue?logo=ros)
![ROS 2 Lyrical](https://img.shields.io/badge/ROS_2-Lyrical-orange?logo=ros)

ROS 2 packages for the **Delto Gripper DG-2F-M** (2-finger, 6 actuated DOF).

## Packages

| Package | Description |
|---|---|
| `dg2f_description` | URDF model, meshes, and RViz display launch |
| `dg2f_driver` | ros2_control hardware driver, controller configs, and launch files |

## Dependencies

This repository needs two shared packages. Note that the **package** names differ
from the **repository** names:

| Package (colcon) | Repository (git) | Description |
|---|---|---|
| `delto_hardware` | [`dg_hardware`](https://github.com/tesollodelto/dg_hardware) | Unified ros2_control hardware interface (model `0x2F02`) |
| `delto_tcp_comm` | [`dg_tcp_comm`](https://github.com/tesollodelto/dg_tcp_comm) | TCP communication library |

Where those packages live depends on how you cloned, so pick **one** of the two
layouts below. Do not mix them: two copies of `delto_hardware` in one workspace
makes `colcon` fail with a duplicate package name.

### Layout A — via the `tesollo_ros2` umbrella repo (recommended)

The shared packages are already included as submodules under `dg_common/`, so
**do not clone them separately**.

```bash
mkdir -p ~/ros2_ws/src && cd ~/ros2_ws/src
git clone https://github.com/tesollodelto/tesollo_ros2.git
cd tesollo_ros2
git submodule update --init dg2f_ros2 dg_common/dg_tcp_comm dg_common/dg_hardware
```

```text
~/ros2_ws/src/
└── tesollo_ros2/
    ├── dg2f_ros2/
    └── dg_common/
        ├── dg_tcp_comm/     # package delto_tcp_comm
        └── dg_hardware/     # package delto_hardware
```

### Layout B — this repository standalone

```bash
mkdir -p ~/ros2_ws/src && cd ~/ros2_ws/src
git clone https://github.com/tesollodelto/dg2f_ros2.git
git clone https://github.com/tesollodelto/dg_hardware.git
git clone https://github.com/tesollodelto/dg_tcp_comm.git
```

```text
~/ros2_ws/src/
├── dg2f_ros2/
├── dg_hardware/            # package delto_hardware
└── dg_tcp_comm/            # package delto_tcp_comm
```

## Build

Identical for both layouts — `--packages-up-to` resolves `delto_tcp_comm` and
`delto_hardware` from wherever they are in `src/`:

```bash
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install --packages-up-to dg2f_description dg2f_driver
source install/setup.bash
```

> **Do not** use `--packages-select dg2f_description dg2f_driver`. It skips `delto_hardware` and
> `delto_tcp_comm`, and the build then fails at `find_package(delto_hardware)`.

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
