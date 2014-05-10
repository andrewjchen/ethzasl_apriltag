cmake_minimum_required(VERSION 2.8.3)
project(ethzasl_apriltag)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  tf
  image_transport
  cv_bridge
  dynamic_reconfigure
)

find_package(Boost REQUIRED COMPONENTS system)
#SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/cmake/)
find_package(OpenCV REQUIRED)
find_package(Eigen REQUIRED)

#add_message_files(
#  FILES
  
#generate_messages(
#  DEPENDENCIES
#  std_msgs
#)

generate_dynamic_reconfigure_options(
  cfg/ethzasl_apriltagParams.cfg
  )


catkin_package(
  INCLUDE_DIRS include
#  LIBRARIES ethzasl_apriltag
  CATKIN_DEPENDS roscpp rospy std_msgs
#  DEPENDS system_lib
)

include_directories(include)
include_directories(cfg/cpp)
include_directories(
  ${catkin_INCLUDE_DIRS}
)
include_directories(${Eigen_INCLUDE_DIRS})

#add_executable(april_node 
#  src/main.cpp 
#  src/april_interface.cpp
#  src/Params.cpp)
#target_link_libraries(april_node ${Boost_LIBRARIES} ${catkin_LIBRARIES})

#include($ENV{ROS_ROOT}/core/rosbuild/rosbuild.cmake)

# Avoid repeating the IF statement in the ENDIF and ELSE
SET(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS true)

# Set the build type.  Options are:
#  Coverage       : w/ debug symbols, w/o optimization, w/ code-coverage
#  Debug          : w/ debug symbols, w/o optimization
#  Release        : w/o debug symbols, w/ optimization
#  RelWithDebInfo : w/ debug symbols, w/ optimization
#  MinSizeRel     : w/o debug symbols, w/ optimization, stripped binaries
set(ROS_BUILD_TYPE RelWithDebInfo)

#rosbuild_init()

# Generate dynamic parameters
#rosbuild_find_ros_package(dynamic_reconfigure)
#include(${dynamic_reconfigure_PACKAGE_PATH}/cmake/cfgbuild.cmake)
#gencfg()

IF(UNIX AND NOT WIN32)
 SET(CMAKE_CXX_FLAGS "-Wall -Wextra -Wno-unused-parameter -fpermissive")		# Enable lots of warning flags
ELSE()
	SET(CMAKE_CXX_FLAGS "/fp:precise /fp:except /bigobj /Zm1000 /EHsc /DWIN32")	# Refer to Stellarium. Flags used on purpose
ENDIF()

########### Find packages ###########
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/cmake/)
FIND_PACKAGE(OpenCV REQUIRED)
FIND_PACKAGE(Eigen3 REQUIRED)

#define ESM_LIB, ESM_HEADER
IF(UNIX AND NOT WIN32)
  IF(APPLE)
 	find_library(ESM_LIB ESMlib.a PATHES ${CMAKE_SOURCE_DIR}/include/esm/osx/ NO_DEFAULT_PATH)
 	SET(ESM_HEADER ${CMAKE_SOURCE_DIR}/include/esm/osx/)
  ELSE()
	IF(CMAKE_SIZEOF_VOID_P EQUAL 8) 
 		find_library(ESM_LIB ESMlib.a PATHES ${CMAKE_SOURCE_DIR}/include/esm/unix64/ NO_DEFAULT_PATH)
 		SET(ESM_HEADER ${CMAKE_SOURCE_DIR}/include/esm/unix64/)
	ELSE()
 		find_library(ESM_LIB ESMlib.a PATHES ${CMAKE_SOURCE_DIR}/include/esm/unix32/ NO_DEFAULT_PATH)
 		SET(ESM_HEADER ${CMAKE_SOURCE_DIR}/include/esm/unix32/)
	ENDIF()
  ENDIF()
ELSE()
 find_library(ESM_LIB ESMlibry.lib PATHES ${CMAKE_SOURCE_DIR}/include/esm/win NO_DEFAULT_PATH)
 find_file(ESM_DLL ESMlibry.dll PATHES ${CMAKE_SOURCE_DIR}/include/esm/win NO_DEFAULT_PATH)
 		SET(ESM_HEADER ${CMAKE_SOURCE_DIR}/include/esm/win/)
ENDIF()

#set the default path for built executables to the "bin" directory
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
#set the default path for built libraries to the "lib" directory
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

########### uninstall files ###############
CONFIGURE_FILE(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
  IMMEDIATE @ONLY)

ADD_CUSTOM_TARGET(uninstall
  "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake")
  
ADD_SUBDIRECTORY(application)


add_dependencies(april_node ${PROJECT_NAME}_gencfg)