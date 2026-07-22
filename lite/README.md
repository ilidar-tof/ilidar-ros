# iTFS-LITE ROS 1 Package

ROS 1 receive-only bridge for the `iTFS::LITE` C++ API.

Version: `V2.0.1`

## Compatibility

|Sensor|Minimum firmware|ROS package|
|:---|:---:|:---:|
|iTFS-LITE|V1.0.0|`ilidar_lite_ros`|

|Ubuntu|ROS 1|C++ baseline|Validation|
|:---:|:---:|:---:|:---|
|16.04|Kinetic|C++14|Build verified|
|18.04|Melodic|C++14|Build verified|
|20.04|Noetic|C++14|Build and sensor runtime verified|

> [!WARNING]
> Kinetic, Melodic, and Noetic have reached end of life. This package is
> maintained for existing ROS 1 deployments.

## Quick Start

From a catkin workspace containing this package:

```bash
source /opt/ros/noetic/setup.bash
rosdep install --from-paths src --ignore-src -r -y
catkin_make -DCMAKE_BUILD_TYPE=Release
source devel/setup.bash
roslaunch ilidar_lite_ros viewer.launch
```

The viewer discovers `/ilidar_lite_<SN>/info` topics and creates one RViz group
per detected serial number. Closing RViz also stops the included driver.

## Dependencies

Dependencies are declared in `package.xml`. The driver uses `roscpp`, standard
ROS messages, `tf2_ros`, and pthreads. The viewer additionally uses `rospy`,
RViz, and PyYAML. OpenCV, PCL, and Open3D are not required.

## Run and Verify

```bash
# Driver only
roslaunch ilidar_lite_ros lite.launch

# Driver and RViz with automatic discovery
roslaunch ilidar_lite_ros viewer.launch

# Create known RViz groups immediately
roslaunch ilidar_lite_ros viewer.launch sensor_sns:="87,88,101"
```

An explicit `sensor_sns` list configures RViz only; it does not filter devices
accepted by the driver. Use `fixed_frame:=<frame>` when the sensors use a
parent other than `base_link`.

Replace `87` with the detected decimal serial number:

```bash
rosnode list
rostopic list | grep ilidar_lite
rostopic echo -n 1 /ilidar_lite_87/info
rostopic hz /ilidar_lite_87/depth/image_raw
rostopic echo -n 1 /diagnostics
```

## Recommended Sensor Output

|Data|Recommended input|ROS output|Reason|
|:---|:---|:---|:---|
|Depth / XYZ-Z|`DEPTH_MM_16` or `XYZ_MM_16`|`mono16` millimeters|No depth restoration|
|Amplitude|`RAW_16`|`mono16` raw scale|No lossy restoration|
|Intensity|`RAW_16`|`mono16` raw scale|No lossy restoration|
|Confidence|`MASK_1`|`mono8`, `0`/`255`|Matches the binary contract|

Other supported modes are converted to the same ROS outputs but may require
extra work or may have lost sensor-side precision. A stream set to `OFF`
produces no corresponding messages.

## Topics and Data Contract

Each device publishes below `/ilidar_lite_<SN>`.

|Topic|Type|Description|
|:---|:---|:---|
|`depth/image_raw`|`sensor_msgs/Image`|Restored depth or XYZ-Z|
|`depth/camera_info`|`sensor_msgs/CameraInfo`|Auxiliary LITE intrinsics|
|`amplitude/image_raw`|`sensor_msgs/Image`|Restored amplitude|
|`intensity/image_raw`|`sensor_msgs/Image`|Restored intensity|
|`confidence/image_raw`|`sensor_msgs/Image`|Binary validity mask|
|`points`|`sensor_msgs/PointCloud2`|Organized XYZ with optional amplitude|
|`info`|`std_msgs/String`|Latched sensor summary|
|`/diagnostics`|`diagnostic_msgs/DiagnosticArray`|Aggregated sensor status|

|Output|Encoding / fields|Unit and invalid-value rule|
|:---|:---|:---|
|Depth|`mono16`|Unsigned millimeters; zero means invalid/no range|
|Amplitude|`mono16`|Restored sensor raw scale|
|Intensity|`mono16`|Restored sensor raw scale|
|Confidence|`mono8`|`0` invalid, `255` valid|
|Point cloud|Organized `x/y/z` `float32`|Meters; invalid points are `NaN`|
|Optional point field|`amplitude` `float32`|Same scale as the amplitude image|

