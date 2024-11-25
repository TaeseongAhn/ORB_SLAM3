  cmake_minimum_required(VERSION 3.5)
  project(orbslam3)

  # You should set the PYTHONPATH to your own python site-packages path
  set(ENV{PYTHONPATH} "/opt/ros/foxy/lib/python3.8/site-packages/")

  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules/")

  # Default to C++14
  if(NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 14)
  endif()

  if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    add_compile_options(-Wall -Wextra -Wpedantic)
  endif()

  find_package(ament_cmake REQUIRED)
  find_package(rclcpp REQUIRED)
  find_package(sensor_msgs REQUIRED)
  find_package(cv_bridge REQUIRED)
  find_package(message_filters REQUIRED)
  find_package(Sophus REQUIRED)
  find_package(Pangolin REQUIRED)
  find_package(ORB_SLAM3 REQUIRED)
  find_package(tf2_ros REQUIRED)
  find_package(tf2 REQUIRED)
  find_package(image_transport REQUIRED)

  set(ORB_SLAM3_DIR $ENV{ORB_SLAM3_DIR})

  include_directories(
    include
    ${ORB_SLAM3_ROOT_DIR}
    ${ORB_SLAM3_ROOT_DIR}/include
    ${ORB_SLAM3_ROOT_DIR}/include/CameraModels
    ${ORB_SLAM3_ROOT_DIR}/Thirdparty/Sophus
    ${ORB_SLAM3_ROOT_DIR}/Thirdparty/DBoW2
    ${ORB_SLAM3_ROOT_DIR}/Thirdparty/DBoW2/DBoW2
  )
  set(LIBS 
      ${ORB_SLAM3_ROOT_DIR}/lib/libORB_SLAM3.so
      ${ORB_SLAM3_ROOT_DIR}/Thirdparty/DBoW2/lib/libDBoW2.so
      ${ORB_SLAM3_ROOT_DIR}/Thirdparty/g2o/lib/libg2o.so
      ${OpenCV_LIBS}
      ${EIGEN3_LIBS}
      ${Pangolin_LIBRARIES}
      ${image_transport_LIBRARIES}
      ${tf2_ros_LIBRARIES}
      ${tf2_LIBRARIES}
      -lboost_serialization
      -lcrypto)

  link_directories(
    include
  )

  add_executable(mono
    src/monocular/mono.cpp
    src/monocular/monocular-slam-node.cpp
  )
  ament_target_dependencies(mono rclcpp sensor_msgs cv_bridge ORB_SLAM3 Pangolin)

  add_executable(rgbd src/rgbd/rgbd.cpp src/rgbd/common.cpp)
  target_link_libraries(rgbd ${LIBS})
  ament_target_dependencies(rgbd rclcpp sensor_msgs cv_bridge message_filters tf2_ros tf2 image_transport Pangolin)


  add_executable(stereo
    src/stereo/stereo.cpp
    src/stereo/stereo-slam-node.cpp
  )
  ament_target_dependencies(stereo rclcpp sensor_msgs cv_bridge message_filters
  ORB_SLAM3 Pangolin)

  add_executable(stereo-inertial
    src/stereo-inertial/stereo-inertial.cpp
    src/stereo-inertial/stereo-inertial-node.cpp
  )
  ament_target_dependencies(stereo-inertial rclcpp sensor_msgs cv_bridge ORB_SLAM3 Pangolin)

  install(TARGETS mono rgbd stereo stereo-inertial
    DESTINATION lib/${PROJECT_NAME})

  # Install launch files.
  #install(DIRECTORY launch config vocabulary
  #  DESTINATION share/${PROJECT_NAME}/)

  ament_package()