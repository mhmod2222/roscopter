cmake_minimum_required(VERSION 2.8.3)
project(roscopter)

find_package(catkin REQUIRED COMPONENTS
  nav_msgs
  cmake_modules
  sensor_msgs
  roscpp
  rospy
  eigen_conversions
  std_msgs
  rosflight_msgs
  rosflight_utils
  dynamic_reconfigure
)
find_package(Eigen 3 REQUIRED)

generate_dynamic_reconfigure_options(
  cfg/Controller.cfg
)

# Added "-fopenmp -O3" to make Eigen run REALLY fast
set(CMAKE_CXX_FLAGS "-std=c++0x -fopenmp -O3")

add_service_files(
  FILES
  AddWaypoint.srv
  RemoveWaypoint.srv
  SetWaypointsFromFile.srv
)

generate_messages(
  DEPENDENCIES
  std_msgs
  geometry_msgs
)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES roscopter
  CATKIN_DEPENDS roscpp rospy nav_msgs std_msgs
)

###########
## Build ##
###########
include_directories(include)
include_directories(${catkin_INCLUDE_DIRS})

## Declare cpp libraries
add_library(roscopter
  include/lib/eigen.h
  src/lib/eigen.cpp
)
add_dependencies(roscopter rosflight_msgs_generate_messages_cpp)
target_link_libraries(roscopter ${catkin_LIBRARIES})


## Declare cpp executables
add_executable(controller
  src/controller/controller_node.cpp
  src/controller/controller.cpp
  include/controller/controller.h)
add_dependencies(controller rosflight_msgs_generate_messages_cpp roscopter_gencfg)
target_link_libraries(controller ${catkin_LIBRARIES})

add_executable(mekf
  src/mekf/mekf_node.cpp
  src/mekf/mekf.cpp
  src/mekf/mekf_math.cpp
)
add_dependencies(mekf rosflight_msgs_generate_messages_cpp)
target_link_libraries(mekf ${catkin_LIBRARIES})
