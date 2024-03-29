cmake_minimum_required(VERSION 2.8.3)
project(xqserial_server)

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
if(COMPILER_SUPPORTS_CXX11)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
   add_definitions(-DCOMPILEDWITHC11)
   message(STATUS "Using flag -std=c++11.")
else()
   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

find_package(catkin REQUIRED COMPONENTS
  nav_msgs
  roscpp
  rospy
  sensor_msgs
  std_msgs
  geometry_msgs
  tf
  message_generation
  xiaoqiang_log
  galileo_serial_server
)

find_package(Eigen3 REQUIRED)

add_service_files(
  FILES
  Shutdown.srv
)

generate_messages(
  DEPENDENCIES
  std_msgs
)

set(BOOST_LIBS thread date_time system)
find_package(Boost COMPONENTS ${BOOST_LIBS} REQUIRED)
find_package(Threads REQUIRED)
find_package(PkgConfig REQUIRED)

pkg_check_modules(JSONCPP REQUIRED jsoncpp)

catkin_package(
  INCLUDE_DIRS include
  CATKIN_DEPENDS nav_msgs roscpp rospy sensor_msgs std_msgs geometry_msgs xiaoqiang_log galileo_serial_server message_runtime
)

include_directories(
  include
  ${Boost_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
  ${JSONCPP_INCLUDE_DIRS}
  ${EIGEN_INCLUDE_DIRS}
  "/usr/include/eigen3"
)
add_definitions(${EIGEN_DEFINITIONS})

set(SRCS src/main.cpp src/AsyncSerial.cpp src/DiffDriverController.cpp src/StatusPublisher.cpp src/crc16.cpp)
add_executable(${PROJECT_NAME} ${SRCS})
add_dependencies(
  ${PROJECT_NAME}
  ${catkin_EXPORTED_TARGETS} xiaoqiang_log_cpp
)

target_link_libraries(${PROJECT_NAME}
  ${catkin_LIBRARIES}
  ${JSONCPP_LIBRARIES}
)
set_target_properties(${PROJECT_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME} PREFIX "")

install(
  TARGETS ${PROJECT_NAME} ${PROJECT_NAME}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(
  DIRECTORY launch
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(
  DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)
