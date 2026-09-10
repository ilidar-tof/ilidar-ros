# Changelog

All notable changes to the iLidar ROS 1 package collection are documented in
this file.

## [2.0.2] - 2026-09-09

### Added

- Added the `advanced_trim` packet definition; internal `ilidar_lite_pack_ver` and `ilidar_lite_lib_ver` are now 2.0.2.
- Support for iTFS-LITE firmware V1.2.6 F1 100 MHz Single output, including
  frequency-aware depth and native XYZ conversion and a dedicated F1 depth
  LOG8 lookup table.

### Fixed

- Corrected RAW and linear 8-bit distance conversion to use the received
  frame's frequency mode: 1.499 m for F1 and 7.495 m for F2/Dual, with
  consistent fixed-point scaling across image and point-cloud outputs.
- Corrected the distortion model used for iTFS-LITE depth-to-point-cloud
  reconstruction, improving agreement with native sensor XYZ output.
- Preserved the organized 320x240 PointCloud2 dimensions and row stride after
  buffer allocation.

## [2.0.1] - 2026-07-22

### Changed

- Changed the iTFS-LITE depth visualization from an RViz Camera display to an
  Image display, consistent with the amplitude, intensity, and confidence
  image streams.
- Updated collection and package version metadata to V2.0.1.

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
