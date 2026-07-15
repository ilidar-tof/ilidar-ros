# Changelog

## [2.0.0] - 2026-07-15

Initial release of `ilidar_itfs_ros`.

### Added

- Multi-device `/ilidar_<SN>` depth, intensity, point-cloud, info, TF, and
  diagnostics outputs.
- Capture modes 0 through 3 with immutable layout validation.
- Shared iTFS-110/iTFS-80 direction-table loading for calibrated point clouds.
- Per-SN worker threads, reusable snapshots, subscriber-driven processing, and
  latest-pending-frame replacement.
- Driver-only and supervised RViz launch modes with automatic SN discovery.
- Kinetic, Melodic, and Noetic build compatibility.
