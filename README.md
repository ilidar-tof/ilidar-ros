# iLidar ROS Packages

ROS 1 C++ packages for the **iTFS** and **iTFS-LITE** product families.

Version: `V2.0.1`

> [!WARNING]
> ROS Kinetic, Melodic, and Noetic have reached end of life. These packages are
> maintained for existing ROS 1 deployments.

## Quick Start

Create a catkin workspace and place this repository under its `src` directory:

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
git clone https://github.com/ilidar-tof/ilidar-ros.git

source /opt/ros/noetic/setup.bash
cd ~/catkin_ws
rosdep install --from-paths src --ignore-src -r -y
catkin_make -DCMAKE_BUILD_TYPE=Release
source devel/setup.bash
```

Launch the viewer for the connected product:

```bash
# iTFS
roslaunch ilidar_itfs_ros viewer.launch

# iTFS-LITE
roslaunch ilidar_lite_ros viewer.launch
```

Each viewer discovers connected sensors and creates one RViz group per serial
number. Closing RViz also stops the driver included by `viewer.launch`.

## Package Overview

|Directory|ROS package|Target|Description|
|:---|:---|:---:|:---|
|`itfs/`|`ilidar_itfs_ros`|iTFS|Images, calibrated point cloud, TF, diagnostics, and RViz|
|`lite/`|`ilidar_lite_ros`|iTFS-LITE|Images, CameraInfo, point cloud, TF, diagnostics, and RViz|

### Interfaces

|Package|Data outputs|Status output|Info output|
|:---|:---|:---|:---|
|`ilidar_itfs_ros`|`sensor_msgs/Image`, `sensor_msgs/PointCloud2`|`diagnostic_msgs/DiagnosticArray`|`std_msgs/String`|
|`ilidar_lite_ros`|`sensor_msgs/Image`, `sensor_msgs/CameraInfo`, `sensor_msgs/PointCloud2`|`diagnostic_msgs/DiagnosticArray`|`std_msgs/String`|

Sensor configuration, firmware, flash, and calibration operations are outside
these receive-side ROS packages. Configure the sensor with its dedicated tool
before starting a driver.

## Dependencies and Platform Support

The drivers use `roscpp`, standard ROS message packages, `tf2_ros`, and
pthreads. The viewers additionally require `rospy`, RViz, and PyYAML. OpenCV,
PCL, and Open3D are not required.

|Ubuntu|ROS 1|Compiler baseline|Validation|
|:---:|:---:|:---:|:---|
|16.04|Kinetic|GCC 5 / C++14|Build verified|
|18.04|Melodic|GCC 7 / C++14|Build verified|
|20.04|Noetic|GCC 9 / C++14|Build and sensor runtime verified|

## Run

Source the workspace in every terminal before launching a package:

```bash
source ~/catkin_ws/devel/setup.bash
```

### iTFS

```bash
# Driver only
roslaunch ilidar_itfs_ros itfs.launch

# Driver and RViz with automatic sensor discovery
roslaunch ilidar_itfs_ros viewer.launch

# Create known RViz groups immediately
roslaunch ilidar_itfs_ros viewer.launch sensor_sns:="2405,2406"
```

Verify the connection using the detected serial number:

```bash
rostopic echo -n 1 /ilidar_2405/info
rostopic echo -n 1 /ilidar_2405/depth/image_raw/encoding
```

### iTFS-LITE

```bash
# Driver only
roslaunch ilidar_lite_ros lite.launch

# Driver and RViz with automatic sensor discovery
roslaunch ilidar_lite_ros viewer.launch

# Create known RViz groups immediately
roslaunch ilidar_lite_ros viewer.launch sensor_sns:="87,88,101"
```

Verify the connection using the detected serial number:

```bash
rostopic echo -n 1 /ilidar_lite_87/info
rostopic echo -n 1 /ilidar_lite_87/depth/image_raw/encoding
```

The `sensor_sns` argument creates RViz groups but does not filter devices
accepted by the driver. Use `fixed_frame:=<frame>` when configured sensors do
not share the default `base_link`.

## Product Data Differences

|Item|iTFS|iTFS-LITE|
|:---|:---|:---|
|Topic prefix|`/ilidar_<SN>`|`/ilidar_lite_<SN>`|
|Images|Depth, intensity/gray|Depth, amplitude, intensity, confidence|
|Image encodings|Depth/intensity `mono16`|Depth/amplitude/intensity `mono16`, confidence `mono8`|
|Point field|Optional intensity|Optional amplitude|
|CameraInfo|Not published; mapping uses a direction table|Published for depth|
|Special mode|Capture mode 0 publishes gray as intensity|Output depends on configured `data_output`|

Both packages publish organized XYZ in meters, static TF, sensor information,
and diagnostics.

## License

The ROS wrappers and included SDK source are licensed under the MIT License.
Copyright 2022-Present HYBO Inc. See [LICENSE](LICENSE). Third-party ROS, RViz,
and system dependencies retain their own licenses.
