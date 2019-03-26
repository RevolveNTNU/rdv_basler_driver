cmake_minimum_required(VERSION 2.8.3)
project(rdv_basler_driver)

add_definitions("-std=gnu++11")
#set(CMAKE_CXX_FLAGS "-g -Wall -Wno-unknown-pragmas -Wno-delete-non-virtual-dtor -Wno-unused-variable")
set(
    CATKIN_COMPONENTS
     actionlib
     camera_control_msgs
     camera_info_manager
     cv_bridge
     diagnostic_updater
     image_geometry
     image_transport
     roscpp
     roslaunch
     sensor_msgs
)

find_package(Pylon QUIET)
if (NOT ${Pylon_FOUND})
    include("${CMAKE_CURRENT_SOURCE_DIR}/cmake/FindPylon.cmake")
endif()
find_package(
    catkin REQUIRED
    COMPONENTS
     ${CATKIN_COMPONENTS}
     #std_srvs
     roslint
)

catkin_package(
    INCLUDE_DIRS
     include
    LIBRARIES
     pylon_camera
    CATKIN_DEPENDS
     ${CATKIN_COMPONENTS}
)

set(
    ROSLINT_CPP_OPTS
     "--extensions=cpp,h,hpp" "--filter=-runtime/references,-readability/todo,-build/include_what_you_use"
)

roslint_cpp(
    src/pylon_camera/binary_exposure_search.cpp
    src/pylon_camera/encoding_conversions.cpp
    src/pylon_camera/main.cpp
    src/pylon_camera/pylon_camera_node.cpp
    src/pylon_camera/pylon_camera_parameter.cpp
    src/pylon_camera/pylon_camera.cpp
    src/pylon_camera/write_device_user_id_to_camera.cpp
    include/pylon_camera/binary_exposure_search.h
    include/pylon_camera/encoding_conversions.h
    include/pylon_camera/pylon_camera_node.h
    include/pylon_camera/pylon_camera_parameter.h
    include/pylon_camera/pylon_camera.h
    include/pylon_camera/internal/pylon_camera.h
    include/pylon_camera/internal/impl/pylon_camera_base.hpp
    include/pylon_camera/internal/impl/pylon_camera_dart.hpp
    include/pylon_camera/internal/impl/pylon_camera_gige.hpp
    include/pylon_camera/internal/impl/pylon_camera_usb.hpp
)

roslaunch_add_file_check(launch)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${catkin_INCLUDE_DIRS}
    ${Pylon_INCLUDE_DIRS}
)

# Add library
add_library(
    pylon_camera
     src/pylon_camera/binary_exposure_search.cpp
     src/pylon_camera/encoding_conversions.cpp
     src/pylon_camera/pylon_camera.cpp
     src/pylon_camera/pylon_camera_node.cpp
     src/pylon_camera/pylon_camera_parameter.cpp
)

target_link_libraries(
    pylon_camera
     ${catkin_LIBRARIES}
     ${Pylon_LIBRARIES}
)

add_dependencies(
    pylon_camera
     ${catkin_EXPORTED_TARGETS}
)

# Add pylon_camera_node
add_executable(
    pylon_camera_node
     src/pylon_camera/main.cpp
)

target_link_libraries(
    pylon_camera_node
     pylon_camera
)

add_executable(
    write_device_user_id_to_camera
     src/pylon_camera/write_device_user_id_to_camera.cpp
)

target_link_libraries(
    write_device_user_id_to_camera
     ${Pylon_LIBRARIES}
)

add_dependencies(
    pylon_camera_node
     ${catkin_EXPORTED_TARGETS}
)

catkin_python_setup()

install(
    DIRECTORY
     launch/
    DESTINATION
     ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
    FILES_MATCHING PATTERN "*.launch"
)

install(
    DIRECTORY
     config/
    DESTINATION
     ${CATKIN_PACKAGE_SHARE_DESTINATION}/config
    FILES_MATCHING PATTERN "*.yaml"
)

install(
    PROGRAMS
     scripts/file_sequencer.py
     scripts/grab_and_save_image_action_server.py
     scripts/result_bag_to_action.py
     scripts/sequence_to_file.py
     scripts/toggle_camera
    DESTINATION
     ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(
    TARGETS
     pylon_camera
     pylon_camera_node
     write_device_user_id_to_camera
    LIBRARY DESTINATION
     ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION
     ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(
    DIRECTORY
     include/pylon_camera/
    DESTINATION
     ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
    FILES_MATCHING PATTERN
     "*.h"
    PATTERN "internal" EXCLUDE
)

## Testing ##
# All Jenkins-Tests are now in the pylon_camera_tests-pkg
############

###############
## QtCreator ##
###############
# entry for QtCreator to show all files
file(GLOB children ${CMAKE_CURRENT_SOURCE_DIR}/*)
foreach(child ${children})
  if(IS_DIRECTORY ${child})
    file(GLOB_RECURSE dir_files "${child}/*")
    list(APPEND pylon_camera_extra_files ${dir_files})
  endif()
endforeach()
add_custom_target(dummy_pylon_camera SOURCES ${pylon_camera_extra_files})

