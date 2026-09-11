# ROS 2 Position Controller Task

## Before you begin

This task uses **ROS 2 Humble Hawksbill** on **Ubuntu 22.04 LTS**. In a terminal, navigate to the `ros2_ws` folder and run these commands after every change:

**Platform options:** Windows users are recommended to run Ubuntu through **WSL**. macOS users are recommended to run an Ubuntu virtual machine with **UTM**. Dual booting or another virtual-machine tool is also acceptable. Use the approach that works best on your machine.

### My recommended alternative to the ones above: prepared Docker image

The fastest route is this [preconfigured Docker image](https://drive.google.com/file/d/110nax0nybZmWMKUrlIdFWkl3z_0rv9V7/view?usp=sharing). It already contains the required environment. You only need to install Docker and load the downloaded `.tar` file. This route can often get you running in under an hour on both Mac and Windows, no VM or dual booting is needed.

```bash
docker load -i /path/to/rover.tar
```

Then replace both placeholder paths below with the folder containing your workspace and run:

```bash
docker run --hostname=c44192d8f274 --env=LANG=C.UTF-8 --env=LC_ALL=C.UTF-8 --env=ROS_DISTRO=humble --env=PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin --volume=<PATH TO FOLDER ON YOUR PC>:<PATH INSIDE DOCKER, WHICH WILL BE THE SAME FOLDER AS THE ONE ON YOUR PC> --network=bridge -p 5001:5001/udp -p 6080:6080 -p 8080:8080 --restart=no --label='org.opencontainers.image.version=22.04' --runtime=runc -t -d rover
```

> Setup is BY FAR the hardest part of this task, which is why I recommend using the Docker image I provided you with above, otherwise, if you are new to Ubuntu, expect the initial setup to take 2 hours or more. If you are new to ROS 2, allow at least 30 minutes to get your environment is working. Confirm the provided packages build and run before writing your controller.

```bash
cd ros2_ws
colcon build
source install/setup.bash
```

`colcon build` builds every ROS 2 package in the workspace and places the runnable result in `install/`. Run it again whenever you change package code or configuration.

`source install/setup.bash` makes the packages you just built available to ROS 2. Run it in every new terminal before using `ros2 run` or `ros2 launch`.

See the [Articulated Robotics “Build a Robot” series](https://www.youtube.com/watch?v=Gg25GfA456o) or the [ROS 2 Humble documentation](https://docs.ros.org/en/humble/) if you think you'll need it.

## What is a `Twist`?

`geometry_msgs/Twist` is a ROS 2 message type. It contains `linear` and `angular` vectors, each with (X, Y, Z) values.

For this task, you only need to use `linear.x` and `linear.y` as 2D movement inputs. Ignore `linear.z` and all `angular` values.

## Provided packages

You start with two Python packages:

| Package | Nodes | Purpose |
| --- | --- | --- |
| `command_simulator` | `emergency_stop_publisher`, `random_velocity_publisher` | Publishes e-stop state and random `Twist` movement commands. |
| `position_checker` | `position_checker` | Checks the position published by YOUR node and reports `Correct` or `Incorrect`. |

command_simulator publishes these topics:

| Topic | Type | Meaning |
| --- | --- | --- |
| `/movement_command` | `Twist` | Random movement command. Use only `linear.x` and `linear.y`. |
| `/emergency_stop` | `Bool` | `true` means the e-stop is engaged and movement must be blocked. |

## Your task

Create a package named `position_controller` with one node (the node can have any name).
Even though the provided packages above are in Python, you can make your package in Python or C++, whichever you prefer.

Your node must:

1. Start at position `(0, 0)`.
2. Subscribe to `/movement_command` and `/emergency_stop`.
3. For each movement command, add `linear.x` and `linear.y` to the current X/Y position.
4. Clamp both coordinates to the inclusive range `-10` through `10`.
5. Ignore movement commands while the e-stop is engaged.
6. Publish a `geometry_msgs/Point` on `/position` for every movement command.

Create a Python package with:

```bash
ros2 pkg create position_controller --build-type ament_python --dependencies rclpy geometry_msgs std_msgs
```

Or create a C++ package with:

```bash
ros2 pkg create position_controller --build-type ament_cmake --dependencies rclcpp geometry_msgs std_msgs
```

## Run and test

Build and source the workspace first. Then **start your controller before the checker**:

```bash
ros2 run position_controller <your_node_name>
```

In a second terminal (also sourced), start the simulator and checker:

```bash
ros2 launch position_checker position_demo.launch.py
```

The checker will print `Correct` when your published `/position` matches the expected result, and `Incorrect` otherwise.
