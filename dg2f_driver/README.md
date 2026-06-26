# dg2f_driver ROS 2 Package 🚀

## 📌 Overview

The `dg2f_driver` ROS 2 package provides a hardware interface leveraging [ros2_control](https://control.ros.org/) for the DG-2F-M gripper (2-finger, 6 actuated DOF), enabling direct robotic control operations.

Actuated joints: `j_1_1, j_1_2, j_1_3, j_2_1, j_2_2, j_2_3` (the `j_X_4` tip joints are fixed/passive).

## 📦 Dependency Installation

```bash
cd ~/your_ws
sudo apt update
rosdep update
rosdep install --from-paths src/tesollo_ros2/dg2f_ros2/dg2f_driver --ignore-src -r -y
colcon build --packages-select dg2f_driver dg2f_description delto_hardware
```

## ⚠️ Before You Control: Notes

The `dg2f_driver` (ros2_control) operates in **Developer Mode**, which uses a custom protocol over Ethernet.
Before launching the driver, set the gripper to **Developer Mode** with the communication mode set to **EtherNET**, and verify the switch/LED settings in your DG-2F-M product manual.

> The DG-2F-M is an **M-series** gripper: motor direction is firmware-dependent. `delto_hardware` applies the legacy direction below firmware **v2.8** and the revised direction at v2.8 or later (handled automatically).

---

## 🚀 Launch Files

| Launch File | Description | Controller Type |
|-------------|-------------|-----------------|
| `dg2f_driver.launch.py` | DG-2F-M - JointTrajectoryController | Position (Trajectory) |
| `dg2f_effort_controller.launch.py` | DG-2F-M - Direct Effort Control | Effort (Direct) |
| `dg2f_pid_controller.launch.py` | DG-2F-M - Individual per-joint PID Controllers | PID (Position→Effort) |
| `dg2f_pid_all_controller.launch.py` | DG-2F-M - Single grouped PID Controller (all joints) | PID (Position→Effort) |
| `dg2f_mock.launch.py` | DG-2F-M - Mock hardware (no device) | Position (Trajectory) |

---

## 🎛️ Controlling DG-2F-M

### 1. JointTrajectoryController (default)
```bash
ros2 launch dg2f_driver dg2f_driver.launch.py delto_ip:=169.254.186.72 delto_port:=502
```

### 2. Effort controller (direct)
```bash
ros2 launch dg2f_driver dg2f_effort_controller.launch.py delto_ip:=169.254.186.72
```

### 3. PID controllers (position reference → effort)
```bash
# Individual: one PidController per joint (/dg2f/<joint>_pospid/reference)
ros2 launch dg2f_driver dg2f_pid_controller.launch.py delto_ip:=169.254.186.72

# All-in-one: a single grouped PidController (/dg2f/j_pospid/reference)
ros2 launch dg2f_driver dg2f_pid_all_controller.launch.py delto_ip:=169.254.186.72
```

### 4. Mock hardware (no device required)
```bash
ros2 launch dg2f_driver dg2f_mock.launch.py
```

### 5. Test scripts

| Script | Controller Type | Description |
|--------|-----------------|-------------|
| `dg2f_jtc_test.py` | JTC (topic) | JointTrajectory topic based test |
| `dg2f_jtc_action_test.py` | JTC (action) | FollowJointTrajectory action based test |
| `dg2f_pid_test.py` | PID (individual) | Publishes a reference to each per-joint `*_pospid` controller |
| `dg2f_pid_all_test.py` | PID (all) | Publishes one reference to the grouped `j_pospid` controller |

```bash
ros2 run dg2f_driver dg2f_jtc_test.py
ros2 run dg2f_driver dg2f_jtc_action_test.py
ros2 run dg2f_driver dg2f_pid_test.py
ros2 run dg2f_driver dg2f_pid_all_test.py
```

---

## 🔧 Controller Types

### 1. JointTrajectoryController (Default)
- **Joints**: 6 (j_1_1, j_1_2, j_1_3, j_2_1, j_2_2, j_2_3)
- **Topic**: `/dg2f/dg2f_controller/joint_trajectory`
- **Action**: `/dg2f/dg2f_controller/follow_joint_trajectory`

### 2. Effort Controller (Direct)
- **Topic**: `/dg2f/effort_controller/commands`

### 3. PID Controllers (Position → Effort)

| Variant | Config | Controllers | Reference Topic |
|---------|--------|-------------|-----------------|
| **Individual** (`pid`) | `dg2f_pid_controller.yaml` | one `pid_controller/PidController` per joint, named `<joint>_pospid` | `/dg2f/<joint>_pospid/reference` |
| **All-in-one** (`pid_all`) | `dg2f_pid_all_controller.yaml` | a single `pid_controller/PidController` named `j_pospid` managing all joints | `/dg2f/j_pospid/reference` |

Both take a `control_msgs/MultiDOFCommand` position reference and output effort. Gains are seeded from the JTC config (`p: 1.5`).

---

## 🌐 Namespace

All DG-2F-M driver nodes use the `/dg2f/` namespace to avoid topic conflicts with other grippers.

## 📄 License
BSD-3-Clause

## 📧 Contact
[TESOLLO SUPPORT](mailto:support@tesollo.com)