All images are `320 x 240`. Products from one completed frame share one host
ROS timestamp. Image and point-cloud conversion is subscriber-driven.

## Frames and CameraInfo

```text
base_link
  -> ilidar_lite_<SN>_link
      -> ilidar_lite_<SN>_optical_frame
```

Images and CameraInfo use the optical frame; PointCloud2 uses the link frame.
Translations are meters and rotations are roll/pitch/yaw radians. Set
`publish_tf: false` when another component owns the transforms.

CameraInfo is published after the SDK reconstruction map becomes valid. It
uses ROS `plumb_bob`; a nonzero SDK `k4*r^8` coefficient cannot be represented
and produces a warning. Use PointCloud2 when exact SDK 3D reconstruction is
required.

## Parameters

|Parameter|Default|Description|
|:---|:---|:---|
|`broadcast_ip`|empty|SDK broadcast-interface override|
|`listening_ip`|empty|SDK receive-interface override|
|`listening_port`|`7256`|UDP receive port|
|`parent_frame_id`|`base_link`|Mount parent frame|
|`link_frame_id`|SN-derived|Point-cloud frame|
|`frame_id`|SN-derived|Image optical frame|
|`publish_tf`|`true`|Publish static transforms|
|`publish_depth`|`true`|Publish depth|
|`publish_amplitude`|`true`|Publish amplitude when available|
|`publish_intensity`|`true`|Publish intensity when available|
|`publish_confidence`|`true`|Publish confidence when available|
|`publish_camera_info`|`true`|Publish CameraInfo when valid|
|`publish_pointcloud`|`true`|Publish organized points|
|`publish_pointcloud_amplitude`|`true`|Add optional amplitude field|
|`publish_info`|`true`|Publish latched sensor summary|
|`publish_diagnostics`|`true`|Publish diagnostics|
|`sensor_qos_depth`|`5`|ROS publisher queue size|
|`diagnostics_qos_depth`|`10`|Diagnostics queue size|

Pose parameters use `mount_translation_{x,y,z}`,
`mount_rotation_{roll,pitch,yaw}`, `optical_translation_{x,y,z}`, and
`optical_rotation_{roll,pitch,yaw}`. Values are fixed at startup.

## Configuration Examples

Single sensor with a measured mount pose:

```yaml
parent_frame_id: base_link
devices:
  ilidar_lite_87:
    mount_translation_x: 0.25
    mount_translation_z: 0.80
    mount_rotation_yaw: 0.0
```

Multiple sensors can override poses and outputs independently:

```yaml
devices:
  ilidar_lite_87:
    mount_translation_y: 0.15
  ilidar_lite_88:
    mount_translation_y: -0.15
    publish_amplitude: false
```

Disable package-owned TF when transforms come from another component:

```yaml
publish_tf: false
```

Pass an external configuration file with:

```bash
roslaunch ilidar_lite_ros lite.launch config:=/absolute/path/lite.yaml
```

Restart the driver after changing YAML, sensor `data_output`, or other sensor
layout values.

## Diagnostics and Performance

Diagnostics include identity, input modes, sensor warnings, temperature,
power, packet error counters, conversion warnings, frame IDs, and
`ros_frame_overwrite_count`.

- `OK`: no active warning and native ROS input modes are used.
- `WARN`: sensor/frame warning, missing rows, or non-native input conversion.
- `ERROR`: sensor identity or `data_output` changed after initialization.

Each sensor has one worker and reusable latest-frame buffers. New input
replaces pending work when conversion cannot keep up, increasing
`ros_frame_overwrite_count`. Disable unused publishers, subscribe only to
required products, and prefer the recommended sensor modes to reduce load.

## Limitations and Troubleshooting

- Sensor configuration and maintenance operations remain external.
- `data_output`, identity, and ROS parameters cannot change during a run.
- Viewer auto-discovery does not add sensors after RViz has started.
- The latest-frame policy is not suitable for lossless recording.

If no sensor topics appear, verify the sensor stream, IPv4 configuration, UDP
port `7256`, firewall, and interface overrides. If a topic is silent, confirm
that it has a subscriber and the corresponding sensor stream is not `OFF`.
For RViz errors, confirm the fixed frame and unique per-sensor child frames.

## Changelog

See the repository [CHANGELOG.md](../CHANGELOG.md) for release history.

## License

This package and its included SDK source are licensed under the MIT License.
See [LICENSE](LICENSE). Third-party ROS and system dependencies retain their
own licenses.
