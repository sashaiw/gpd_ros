cmake_minimum_required(VERSION 3.8)
project(gpd_ros)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

#find_package(eigen_conversions REQUIRED)
find_package(ament_cmake REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(rclcpp REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(std_msgs REQUIRED)
find_package(builtin_interfaces REQUIRED)
#find_package(rclpy REQUIRED)
find_package(tf2_eigen REQUIRED)

set(THIS_PACKAGE_INCLUDE_DEPENDS
  builtin_interfaces
  rclcpp
  tf2_eigen
  sensor_msgs
  geometry_msgs
  std_msgs
)

# PCL
find_package(PCL 1.9 REQUIRED)
include_directories(${PCL_INCLUDE_DIRS})
link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})

# OpenCV
find_package(OpenCV REQUIRED)

# System dependencies are found with CMake's conventions
find_library(GPD_LIB NAMES gpd PATHS /usr/local/lib PATH_SUFFIXES lib NO_DEFAULT_PATH)
if (GPD_LIB)
  message(STATUS "Library GPD found in ${GPD_LIB}")
else()
  message(FATAL_ERROR "Library GPD not found")
endif()
include_directories(${GPD_LIB_INCLUDE_DIR})
message(STATUS "gpd_include_dir: ${GPD_LIB_INCLUDE_DIR}")

# msg/srv definitions
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/CloudIndexed.msg"
  "msg/CloudSamples.msg"
  "msg/CloudSources.msg"
  "msg/GraspConfig.msg"
  "msg/GraspConfigList.msg"
  "msg/SamplesMsg.msg"
  "srv/DetectGrasps.srv"
  DEPENDENCIES builtin_interfaces std_msgs geometry_msgs sensor_msgs
)

include_directories(include ${catkin_INCLUDE_DIRS} ${PCL_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS})

add_executable(${PROJECT_NAME}_detect_grasps src/gpd_ros/grasp_detection_node.cpp)
ament_target_dependencies(${PROJECT_NAME}_detect_grasps ${THIS_PACKAGE_INCLUDE_DEPENDS})
# required to use interface from same package
rosidl_target_interfaces(${PROJECT_NAME}_detect_grasps ${PROJECT_NAME} "rosidl_typesupport_cpp")

ament_export_dependencies(rosidl_default_runtime)
ament_package()
