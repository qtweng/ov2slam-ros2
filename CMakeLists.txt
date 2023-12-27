cmake_minimum_required(VERSION 3.5)
project(ov2slam)

# Set C++14 Standard
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_BUILD_TYPE "Release")

# Set Compiler Flags
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -Wall -march=native -g")

# Option for OpenCV contrib modules
option(WITH_OPENCV_CONTRIB "Use OpenCV contrib modules" ON)

# Output message based on OpenCV contrib usage
if(WITH_OPENCV_CONTRIB)
    message(STATUS "Using OpenCV contrib modules.")
    add_definitions(-DOPENCV_CONTRIB)
endif()

# Find ROS2 and other required packages
find_package(ament_cmake REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(image_transport REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(PCL REQUIRED)
find_package(pcl_msgs REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)
find_package(Boost REQUIRED COMPONENTS system filesystem)

# Find non-ROS packages
find_package(OpenCV REQUIRED)
find_package(Eigen3 3.4.0 REQUIRED NO_MODULE)
find_package(opengv QUIET)
find_package(Ceres REQUIRED PATHS ${PROJECT_SOURCE_DIR}/Thirdparty/ceres-solver/install/)
find_package(Sophus REQUIRED PATHS ${PROJECT_SOURCE_DIR}/Thirdparty/Sophus/build/)

# iBOW-LCD option
option(WITH_IBOW_LCD "Use iBoW-LCD for Loop Closing" OFF)
if(WITH_IBOW_LCD AND EXISTS "${PROJECT_SOURCE_DIR}/Thirdparty/ibow_lcd/build/liblcdetector.so")
  add_definitions(-DIBOW_LCD)
  message(STATUS "iBoW-LCD found! Loop Closer will be enabled.")
else()
  message(STATUS "iBoW-LCD not found. Loop Closer will not be enabled.")
endif()

# Declare a C++ library
add_library(${PROJECT_NAME}
  src/ov2slam.cpp
  src/camera_visualizer.cpp
  src/visual_front_end.cpp
  src/frame.cpp
  src/slam_params.cpp
  src/camera_calibration.cpp
  src/feature_extractor.cpp
  src/feature_tracker.cpp
  src/map_manager.cpp
  src/map_point.cpp
  src/mapper.cpp
  src/multi_view_geometry.cpp
  src/ceres_parametrization.cpp
  src/optimizer.cpp
  src/estimator.cpp
  src/loop_closer.cpp
)

# Add ROS2 dependencies
ament_target_dependencies(${PROJECT_NAME}
  "rclcpp"
  "std_msgs"
  "sensor_msgs"
  "geometry_msgs"
  "nav_msgs"
  "visualization_msgs"
  "cv_bridge"
  "image_transport"
  "tf2_ros"
  "tf2_geometry_msgs"
  "pcl_conversions"
)

# Add include directories
target_include_directories(${PROJECT_NAME} PUBLIC
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${CMAKE_CURRENT_SOURCE_DIR}/include/ceres_parametrization
  ${PCL_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIRS}
  ${Ceres_INCLUDE_DIRS}
  ${Sophus_INCLUDE_DIRS}
  /opt/ros/humble/include
)

# Link libraries
target_link_libraries(${PROJECT_NAME}
  Eigen3::Eigen
  Sophus::Sophus
  ${PCL_LIBRARIES}
  ${Boost_LIBRARIES}
  ${OpenCV_LIBS}
  Ceres::ceres
)

# Conditional linking for iBoW-LCD and OpenGV
if(WITH_IBOW_LCD)
  target_link_libraries(${PROJECT_NAME} ${PROJECT_SOURCE_DIR}/Thirdparty/ibow_lcd/build/liblcdetector.so)
endif()
if(opengv_FOUND)
  target_link_libraries(${PROJECT_NAME} opengv)
endif()

# Declare a C++ executable
add_executable(${PROJECT_NAME}_node src/ov2slam_node.cpp)

# Specify libraries to link the executable against
target_link_libraries(${PROJECT_NAME}_node ${PROJECT_NAME})

# Install targets
install(TARGETS ${PROJECT_NAME} ${PROJECT_NAME}_node
  DESTINATION lib/${PROJECT_NAME}
)

# Export dependencies and include directories
ament_export_include_directories(include)
ament_export_dependencies(
  rclcpp
  std_msgs
  sensor_msgs
  geometry_msgs
  visualization_msgs
  cv_bridge
  image_transport
  tf2_ros
  tf2_geometry_msgs
  pcl_conversions
)

ament_package()

