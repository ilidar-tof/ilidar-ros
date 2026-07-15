# Changelog

All notable changes to the iLidar ROS 1 package collection are documented in
this file. Package-level details are available in each package directory.

## [2.0.0] - 2026-07-15

Initial ROS 1 V2 release.

### Added

- `ilidar_itfs_ros` and `ilidar_lite_ros` receive-only packages.
- SN-derived multi-device topics, frames, workers, and configuration overrides.
- Stable ROS image contracts, organized PointCloud2, static TF, sensor info,
  and diagnostics.
- Driver-only and supervised RViz launch modes with automatic SN discovery.
- Kinetic, Melodic, and Noetic source compatibility.
- Detailed quick-start, parameter, performance, and troubleshooting guides.

### Notes

- Sensor configuration remains the responsibility of the dedicated product
  tools.
- Runtime sensor identity and capture-layout changes require a driver restart.
