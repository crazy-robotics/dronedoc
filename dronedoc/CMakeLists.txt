cmake_minimum_required(VERSION 2.8.8 FATAL_ERROR)

project(dronedoc)

add_compile_options(-std=c++11)

find_package(Boost 1.4.0 COMPONENTS system filesystem REQUIRED)

find_package(catkin REQUIRED COMPONENTS
  geometry_msgs
  mavros_msgs
  roscpp
  sensor_msgs
  nav_msgs
  nav_core
  tf
  costmap_2d
  actionlib
  trajectory_msgs
  pluginlib
  moveit_msgs
  moveit_core
)

find_package(yaml-cpp REQUIRED)
find_package(Protobuf REQUIRED)
find_package(ompl REQUIRED)

find_package(gazebo REQUIRED)
if("${GAZEBO_VERSION}" VERSION_LESS "7.4")
  message(FATAL_ERROR "Gazebo 7.4 or later is required.")
endif()

include (FindPkgConfig)
if (PKG_CONFIG_FOUND)
  pkg_check_modules(GAZEBO gazebo)
  pkg_check_modules(SDF sdformat)
endif()

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES map_builder
  CATKIN_DEPENDS
    geometry_msgs
    mavros_msgs
    roscpp
    sensor_msgs
    nav_msgs
    nav_core
    tf
    navigation
    actionlib
    moveit_core
    moveit_msgs
    trajectory_msgs
    pluginlib
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${YAML_CPP_INCLUDE_DIR}
  ${GAZEBO_INCLUDE_DIRS}
  ${SDF_INCLUDE_DIRS}
  ${CMAKE_CURRENT_BINARY_DIR}/msgs/proto
)

link_directories(${GAZEBO_LIBRARY_DIRS} ${CMAKE_CURRENT_BINARY_DIR}/msgs/proto)
add_subdirectory(msgs/proto)

# Build map builder and request publisher
# add_executable (request_publisher src/gazebo/request_publisher.cpp)
# target_link_libraries(request_publisher dronedoc ${GAZEBO_LIBRARIES}
#                                                  ${Boost_LIBRARIES}
#                                                  ${SDF_LIBRARIES}
#                                                  ${catkin_LIBRARIES})
# add_dependencies(request_publisher dronedoc)
# add_library(map_builder SHARED src/gazebo/map_builder.cpp)
# target_link_libraries(map_builder dronedoc ${Boost_LIBRARIES}
#                                            ${GAZEBO_LIBRARIES}
#                                            ${SDF_LIBRARIES}
#                                            ${YAML_CPP_LIBRARIES}
#                                            ${catkin_LIBRARIES})
# add_dependencies(map_builder dronedoc)

# Build moveit library
add_library(${PROJECT_NAME}_moveit_multi_dof_controller_manager src/moveit_multi_dof_controller_manager.cpp)
target_link_libraries(${PROJECT_NAME}_moveit_multi_dof_controller_manager ${catkin_LIBRARIES} ${Boost_LIBRARIES})

# Build ROS nodes
add_executable(drone_controller src/drone_controller.cpp)
add_executable(offboard_sample src/offboard_sample.cpp)
add_executable(odom_publisher src/odom_publisher.cpp)
add_executable(rrt_planner src/rrt_planner.cpp)

target_link_libraries(drone_controller ${catkin_LIBRARIES})
target_link_libraries(offboard_sample ${catkin_LIBRARIES})
target_link_libraries(odom_publisher ${catkin_LIBRARIES})
target_link_libraries(rrt_planner ${catkin_LIBRARIES}
                                  ${OMPL_LIBRARIES}
                                  ${Boost_LIBRARIES})

# Create symbolic link of airframe defs
set(PACK_AIRFRAME_DIR ${CMAKE_CURRENT_SOURCE_DIR}/posix_airframes)
set(LOCAL_AIRFRAME_DIR $ENV{HOME}/.ros/etc/init.d-posix)

if(NOT EXISTS ${LOCAL_AIRFRAME_DIR})
  file(MAKE_DIRECTORY ${LOCAL_AIRFRAME_DIR})
endif()

add_custom_target(iris_2d_lidar
                  ALL ln -s -b ${PACK_AIRFRAME_DIR}/70010_iris_2d_lidar ${LOCAL_AIRFRAME_DIR}/)

