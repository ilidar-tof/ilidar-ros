cmake_minimum_required(VERSION 3.0.2)
project(ilidar_itfs_ros VERSION 2.0.0)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

find_package(catkin REQUIRED COMPONENTS
  diagnostic_msgs
  geometry_msgs
  roscpp
  roslib
  sensor_msgs
  std_msgs
  tf2
  tf2_ros
)
find_package(Threads REQUIRED)

catkin_package(CATKIN_DEPENDS
  diagnostic_msgs geometry_msgs roscpp roslib sensor_msgs std_msgs tf2 tf2_ros
)

add_executable(ilidar_core src/ilidar_core.cpp src/ilidar.cpp)
target_include_directories(ilidar_core PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src ${catkin_INCLUDE_DIRS})
target_link_libraries(ilidar_core ${catkin_LIBRARIES} Threads::Threads)

install(TARGETS ilidar_core RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
catkin_install_python(PROGRAMS scripts/ilidar_viewer DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
install(DIRECTORY launch config rviz dat DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})
install(FILES README.md CHANGELOG.md LICENSE
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})
