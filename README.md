# Franka-IRL
Franka IRL
# Franka Emika ROS 2 Humble + MoveIt 2 Simulation Setup Guide

This repository provides a step-by-step guide to set up a simulation environment for a Franka Emika robot (tested with configuration for **FP3**, adaptable for **Panda**) using **ROS 2 Humble Hawksbill** and **MoveIt 2**. It covers installation, workspace setup, launching the simulation with RViz, and basic motion planning interaction.

**Target Audience:** Users familiar with Ubuntu Linux and basic terminal commands, aiming to run a Franka Emika simulation with MoveIt 2 on ROS 2 Humble.

![ROS Humble](https://img.shields.io/badge/ROS-Humble-blue)
![Ubuntu 22.04](https://img.shields.io/badge/Ubuntu-22.04-orange)
![MoveIt 2](https://img.shields.io/badge/MoveIt-2-green)

---

## Table of Contents

1.  [Prerequisites](#prerequisites)
2.  [Installation Steps](#installation-steps)
    * [Install ROS 2 Humble](#install-ros-2-humble)
    * [Install MoveIt 2](#install-moveit-2)
    * [Install Build Tools](#install-build-tools)
    * [Create ROS 2 Workspace](#create-ros-2-workspace)
    * [Clone `franka_ros2` Repository](#clone-franka_ros2-repository)
    * [Install Dependencies](#install-dependencies)
    * [(Optional) Install `libfranka`](#optional-install-libfranka)
    * [Build the Workspace](#build-the-workspace)
3.  [Environment Setup (Sourcing)](#environment-setup-sourcing)
4.  [Running the Simulation](#running-the-simulation)
5.  [Basic Usage in RViz](#basic-usage-in-rviz)
6.  [Running with Real Hardware (Brief Overview)](#running-with-real-hardware-brief-overview)
7.  [Troubleshooting](#troubleshooting)
8.  [License](#license)
9.  [Acknowledgements](#acknowledgements)

---

## Prerequisites

Before you begin, ensure you have the following:

* **Operating System:** Ubuntu 22.04 LTS
* **ROS 2 Humble:** ROS 2 Humble Hawksbill **Desktop Install**. If not installed, follow the [Official ROS 2 Humble Installation Guide](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html).
* **Git:** `sudo apt update && sudo apt install git`
* Basic familiarity with the Linux terminal.

---

## Installation Steps

### Install ROS 2 Humble

Ensure you have a working ROS 2 Humble Desktop installation as mentioned in the prerequisites. Remember to source the setup file in your terminal:
```bash
source /opt/ros/humble/setup.bash
```

### Install MoveIt 2

Install the MoveIt 2 binaries for ROS 2 Humble:
```bash
sudo apt update && sudo apt install ros-humble-moveit -y
```

### Install Build Tools

Install `colcon` (the ROS 2 build tool) and `rosdep` (for dependency management):
```bash
sudo apt install python3-colcon-common-extensions python3-rosdep -y
```

Initialize `rosdep` (only needs to be done once):
```bash
sudo rosdep init
rosdep update
```
*(If `sudo rosdep init` reports the file already exists, that's okay, just proceed with `rosdep update`)*

### Create ROS 2 Workspace

Create a dedicated workspace to build the Franka packages:
```bash
mkdir -p ~/ros2_franka/src
cd ~/ros2_franka
```

### Clone `franka_ros2` Repository

Clone the official `franka_ros2` repository into your workspace's `src` directory. We will use the `humble` branch (verify this is the correct/latest stable branch for Humble support on the [franka_ros2 GitHub page](https://github.com/frankaemika/franka_ros2)).
```bash
cd ~/ros2_franka/src
git clone [https://github.com/frankaemika/franka_ros2.git](https://github.com/frankaemika/franka_ros2.git) -b humble
```

### Install Dependencies

Navigate back to the root of your workspace and use `rosdep` to install all required dependencies for the cloned packages:
```bash
cd ~/ros2_franka
rosdep install --from-paths src --ignore-src -r -y
```
This command might take some time as it installs necessary libraries and ROS packages.

### (Optional) Install `libfranka`

`libfranka` is the C++ library required to communicate with the **real Franka Emika hardware**.
* **Not required for simulation.** You can skip this step if you only plan to use RViz simulation or Gazebo.
* If you intend to connect to a real robot later, follow the [Official libfranka Installation Instructions](https://frankaemika.github.io/docs/installation_linux.html).

### Build the Workspace

Build all the packages in your workspace using `colcon`:
```bash
cd ~/ros2_franka
colcon build --symlink-install
```
* Monitor the output for any **errors**. Warnings (like the `CMake Warning` related to `FindBoost` / `python310` encountered earlier) are often acceptable if the build summary shows all packages finished successfully.
* `--symlink-install` is convenient as it allows changes to Python scripts or configuration files without needing a full rebuild.

---

## Environment Setup (Sourcing)

**This is a critical step!** Before running any nodes from your built workspace, you need to make the ROS 2 environment aware of them. You must source the main ROS 2 setup file *first*, followed by your workspace's setup file.

Open a **new terminal** (or ensure previous sourcing is cleared) and run:
```bash
# 1. Source ROS 2 Humble main installation
source /opt/ros/humble/setup.bash

# 2. Source your Franka workspace overlay
source ~/ros2_franka/install/setup.bash
```

**Tip:** To avoid sourcing manually in every new terminal, you can add the workspace sourcing line to your shell's startup script (e.g., `.bashrc`):
```bash
echo "source ~/ros2_franka/install/setup.bash" >> ~/.bashrc
# Remember to also have the main ROS 2 source line in your .bashrc or source it manually first.
```

---

## Running the Simulation

With the environment correctly sourced, you can now launch the MoveIt simulation using RViz with fake controllers.

The specific launch command depends on the MoveIt configuration package name that was built (which depends on the `franka_ros2` version/branch). Based on our previous interaction, `franka_fr3_moveit_config` was identified.

```bash
# Ensure your environment is sourced correctly in this terminal!

ros2 launch franka_fr3_moveit_config moveit.launch.py robot_ip:=dont-care load_gripper:=true use_fake_hardware:=true

```


* **Check Package Name:** If this fails with "Package not found", double-check the exact name of the `_moveit_config` package built in your `~/ros2_franka/install/` directory (e.g., `franka_fr3_moveit_config`).
* **`load_gripper:=true`:** This argument tells the launch file to also load the configuration for the Franka Hand/Gripper. Set to `false` if you don't need the gripper.

If successful, RViz should open, displaying the Franka robot model in its default state.

---

## Basic Usage in RViz
```bash
 Launch the demo for FR3 (adjust package name if needed for Panda)
ros2 launch franka_fr3_moveit_config demo.launch.py load_gripper:=true
```

Once RViz is running with the robot model and the MoveIt `MotionPlanning` panel:

1.  **Select Planning Group:** In the "MotionPlanning" panel -> "Context" tab, choose the `Planning Group` (e.g., `fr3_arm` or `panda_arm`).
2.  **Set Goal State:** Go to the "Planning" tab. Drag the **interactive markers** (orange rings/arrows) on the robot's end-effector in the 3D view to set your desired goal pose. A 'ghost' robot will show the goal.
3.  **Plan:** Click the `Plan` button. MoveIt computes a trajectory, which will be visualized in RViz.
4.  **Execute:** Click the `Execute` button. The main robot model in RViz will move along the planned path, simulating the execution.
5.  **Plan and Execute:** Use this button for a combined action.

You can add obstacles using the "Scene Objects" tab to test collision avoidance.

---

## Running with Real Hardware (Brief Overview)

Connecting to and controlling the physical robot requires additional steps and precautions:

* **`libfranka` Must Be Installed:** See Step 7 in Installation.
* **Real-Time Kernel:** Strongly recommended for stable control performance. See [Franka Control Interface (FCI) documentation](https://frankaemika.github.io/docs/installation_linux.html#setting-up-the-real-time-kernel).
* **Correct IP Address:** You need the actual IP address of your robot's control unit.
* **Different Launch File/Arguments:** You typically use a different launch file (e.g., `moveit.launch.py`) and provide the IP address:
    ```bash
    # Example ONLY - Verify correct launch file and args from franka_ros2 docs
    ros2 launch franka_moveit_config moveit.launch.py robot_ip:=<YOUR_ROBOT_IP> load_gripper:=true use_fake_hardware:=false
    ```
* **Robot State:** Ensure the robot is unlocked via the Franka Desk interface and ready for external control.
* **Safety:** Real robot arms can move unexpectedly. Ensure a safe workspace, understand the E-stop, and **proceed with caution**.

**Refer to the official `franka_ros2` documentation for detailed instructions on hardware operation.**

---

## Troubleshooting

* **`Package ... not found`:**
    * Did `colcon build` complete successfully? Check the build logs.
    * Did you source the environment **correctly** (`/opt/ros/...` first, then `~/ros2_franka/...`) in the *current* terminal?
    * Are you using the correct package name in your `ros2 launch` command? Verify the name in `~/ros2_franka/install/`.
* **`libfranka: Connection error: DNS error` (or similar connection errors during launch):**
    * You are likely launching a configuration intended for real hardware (`moveit.launch.py`?) with `robot_ip:=dont-care` or an invalid IP.
    * **Solution:** Ensure you are launching the correct file for simulation (e.g., `demo.launch.py`) or using the correct arguments (like `use_fake_hardware:=true`, if available) to explicitly request simulation mode.
* **Build Errors:**
    * Run `rosdep install ...` again (Step 6) to ensure all dependencies are met.
    * Check the specific error messages from `colcon build`. Often indicates missing libraries or incorrect compiler versions. Search online for the specific error.
* **RViz Issues (Model not loaded, Frame errors):**
    * Check the terminal output where you ran `ros2 launch` for errors from `robot_state_publisher` or TF.
    * In RViz -> "Global Options", ensure the `Fixed Frame` is set correctly (e.g., `world`, `panda_link0`, `fr3_link0`).
    * Ensure all required nodes started correctly (check terminal logs).

---

## License

Consider adding a license file (e.g., `LICENSE`) to your repository. Common choices for ROS projects include Apache 2.0 or MIT License. State the chosen license here.

Example: This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Acknowledgements

* [Franka Emika](https://www.franka.de/)
* [franka_ros2 Developers](https://github.com/frankaemika/franka_ros2)
* [MoveIt Maintainers](https://moveit.ros.org/)
* [ROS 2 Team (Open Robotics / Intrinsic)](https://ros.org/)
