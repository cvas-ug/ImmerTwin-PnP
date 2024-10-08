# ImmerTwin PnP

Master package for ImmerTwin, a modular and plug and play teleoperation system inside of a digital Twin, that can be used with any kind of robotic arms. This repository contains both Baxter and UR Robot.

![docs/images/Immertwin.png](docs/images/Immertwin.png)

## Paper Abstract

This repository is published along with a paper [IMMERTWIN: A Mixed Reality Framework for Enhanced Robotic Arm Teleoperation](https://arxiv.org/abs/2409.08964). Please cite it if you use part of this package.

## Needed packages

- Omniverse Isaac Sim
- Unreal Engine 5.4
- SteamVR
- ROS1 Noetic
- ROS2 Galactic (May work on other version but has not been tested)

## Installation

This repository contains multiple submodules of the needed pakckages. Please use:

`git clone --recurse-submodules git@github.com:cvas-ug/ImmerTwin-PnP.git`

to download all of them

Navigate to the ROS1 folder and use catkin_make to build the workspace. Make sure you have sourced ROS Noetic before doing so.

Navigate to the ROS2 folder and use colcon build to build the workspace. Make sure you have sourced ROS Galactic before doing so. ROS noetic needs to be installed to build any of the baxter packages but does not need to be sourced

Source the local workspace and ROS2 for all the following steps except Isaac Sim.

## How to Use

More detailed version of the each part of setup can be found on the individual repositories.

Make sure to run `ImmerTwin/run_discovery_service.sh` as it is required to have ROS2 communicates with Unreal Engine. Make sure for all the following terminal to run `source ImmerTwin/fastdds_setup.sh` except for Isaac Sim

### [Unreal Engine](https://github.com/09ubberboy90/ImmerTwin/tree/main)

Launch the project inside of Unreal Engine and press Run In VR once everything is configured

### Cameras

In order to view the state of the world the ZED2 camera node needs to be run. This can be achieved using

`ros2 launch zed_wrapper zed_camera.launch.py camera_model:={model} node_name:={node_name} serial_number:={serial_number}`

### [Isaac Sim](https://github.com/09ubberboy90/telesim_isaac)

BEWARE: Do not source ROS or have ROS sourced for the following !

Find your isaac sim python path; It should be in `~/.local/share/ov/pkg/isaac_sim-2022.2.1/python.sh`. It will be referred henceforth as `isp`

Then you need to export the path of the packages you are going to use so that Isaac Sim can load them.

`export ROS_PACKAGE_PATH=ROS_PACKAGE_PATH:/opt/ros/galactic/share`

Note: This will only load the packages installed through APT not the local packages. You need to add them manually (See below for example)

You also need to run `export FASTRTPS_DEFAULT_PROFILES_FILE='ImmerTwin/fastdds_config.xml'`

#### First Time Usage

Make sure you also have installed pyquaternion for Isaac Sim. This can be done by running:

`isp -m pip install pyquaternion`

Make sure you have updated the urdf and rmp path according to your need in either [ur_world.py](ur3/ur_world.py) or [baxter_world.py](baxter/baxter_world.py) file. They are defined in the init as `self.urdf` and `self.rmp` respectively

#### Baxter

To add the packages needed for Baxter:

`export ROS_PACKAGE_PATH=ROS_PACKAGE_PATH:{ros_ws}/install/rethink_ee_description/share:{ros_ws}/install/baxter_description/share`

To run for baxter

`isp ROS2/src/isaac_sim/baxter/baxter_world.py`

#### UR

To add the packages needed for UR3 with the T42 gripper:

`export ROS_PACKAGE_PATH=ROS_PACKAGE_PATH:{ros_ws}/install/t42_gripper_description/share`

To run for the UR3

`isp ROS2/src/isaac_sim/ur/ur_world.py`

## Real robot control

### UR

Make sure you have installed [ROS2 driver](https://github.com/UniversalRobots/Universal_Robots_ROS2_Driver) for UR and followed the instructions there.

Once this is done you can connect to the real robot by running:

`ros2 launch ur_bringup ur_control.launch.py ur_type:={ur_type} robot_ip:={robot_ip} launch_rviz:=true`

Note: `ur_type` needs to be ur3 for this package. But additional type of UR robot can be used by creating another package similar to [this](https://github.com/09ubberboy90/ur_robotiq) for the UR5 and Robotiq Gripper. More instruction on how to add new robots to the system is available [here](https://github.com/09ubberboy90/telesim_isaac/blob/master/README.md#adding-new-robots).

In another terminal run:

`ros2 control switch_controllers --start forward_position_controller --stop scaled_joint_trajectory_controller`

And make sure you have URCap setup on the real robot.

Once this is done you can start transmitting joint position from isaac sim to the robot using:

`ros2 run ur_isaac joint_controller`

### Baxter

Baxter needs to have some terminal using ROS1 and ROS2

#### ROS1

Run the following command

``` sh
source ROS1/devel/setup.zsh
export ROS_IP="{Your PC IP}"
export ROS_MASTER_URI="{Baxter IP}"
rosrun baxter_tools enable_robot.py -e
```

#### ROS2

Run the following command

```sh
export ROS_IP="{Your PC IP}"
export ROS_MASTER_URI="{Baxter IP}"
ros2 launch baxter_bridge baxter_bridge_launch.py
```

and in another terminal

`ros run baxter_joint_controller controller`
