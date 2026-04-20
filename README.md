## ARGoS Install Guidelines

0. Clone this repo
1. This project requires argos3 (https://github.com/ilpincy/argos3) to be installed.
	I tested it will the current newest version, which is:
	```bash
	commit 4bb398cd6bfdd09fc919f24c9cccbff38370849b (origin/master, origin/HEAD, master)
	Author: Carlo Pinciroli <carlo@pinciroli.net>
	Date:   Thu Feb 12 17:22:55 2026 -0500

	Added flag `-S` to `env` shebang in uninstall script
	```

	Clone, checkout the right version, and create a branch for it:

	```bash
	git clone https://github.com/ilpincy/argos3
	cd argos3
	git checkout 4bb398cd6bfdd09fc919f24c9cccbff38370849b -b ros2_bridge
	```

2. Before compiling and installing argos, it is highly recommended to delete old version of argos installation from your system. To do that, check /usr/local, which is the default place where argos installs itself. To check:
	```bash
	cd /usr/local
	find . -name "*argos*"
	```

	All the files that contains the name argos will be listed. **Check carefully** to delete only argos files but not other system files. Usually, something like:
	```bash
	rm -rf */argos3
	```
	would do, but again, **check carefully**.

	After removing old version of argos installation, and before compiling our version, you may want to apply some patches for argos3 from folder `argos3_patch`, depends on what do you need. Currently, `DroneVelocityGlobalControl.patch` is essential. It exposes the velocity control and the global position control of the drones. I know global position sounds kind of cheating for swarm, you can of course disable it later and use velocity mode. I have already implemented velocity mode, after this patch, you can simply use velocity mode by adding global_mode="true" in the drone controller in the argos3 configuration file.

	For now, simply:
	```bash
	cd argos3
	git apply ../argos_drone_ros2_bridge/argos_patch/*.patch
	git commit -m "Apply the patches for ros2_bridge"
	```
	or apply them one by one
	```bash
	cd argos3
	git apply ../argos_drone_ros2_bridge/argos_patch/xxx.patch
	git commit -m "Apply the patch for xxx"
	```

3. To compile and install argos, follow the instructions of argos. Here is a guideline, you may need to change some details based on your platform and system setup.
	```bash
	cd argos3
	mkdir build
	cd build
	cmake -DARGOS_BUILD_NATIVE=ON \
	      -DARGOS_DOCUMENTATION=OFF \
	      ../src
	make -j 32
	sudo make install
	```

## ROS2 Bridge Package Install Guidelines
Simply do standard ros2 procedure
```bash
cd argos_drone_ros2_bridge
source /opt/ros/iron/setup.bash
colcon build
source install/local_setup.bash
```

## Commands

to run : 
```bash
ros2 launch drone drone_launch.xml
```
Press play button.
There will be drones fly towards the default target position (0, 0, 0), and they will overlap each other.

to give a command to a drone :
```bash
ros2 topic pub /drone1/dronePoseActuator geometry_msgs/msg/Pose "{position: {x: 1.0, y: 2.0, z: 3.0}, orientation: {x: 0.0, y: 0.0, z: 3.14, w: 0.0}}" --once
```

position x y z are the target position
orientation z is the target yaw

## Explanation

In `src/argos3-bridge` is a ros2 package which will compile an argos3 drone controller with ros2 stuff in it.
The generated .so file is in `install/argos3-bridge/lib/argos3_plugin/libdrone_ros2_bridge_controller.so`
It subscribes to the topic `dronePoseActuator` and give actuator commands to argos3 
It reads from argos3 sensor and publish to `dronePoseSensor`.

In `src/drone` is everything else to create a scenario. In `src/drone/src/droneFlightSystem.cpp` is a ros2 node which subscribes to `dronePoseSensor` and prints the drone's location. In the future you can make it publish to `dronePoseActuator` to make the drone fly automatically.

`src/drone/scripts/configuration.in.argos` is the argos configuration file, during colcon build, a `configuration.argos` will be generated under drone package in build folder.

`src/drone/launch/drone_launch.xml` is where everything come together.
It will call argos3 -c build/xxxx/configuration.argos
It will start two drone nodes in different namespaces.
argos3 and these two drone nodes will communicate with each other through ros2 topics.