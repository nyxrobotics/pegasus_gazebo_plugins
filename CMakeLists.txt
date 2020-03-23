cmake_minimum_required(VERSION 2.8.3)
cmake_policy(SET CMP0054 NEW)
project(pegasus_gazebo_plugins)
set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")

option(ENABLE_CLANG_BUILD "Enable Clang Build Tools" ON)
option(ENABLE_CLANG_FORMAT_FIX "Perform fixes for Clang-Format" OFF)
option(ENABLE_CLANG_TIDY_FIX "Perform fixes for Clang-Tidy" OFF)

include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/ClangFormatMacros.cmake)
#set(DUMP_CONFIG OFF)
if(ENABLE_CLANG_BUILD)
	enable_clang_build()
endif()
if(ENABLE_CLANG_FORMAT_FIX)
	clang_format_fix()
endif()
if(ENABLE_CLANG_TIDY_FIX)
	clang_tidy_fix()
endif()

# Load catkin and all dependencies required for this package
find_package(catkin REQUIRED COMPONENTS 
  roscpp 
  gazebo_ros
  control_toolbox
)

# Depend on system install of Gazebo
find_package(gazebo REQUIRED)
find_package(Boost REQUIRED)

link_directories(${GAZEBO_LIBRARY_DIRS})
include_directories(${Boost_INCLUDE_DIR} ${catkin_INCLUDE_DIRS} ${GAZEBO_INCLUDE_DIRS} include)

add_library(pegasus_gazebo_closed_loop_plugin src/closed_loop_plugin.cpp)
target_link_libraries(pegasus_gazebo_closed_loop_plugin ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES})

catkin_package(
  DEPENDS 
    roscpp 
    gazebo_ros
    control_toolbox
)
