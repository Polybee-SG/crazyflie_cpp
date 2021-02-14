cmake_minimum_required(VERSION 3.5)
project(crazyflie_cpp)

#---- Enable C++14 ----
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_compile_options(-Wall -Wextra -Werror)

set(CRAZYFLIE_CPP_LIBRARY ${PROJECT_NAME})

# Update the policy setting to avoid an error when loading the ament_cmake package
# at the current cmake version level
if(POLICY CMP0057)
  cmake_policy(SET CMP0057 NEW)
endif()

find_library(USB_LIB usb-1.0)
find_package(ament_cmake QUIET)

if ( ament_cmake_FOUND )

  # Not adding -DUSING_ROS since xml_parsing.cpp hasn't been ported to ROS2

  message(STATUS "------------------------------------------")
  message(STATUS "crazyflie_cpp is being built using AMENT.")
  message(STATUS "------------------------------------------")

  set(BUILD_TOOL_INCLUDE_DIRS ${ament_INCLUDE_DIRS})

elseif( CATKIN_DEVEL_PREFIX OR CATKIN_BUILD_BINARY_PACKAGE)

  set(catkin_FOUND 1)
  add_definitions( -DUSING_ROS )
  find_package(catkin REQUIRED COMPONENTS roslib)

  message(STATUS "------------------------------------------")
  message(STATUS "crazyflie_cpp is being built using CATKIN.")
  message(STATUS "------------------------------------------")

  catkin_package(
    INCLUDE_DIRS include # do not include "3rdparty" here
    LIBRARIES ${CRAZYFLIE_CPP_LIBRARY}
    CATKIN_DEPENDS roslib
  )

  set(BUILD_TOOL_INCLUDE_DIRS ${catkin_INCLUDE_DIRS})

endif()

if(ament_cmake_FOUND)
  set( CRAZYFLIE_CPP_LIB_DESTINATION   lib )
  set( CRAZYFLIE_CPP_INC_DESTINATION   include )
  set( CRAZYFLIE_CPP_BIN_DESTINATION   bin )

  ament_export_include_directories(include)
  ament_export_libraries(${CRAZYFLIE_CPP_LIBRARY})
  ament_package()
elseif(catkin_FOUND)
  set( CRAZYFLIE_CPP_LIB_DESTINATION   ${CATKIN_PACKAGE_LIB_DESTINATION} )
  set( CRAZYFLIE_CPP_INC_DESTINATION   ${CATKIN_GLOBAL_INCLUDE_DESTINATION} )
  set( CRAZYFLIE_CPP_BIN_DESTINATION   ${CATKIN_GLOBAL_BIN_DESTINATION} )
else()
  set( CRAZYFLIE_CPP_LIB_DESTINATION   lib )
  set( CRAZYFLIE_CPP_INC_DESTINATION   include )
  set( CRAZYFLIE_CPP_BIN_DESTINATION   bin )

  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CRAZYFLIE_CPP_BIN_DESTINATION}" )
  set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CRAZYFLIE_CPP_LIB_DESTINATION}" )
  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CRAZYFLIE_CPP_BIN_DESTINATION}" )
endif()

message( STATUS "CRAZYFLIE_CPP_LIB_DESTINATION:   ${CRAZYFLIE_CPP_LIB_DESTINATION} " )
message( STATUS "CRAZYFLIE_CPP_BIN_DESTINATION:   ${CRAZYFLIE_CPP_BIN_DESTINATION} " )
message( STATUS "CMAKE_RUNTIME_OUTPUT_DIRECTORY:  ${CMAKE_RUNTIME_OUTPUT_DIRECTORY} " )
message( STATUS "CMAKE_LIBRARY_OUTPUT_DIRECTORY:  ${CMAKE_LIBRARY_OUTPUT_DIRECTORY} " )
message( STATUS "CMAKE_ARCHIVE_OUTPUT_DIRECTORY:  ${CMAKE_ARCHIVE_OUTPUT_DIRECTORY} " )

#############################################################
# LIBRARY

add_library(${CRAZYFLIE_CPP_LIBRARY}
  src/Crazyflie.cpp
  src/CrazyflieUSB.cpp
  src/Crazyradio.cpp
  src/crtp.cpp
  src/USBDevice.cpp
  src/ITransport.cpp
)

if(catkin_FOUND)
  target_link_libraries(${CRAZYFLIE_CPP_LIBRARY} PUBLIC
    ${catkin_LIBRARIES}
  )
endif()

target_link_libraries(${CRAZYFLIE_CPP_LIBRARY} PRIVATE
  ${USB_LIB}
)

target_include_directories(${CRAZYFLIE_CPP_LIBRARY} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
  ${BUILD_TOOL_INCLUDE_DIRS}
)

######################################################
# INSTALL

INSTALL(TARGETS ${CRAZYFLIE_CPP_LIBRARY}
  EXPORT CrazyflieCppConfig
  ARCHIVE DESTINATION ${CRAZYFLIE_CPP_LIB_DESTINATION}
  LIBRARY DESTINATION ${CRAZYFLIE_CPP_LIB_DESTINATION}
  RUNTIME DESTINATION ${CRAZYFLIE_CPP_BIN_DESTINATION}
)

INSTALL( DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
  DESTINATION ${CRAZYFLIE_CPP_INC_DESTINATION}
  FILES_MATCHING PATTERN "*.h*")

install(EXPORT CrazyflieCppConfig
  DESTINATION "${CRAZYFLIE_CPP_LIB_DESTINATION}/CrazyflieCpp/cmake"
)

export(TARGETS ${PROJECT_NAME}
  FILE "${CMAKE_CURRENT_BINARY_DIR}/CrazyflieCppConfig.cmake")

export(PACKAGE ${PROJECT_NAME})
