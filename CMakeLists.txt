cmake_minimum_required(VERSION 3.16)
project(direct_visual_lidar_calibration)

add_compile_options(-std=c++17)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_LIST_DIR}/cmake")

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING "Choose the type of build." FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

if($ENV{ROS_VERSION} EQUAL 1)
  # ROS1
  find_package(catkin REQUIRED COMPONENTS rosbag sensor_msgs cv_bridge)
  catkin_package()
else()
  # ROS2
  find_package(ament_cmake_auto REQUIRED)
  find_package(ament_cmake_python REQUIRED)
  ament_auto_find_build_dependencies()
endif()

find_package(PCL REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(OpenCV REQUIRED)
find_package(Boost REQUIRED COMPONENTS filesystem program_options)
find_package(Iridescence REQUIRED)

include_directories(
  ../include
  ../thirdparty/json/include
  ../thirdparty/Bonxai/include
  ../thirdparty/nanoflann/include
  ../thirdparty/Sophus
  ${Boost_INCLUDE_DIRS}
  ${CERES_INCLUDE_DIRS}
  ${GTSAM_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${Eigen3_INCLUDE_DIRS}
)

set(OLD_DISTRO "humble;galactic;foxy;rolling")
if($ENV{ROS_DISTRO} IN_LIST OLD_DISTRO)
  add_definitions(-DCV_BRIDGE_INCLUDE_H)
else()
  add_definitions(-DCV_BRIDGE_INCLUDE_HPP)
endif()

find_package(OpenMP)
if (OPENMP_FOUND)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

add_subdirectory(direct_visual_lidar_calibration-nonROS)


if($ENV{ROS_VERSION} EQUAL 1)
  # ROS1
  add_executable(preprocess
    src/preprocess_ros1.cpp
  )
  target_link_libraries(preprocess
    direct_visual_lidar_calibration
  )
  target_include_directories(preprocess PUBLIC
    ${catkin_INCLUDE_DIRS}
  )
  target_link_libraries(preprocess
    ${catkin_LIBRARIES}
  )
else()
  # ROS2
  ament_auto_add_executable(preprocess
    src/preprocess_ros2.cpp
  )
  target_link_libraries(preprocess
    direct_visual_lidar_calibration
  )
endif()

# preprocess_map
add_executable(preprocess_map
  src/preprocess_map.cpp
)
target_include_directories(preprocess_map PRIVATE
  ${catkin_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
)
target_link_libraries(preprocess_map
  direct_visual_lidar_calibration
  Iridescence::Iridescence
  ${Boost_LIBRARIES}
  # ${PCL_LIBRARIES}
  ${OpenCV_LIBRARIES}
)

install(
  TARGETS
  
)

install(
  TARGETS
  preprocess_map 
  DESTINATION
  lib/${PROJECT_NAME}
)

if($ENV{ROS_VERSION} EQUAL 1)
  # ROS1
else()
  # ROS2
  ament_python_install_package(${PROJECT_NAME})
  install(PROGRAMS scripts/find_matches_superglue.py DESTINATION lib/${PROJECT_NAME})
  ament_export_libraries(direct_visual_lidar_calibration)
  ament_auto_package()
endif()