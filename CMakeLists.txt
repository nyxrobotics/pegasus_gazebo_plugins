cmake_minimum_required(VERSION 2.8.3)
project(pegasus_gazebo_plugins)
option(ENABLE_CLANG_BUILD "Enable Clang Build Tools" ON)
option(ENABLE_CLANG_FORMAT_FIX "Perform fixes for Clang-Format" ON)
option(ENABLE_CLANG_TIDY_FIX "Perform fixes for Clang-Tidy" ON)

message( "[CMake Option] ENABLE_CLANG_BUILD (ON/OFF)    : ${ENABLE_CLANG_BUILD}")
message( "[CMake Option] CLANG_FORMAT_FIX (ON/OFF)      : ${ENABLE_CLANG_FORMAT_FIX}")
message( "[CMake Option] ENABLE_CLANG_TIDY_FIX (ON/OFF) : ${ENABLE_CLANG_TIDY_FIX}")

#set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${GAZEBO_CXX_FLAGS}")

if(ENABLE_CLANG_BUILD)
	set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
	set(CMAKE_C_COMPILER "/usr/bin/clang-8" CACHE STRING "clang compiler" FORCE)
	set(CMAKE_CXX_COMPILER "/usr/bin/clang++-8" CACHE STRING "clang++ compiler" FORCE)
	set(CMAKE_C_STANDARD 11)
	set(CMAKE_CXX_STANDARD 11)
	set(CMAKE_C_EXTENSIONS OFF)
	set(CMAKE_CXX_EXTENSIONS OFF)
endif()

if(ENABLE_CLANG_FORMAT_FIX)
	find_program(CLANG_FORMAT_BIN clang-format-8)
	file(GLOB_RECURSE MY_SOURCES_C "src/*.c")
	file(GLOB_RECURSE MY_SOURCES_CPP "src/*.cpp")
	set(MY_SOURCES ${MY_SOURCES_C} ${MY_SOURCES_CPP})
	file(GLOB_RECURSE MY_HEADERS_H "include/*.h")
	file(GLOB_RECURSE MY_HEADERS_HPP "include/*.hpp")
	set(MY_HEADERS ${MY_HEADERS_H} ${MY_HEADERS_HPP})
	set(FILES_TO_FORMAT ${MY_SOURCES} ${MY_HEADERS})
	message("[CLANG_FORMAT]Formatting Files \n >>> \n ${FILES_TO_FORMAT} \n <<< \n")
	add_custom_target(call_clang_format
		ALL
		COMMAND
			${CLANG_FORMAT_BIN}
			-style=file
			-i
			${FILES_TO_FORMAT}
	)
endif()

if(ENABLE_CLANG_TIDY_FIX)
	set(
	  CMAKE_CXX_CLANG_TIDY
	  clang-tidy-8;
	  -p;
	  -modernize-use-auto;
	  -header-filter=${CMAKE_SOURCE_DIR}/include;
	  -fix;
	  -fix-errors;
	  -extra-arg=-std=c++11;
	  # -format-style=google;
	  # -checks=*;
	  # -checks=*,clang-analyzer-*,-clang-analyzer-cplusplus*;
	  -warnings-as-errors=*;
	)
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
