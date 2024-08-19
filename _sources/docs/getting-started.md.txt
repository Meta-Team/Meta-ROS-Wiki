# Getting Started

## Install ROS 2 Humble
Please checkout [ROS documentation](https://docs.ros.org/en/humble/) on how to install ROS 2 Humble.

## Create a ROS workspace and clone
It is generally recommended to create a workspace when working with ROS.
```bash
mkdir -p meta_ws/src
cd meta_ws/src
git clone --recurse-submodules https://github.com/Meta-Team/Meta-ROS
```

## Install all the dependencies
The project dependencies can be easily installed with a single `rosdep` command.
```bash
rosdep install -y --rosdistro humble --from-paths . --ignore-src
```

## Compile the project
```bash
colcon build --symlink-install
```

## Run the sentry simulation
The sentry robot is the baseline robot of this project, you can run a simulation in Gazebo Fortress with the `sentry.launch.py` launch file.

```bash
ros2 launch meta_bringup sentry.launch.py enable_simulation:=true
```

Don't forget to source ROS setup script first
```bash
source install/local_setup.bash
```
