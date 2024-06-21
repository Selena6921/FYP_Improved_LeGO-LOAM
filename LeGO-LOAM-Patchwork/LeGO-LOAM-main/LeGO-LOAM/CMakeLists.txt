cmake_minimum_required(VERSION 3.0.2)
project(patch_lego_loam)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -O3")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_BUILD_TYPE "Release")

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
find_package(Boost REQUIRED COMPONENTS timer thread serialization chrono)

find_package(catkin REQUIRED COMPONENTS
  tf
  roscpp
  rospy
  cv_bridge
  image_transport

  roslaunch
  laser_geometry

  pcl_ros
  pcl_conversions

  std_msgs
  sensor_msgs
  geometry_msgs
  nav_msgs
  cloud_msgs
  message_generation
)

find_package(GTSAM REQUIRED QUIET)
find_package(PCL REQUIRED QUIET)
find_package(OpenCV REQUIRED QUIET)
find_package(Boost 1.54 REQUIRED)

catkin_package(
  INCLUDE_DIRS include
  CATKIN_DEPENDS cloud_msgs
  DEPENDS PCL
  CATKIN_DEPENDS roscpp rospy std_msgs
)

include_directories(
	include
	${catkin_INCLUDE_DIRS}
	${PCL_INCLUDE_DIRS}
	${OpenCV_INCLUDE_DIRS}
	${GTSAM_INCLUDE_DIR}
)

link_directories(
	include
	${OpenCV_LIBRARY_DIRS}
	${PCL_LIBRARY_DIRS}
	${GTSAM_LIBRARY_DIRS}
)

SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp")
include(/home/selena/catkin_ws/src/LeGO-LOAM-Patchwork/LeGO-LOAM-main/LeGO-LOAM/3rdparty/find_dependencies.cmake)

option(USE_SYSTEM_TBB "Use system pre-installed oneAPI/tbb" OFF)
include(/home/selena/catkin_ws/src/LeGO-LOAM-Patchwork/LeGO-LOAM-main/LeGO-LOAM/3rdparty/find_dependencies2.cmake)

 # Get a list of all targets in the current CMake project
 get_property(target_names GLOBAL PROPERTY TARGETS)

 # Iterate over each target and print its properties
 foreach(target_name ${target_names})
     # Get properties of the target
     get_target_property(target_type ${target_name} TYPE)
     get_target_property(target_sources ${target_name} SOURCES)
     get_target_property(target_include_dirs ${target_name} INCLUDE_DIRECTORIES)
     get_target_property(target_link_libraries ${target_name} LINK_LIBRARIES)
 
     # Print or log the target information
     message(STATUS "Target: ${target_name}")
     message(STATUS "  Type: ${target_type}")
     message(STATUS "  Sources: ${target_sources}")
     message(STATUS "  Include Directories: ${target_include_dirs}")
     message(STATUS "  Link Libraries: ${target_link_libraries}")
 endforeach()

add_executable(pub_for_legoloam src/pub_for_legoloam.cpp)
add_dependencies(pub_for_legoloam patchwork_generate_messages_gencpp)
target_link_libraries(pub_for_legoloam ${catkin_LIBRARIES} ${PCL_LIBRARIES} TBB::tbb)

add_executable(imageProjection src/imageProjection.cpp)
add_dependencies(imageProjection ${catkin_EXPORTED_TARGETS} cloud_msgs_gencpp)
target_link_libraries(imageProjection ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBRARIES})

add_executable(featureAssociation src/featureAssociation.cpp)
add_dependencies(featureAssociation ${catkin_EXPORTED_TARGETS} cloud_msgs_gencpp)
target_link_libraries(featureAssociation ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBRARIES})

add_executable(mapOptmization src/mapOptmization.cpp)
target_link_libraries(mapOptmization ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBRARIES} gtsam)

add_executable(transformFusion src/transformFusion.cpp)
target_link_libraries(transformFusion ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBRARIES})


