cmake_minimum_required(VERSION 3.0.2)
project(ilidar_lite_ros VERSION 2.0.0)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

find_package(catkin REQUIRED COMPONENTS
  diagnostic_msgs
  geometry_msgs
  roscpp
  sensor_msgs
  std_msgs
  tf2
  tf2_ros
)
find_package(Threads REQUIRED)

# Export only ROS dependencies: the copied SDK remains an implementation detail
# of this executable rather than a public package library.
catkin_package(
  CATKIN_DEPENDS
    diagnostic_msgs
    geometry_msgs
    roscpp
    sensor_msgs
    std_msgs
    tf2
    tf2_ros
)

add_executable(ilidar_lite_core
  src/ilidar_lite_core.cpp
  src/ilidar_lite.cpp
)

target_include_directories(ilidar_lite_core PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${catkin_INCLUDE_DIRS}
)

target_link_libraries(ilidar_lite_core
  ${catkin_LIBRARIES}
  Threads::Threads
)

# Install both launch modes, their shared configuration, and the RViz template
# so source-space and install-space usage follow the same paths.
install(TARGETS ilidar_lite_core
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

catkin_install_python(PROGRAMS scripts/ilidar_lite_viewer
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY launch config rviz
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(FILES README.md CHANGELOG.md LICENSE
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
