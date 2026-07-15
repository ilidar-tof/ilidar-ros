# iTFS ROS 1 Package

ROS 1 receive-only bridge for the `iTFS::LiDAR` C++ API.

Version: `V2.0.0`

## Compatibility

|Sensor|Minimum firmware|ROS package|
|:---|:---:|:---:|
|iTFS-110|V1.5.0|`ilidar_itfs_ros`|
|iTFS-80|V1.5.0|`ilidar_itfs_ros`|

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
roslaunch ilidar_itfs_ros viewer.launch
```

The viewer discovers `/ilidar_<SN>/info` topics and creates one RViz group per
detected serial number. Closing RViz also stops the included driver.

## Dependencies

Dependencies are declared in `package.xml`. The driver uses `roscpp`, standard
ROS messages, `tf2_ros`, and pthreads. The viewer additionally uses `rospy`,
RViz, and PyYAML. OpenCV, PCL, and Open3D are not required.

## Run and Verify

```bash
# Driver only
roslaunch ilidar_itfs_ros itfs.launch

# Driver and RViz with automatic discovery
roslaunch ilidar_itfs_ros viewer.launch

# Create known RViz groups immediately
roslaunch ilidar_itfs_ros viewer.launch sensor_sns:="2405,2406"
```

An explicit `sensor_sns` list configures RViz only; it does not filter devices
accepted by the driver. Use `fixed_frame:=<frame>` when the sensors use a
parent other than `base_link`.

Replace `2405` with the detected decimal serial number:

```bash
rosnode list
rostopic list | grep ilidar
rostopic echo -n 1 /ilidar_2405/info
rostopic echo -n 1 /ilidar_2405/depth/image_raw/encoding
rostopic echo -n 1 /diagnostics
```

## Topics and Data Contract

Each device publishes below `/ilidar_<SN>`.

|Topic|Type|Description|
|:---|:---|:---|
|`depth/image_raw`|`sensor_msgs/Image`|Depth image|
|`intensity/image_raw`|`sensor_msgs/Image`|Intensity or capture-mode-0 gray image|
|`points`|`sensor_msgs/PointCloud2`|Organized calibrated XYZ with optional intensity|
|`info`|`std_msgs/String`|Latched sensor summary|
|`/diagnostics`|`diagnostic_msgs/DiagnosticArray`|Aggregated sensor status|
|`/tf_static`|`tf2_msgs/TFMessage`|Mount and optical transforms|

|Output|Encoding / fields|Unit and invalid-value rule|
|:---|:---|:---|
|Depth|`mono16`|Unsigned millimeters; zero means invalid/no range|
|Intensity / gray|`mono16`|Original unsigned sensor value|
|Point cloud|Organized `x/y/z` `float32`|Meters; invalid points are `NaN`|
|Optional point field|`intensity` `float32`|Converted sensor intensity|

Products from one completed frame share one host ROS timestamp. Topics are
advertised only when their `publish_*` parameter is enabled and may remain
silent when the configured sensor output does not contain that product.

### Capture Modes

|Capture mode|Image layout|ROS behavior|
|:---:|:---|:---|
|0|`240 x 320` gray|Intensity topic only|
|1–3|`capture_row x 320` depth with optional intensity|Depth, optional intensity, and points|

Capture mode, row count, `data_output`, model, and serial number are fixed when
the device context is created. Restart the driver after changing them.

## Mapping, Frames, and TF

PointCloud2 uses a calibrated direction table. The default is `iTFS-110.dat`;
select `iTFS-80.dat` or a custom table with `mapping_file`. If loading fails,
images remain available while PointCloud2 is disabled.

The direction table is not a pinhole camera model, so this package does not
publish CameraInfo.

```text
base_link
  -> ilidar_<SN>_link
      -> ilidar_<SN>_optical_frame
```

Images use the optical frame and PointCloud2 uses the link frame. Translations
are meters and rotations are roll/pitch/yaw radians. Set `publish_tf: false`
when another component owns the transforms.

## Parameters

|Parameter|Default|Description|
|:---|:---|:---|
|`broadcast_ip`|empty|SDK broadcast-interface override|
|`listening_ip`|empty|SDK receive-interface override|
|`listening_port`|`7256`|UDP receive port|
|`mapping_file`|installed `iTFS-110.dat`|Direction table|
|`parent_frame_id`|`base_link`|Mount parent frame|
|`link_frame_id`|SN-derived|Point-cloud frame|
|`frame_id`|SN-derived|Image optical frame|
|`publish_tf`|`true`|Publish static transforms|
|`publish_depth`|`true`|Publish depth when available|
|`publish_intensity`|`true`|Publish intensity or gray|
|`publish_pointcloud`|`true`|Publish organized points|
|`publish_pointcloud_intensity`|`true`|Add optional intensity field|
|`publish_info`|`true`|Publish latched sensor summary|
|`publish_diagnostics`|`true`|Publish diagnostics|
|`sensor_qos_depth`|`5`|ROS publisher queue size|
|`diagnostics_qos_depth`|`10`|Diagnostics queue size|

Pose parameters use `mount_translation_{x,y,z}`,
`mount_rotation_{roll,pitch,yaw}`, `optical_translation_{x,y,z}`, and
`optical_rotation_{roll,pitch,yaw}`. Values are fixed at startup.

## Configuration Examples

Single iTFS-80 with a measured mount offset:

```yaml
parent_frame_id: base_link
devices:
  ilidar_2405:
    mapping_file: /absolute/path/iTFS-80.dat
    mount_translation_x: 0.25
    mount_translation_z: 0.80
```

Multiple sensors inherit common defaults and override only their mount poses:

```yaml
devices:
  ilidar_2405:
    mount_translation_y: 0.15
  ilidar_2406:
    mount_translation_y: -0.15
```

Disable package-owned TF when transforms come from another component:

```yaml
publish_tf: false
```

Pass an external configuration file with:

```bash
roslaunch ilidar_itfs_ros itfs.launch config:=/absolute/path/itfs.yaml
```

## Diagnostics and Performance

Each initialized sensor contributes `ilidar_<SN>/status` to `/diagnostics`.
Diagnostics include identity, capture layout, sensor warnings, temperature,
power, packet error counters, mapping status, frame IDs, and
`ros_frame_overwrite_count`.

- `OK`: status is present with no active warning.
- `WARN`: waiting for status, sensor/frame warning, missing rows, or missing mapping.
- `ERROR`: sensor identity or immutable capture layout changed.

The receive callback copies requested data into a reusable latest-frame
snapshot. Each sensor has one worker. If processing falls behind, newer input
replaces pending work and `ros_frame_overwrite_count` increases. This favors
low latency and is not a lossless recording queue.

## Limitations and Troubleshooting

- Sensor configuration and maintenance operations remain external.
- Runtime identity, capture layout, and ROS parameter changes require restart.
- Viewer auto-discovery does not add sensors after RViz has started.
- CameraInfo is unavailable because the mapping table is not a pinhole model.

If no sensor topics appear, verify the sensor stream, IPv4 configuration, UDP
port `7256`, firewall, and interface overrides. If PointCloud2 is absent, check
depth output, `publish_pointcloud`, and `mapping_file`. For TF errors, confirm
the RViz fixed frame and unique per-sensor child frames.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for release history.

## License

This package and its included SDK source are licensed under the MIT License.
See [LICENSE](LICENSE). Third-party ROS and system dependencies retain their
own licenses.
