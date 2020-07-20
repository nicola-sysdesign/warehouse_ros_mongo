cmake_minimum_required(VERSION 2.8.3)
project(warehouse_ros_mongo)

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/")

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED
  roscpp
  rostime
  rostest
  std_msgs
  warehouse_ros
  class_loader
)

## System dependencies are found with CMake's conventions
find_package(Boost REQUIRED COMPONENTS system filesystem thread)
find_package(OpenSSL REQUIRED)
find_package(MongoDB REQUIRED)

if(EXISTS "${MongoDB_INCLUDE_DIR}/mongo/version.h")
  add_definitions(-DWAREHOUSE_ROS_MONGO_HAVE_MONGO_VERSION_H)
endif()

set(MONGO_EXPORT)
if("${MongoDB_LIBRARIES}" MATCHES "\\.so$")
  set(MONGO_EXPORT MongoDB)
endif()

file(MAKE_DIRECTORY "${CATKIN_DEVEL_PREFIX}/include")

## Uncomment this if the package has a setup.py. This macro ensures
## modules and global scripts declared therein get installed
catkin_python_setup()

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if your package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
  INCLUDE_DIRS include ${CATKIN_DEVEL_PREFIX}/include
  LIBRARIES warehouse_ros_mongo
  CATKIN_DEPENDS roscpp rostime std_msgs
  DEPENDS Boost ${MONGO_EXPORT}
)

if (NOT MongoDB_EXPOSE_MACROS)
  add_definitions(-DMONGO_EXPOSE_MACROS)
endif()

configure_file(
  "include/warehouse_ros_mongo/config.h.in"
  "${CATKIN_DEVEL_PREFIX}/include/warehouse_ros_mongo/config.h"
)

###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(
  include
  ${CATKIN_DEVEL_PREFIX}/include
  ${catkin_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${MongoDB_INCLUDE_DIR}
)

link_directories(${catkin_LINK_DIRS})
link_directories(${Boost_LIBRARY_DIRS})

## Declare a C++ library
add_library(warehouse_ros_mongo
  SHARED src/warehouse_ros_mongo_dummy.cpp src/query_results.cpp src/message_collection.cpp
  src/database_connection.cpp)

## Specify libraries to link a library or executable target against
target_link_libraries(warehouse_ros_mongo
  ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${OPENSSL_LIBRARIES} ${MongoDB_LIBRARIES})


if (CATKIN_ENABLE_TESTING)
  add_executable(test_warehouse_ros_mongo_cpp test/test_warehouse_ros_mongo.cpp)
  target_link_libraries(test_warehouse_ros_mongo_cpp warehouse_ros_mongo ${GTEST_LIBRARIES})

  # Disable unit test, because it hangs in Travis. TODO: Provide a mongodb server in Travis.
  # add_rostest(test/warehouse_ros_mongo.test)
endif()

#############
## Install ##
#############

# all install targets should use catkin DESTINATION variables
# See http://ros.org/doc/api/catkin/html/adv_user_guide/variables.html

## Mark executable scripts (Python etc.) for installation
## in contrast to setup.py, you can choose the destination
install(
  PROGRAMS src/mongo_wrapper_ros.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

## Mark libraries for installation
install(
  TARGETS warehouse_ros_mongo
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)

## Mark cpp header files for installation
install(
  DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
)
install(
  FILES "${CATKIN_DEVEL_PREFIX}/include/warehouse_ros_mongo/config.h"
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)

## Mark other files for installation (e.g. launch and bag files, etc.)
install(
  FILES mongo_database_connection_plugin_description.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
