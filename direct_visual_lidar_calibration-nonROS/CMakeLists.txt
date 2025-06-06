cmake_minimum_required(VERSION 3.16)
project(direct_visual_lidar_calibration_nonROS)

add_compile_options(-std=c++17)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_LIST_DIR}/cmake")

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING "Choose the type of build." FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

find_package(PCL REQUIRED)
find_package(Ceres REQUIRED)
find_package(GTSAM REQUIRED)
find_package(OpenCV REQUIRED)
find_package(Boost REQUIRED COMPONENTS filesystem program_options)
find_package(Iridescence REQUIRED)

find_package(OpenMP)
if (OPENMP_FOUND)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

# direct_visual_lidar_calibration
add_library(direct_visual_lidar_calibration SHARED
  src/camera/create_camera.cpp
  src/vlcal/common/frame.cpp
  src/vlcal/common/frame_cpu.cpp
  src/vlcal/common/ivox.cpp
  src/vlcal/common/cloud_covariance_estimation.cpp
  src/vlcal/common/time_keeper.cpp
  src/vlcal/common/estimate_fov.cpp
  src/vlcal/common/estimate_pose.cpp
  src/vlcal/common/points_color_updater.cpp
  src/vlcal/common/visual_lidar_data.cpp
  src/vlcal/common/visual_lidar_visualizer.cpp
  src/vlcal/preprocess/preprocess.cpp
  src/vlcal/preprocess/generate_lidar_image.cpp
  src/vlcal/preprocess/static_point_cloud_integrator.cpp
  src/vlcal/preprocess/dynamic_point_cloud_integrator.cpp
  src/vlcal/calib/cost_calculator_nid.cpp
  src/vlcal/calib/view_culling.cpp
  src/vlcal/calib/visual_camera_calibration.cpp
)
target_include_directories(direct_visual_lidar_calibration PUBLIC
  include
  ../3rdparty/json/include
  ../3rdparty/nanoflann/include
  ../3rdparty/Sophus
  ${Boost_INCLUDE_DIRS}
  ${CERES_INCLUDE_DIRS}
  ${GTSAM_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
)
target_link_libraries(direct_visual_lidar_calibration
  fmt
  ${Boost_LIBRARIES}
  ${PCL_LIBRARIES}
  ${CERES_LIBRARIES}
  ${GTSAM_LIBRARIES}
  ${OpenCV_LIBRARIES}
  Iridescence::Iridescence
)


# initial_guess_manual
add_executable(initial_guess_manual
  src/initial_guess_manual.cpp
)

target_link_libraries(initial_guess_manual
  direct_visual_lidar_calibration
)

# initial_guess_auto
add_executable(initial_guess_auto
  src/initial_guess_auto.cpp
)

target_link_libraries(initial_guess_auto
  direct_visual_lidar_calibration
)

# calibrate
add_executable(calibrate
  src/calibrate.cpp
)

target_link_libraries(calibrate
  direct_visual_lidar_calibration
)

# viewer
add_executable(viewer
  src/viewer.cpp
)

target_link_libraries(viewer
  direct_visual_lidar_calibration
)

install(
  TARGETS
  direct_visual_lidar_calibration
)

install(
  TARGETS
  
  initial_guess_manual
  initial_guess_auto
  calibrate
  viewer
  DESTINATION
  lib/${PROJECT_NAME}
)