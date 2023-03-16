cmake_minimum_required(VERSION 3.0.0)
# project(gazebosensorplugins)
project(uwb_gazebo_plugin)

# QT related:
# set(CMAKE_AUTOMOC ON)
# set(CMAKE_AUTOUIC ON)

SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}  -std=c++11")

add_definitions(-std=c++11)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  gazebo_ros
  roscpp
  rospy
  std_msgs
  message_generation
  tf
)

## Generate messages in the 'msg' folder
add_message_files(
  FILES
  Ranging.msg
 )

## Generate added messages and services with any dependencies listed here
generate_messages(
  DEPENDENCIES
  std_msgs
)

# Depend on system install of Gazebo
find_package(gazebo REQUIRED)

link_directories(${GAZEBO_LIBRARY_DIRS})
include_directories(${Boost_INCLUDE_DIR} ${catkin_INCLUDE_DIRS} ${GAZEBO_INCLUDE_DIRS})

add_library(uwb_plugin src/UwbPlugin.cpp)
target_link_libraries(uwb_plugin ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES} ${Boost_LIBRARIES})
add_dependencies(uwb_plugin ${catkin_EXPORTED_TARGETS})

# add_library(tag_pos_publisher_plugin src/TagPosPublisherPlugin.cpp)
# target_link_libraries(tag_pos_publisher_plugin ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES} ${Boost_LIBRARIES})
# add_dependencies(tag_pos_publisher_plugin ${catkin_EXPORTED_TARGETS})

catkin_package(
  DEPENDS 
    roscpp 
    gazebo_ros
    message_runtime
)
##add_dependencies(${PROJECT_NAME} generate_messages_cpp)
