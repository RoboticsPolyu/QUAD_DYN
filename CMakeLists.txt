cmake_minimum_required(VERSION 3.10)
project(IPN_MPC)

option(BUILD_SHARED_LIBS "Build shared gtsam library, instead of static" ON)
option(BUILD_WITH_ROS    "Enable Ros Support" OFF)

set(DEFAULT_BUILD_TYPE "Release")
if (NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to '${DEFAULT_BUILD_TYPE}' as none was specified.")
    set(CMAKE_BUILD_TYPE "${DEFAULT_BUILD_TYPE}" CACHE
            STRING "Choose the type of build." FORCE)
    # Set the possible values of build type for cmake-gui
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
            "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif ()

set(CMAKE_CXX_FLAGS "-std=c++14 -g -Wall")

if(APPLE)
    set(ENV{CC} "/usr/local/opt/llvm/bin/clang-8")
    set(ENV{CXX} "/usr/local/opt/llvm/bin/clang++")
    set(ENV{LDFLAGS} "-L/usr/local/opt/llvm/lib")
    set(ENV{CPPFLAGS} "-I/usr/local/opt/llvm/include")
endif()


set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS} -fpermissive -Wno-reorder" CACHE STRING "" FORCE)
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -fpermissive -DNDEBUG -Wno-reorder -O2" CACHE STRING "" FORCE)

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")
list(APPEND CMAKE_MODULE_PATH "/usr/local/cmake/lib/cmake")

if(BUILD_WITH_ROS)
find_package(catkin REQUIRED COMPONENTS
  roscpp message_generation std_msgs geometry_msgs
)

add_message_files(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/msg" FILES IMU.msg INPUT.msg Rsm.msg)
generate_messages(DEPENDENCIES std_msgs geometry_msgs)
include_directories(${catkin_INCLUDE_DIRS})
catkin_package(
    CATKIN_DEPENDS 
    roscpp 
    mavros_msgs
    message_runtime 
    std_msgs 
    geometry_msgs)
endif()

# Eigen
find_package(Eigen3 3.3 REQUIRED NO_MODULE)
set(EIGEN3_INCLUDE_DIR "/usr/local/include/eigen3")
include_directories(${EIGEN3_INCLUDE_DIR})

# gtsam
find_package(GTSAM REQUIRED)
include_directories(${GTSAM_INCLUDE_DIR})

# include
include_directories(include)

# Pangolin
# find_package(Pangolin 0.8 REQUIRED)
# include_directories(${Pangolin_INCLUDE_DIRS})

# Find yaml-cpp
find_package(yaml-cpp)
include_directories(${YAML_CPP_INCLUDE_DIRS})


add_library(quadrotor_dynamics 
            include/gtsam_wrapper.h
            src/calibration/Calibration_factor.cpp
            src/calibration/Calibration_inertial.cpp
            src/GTSAMUtils.cpp
            src/Marginalization.cpp)

target_link_libraries(quadrotor_dynamics PUBLIC gtsam gtsam_unstable ${Pangolin_LIBRARIES} ${YAML_CPP_LIBRARIES})

# CALIBRATION
add_executable(Aerial_vehicle_calibration_BLACK app/Aerial_vehicle_calibration_BLACK.cpp)
target_link_libraries(Aerial_vehicle_calibration_BLACK gtsam quadrotor_dynamics ${Pangolin_LIBRARIES} ${YAML_CPP_LIBRARIES})

add_executable(Quadrotor_dynamics_calibration app/Quadrotor_dynamics_calibration.cpp)
target_link_libraries(Quadrotor_dynamics_calibration gtsam quadrotor_dynamics ${Pangolin_LIBRARIES} ${YAML_CPP_LIBRARIES})

# add_executable(Aerial_vehicle_calibration_sim app/Aerial_vehicle_calibration_sim.cpp)
# target_link_libraries(Aerial_vehicle_calibration_sim gtsam quadrotor_dynamics ${Pangolin_LIBRARIES} ${YAML_CPP_LIBRARIES})

add_executable(Aerial_vehicle_calibration_NBEM app/Aerial_vehicle_calibration_NBEM.cpp)
target_link_libraries(Aerial_vehicle_calibration_NBEM quadrotor_dynamics ${Pangolin_LIBRARIES} ${YAML_CPP_LIBRARIES})

add_executable(Motor_calibration app/Motor_calibration.cpp)
target_link_libraries(Motor_calibration quadrotor_dynamics ${Pangolin_LIBRARIES} ${YAML_CPP_LIBRARIES})

add_executable(Online_Calibration_BLACK app/Online_Calibration_BLACK.cpp)
target_link_libraries(Online_Calibration_BLACK quadrotor_dynamics ${Pangolin_LIBRARIES} ${YAML_CPP_LIBRARIES})

add_executable(Batch_Opt_Dynamics_NBEM app/Batch_Opt_Dynamics_NBEM.cpp)
target_link_libraries(Batch_Opt_Dynamics_NBEM quadrotor_dynamics ${Pangolin_LIBRARIES} ${YAML_CPP_LIBRARIES})

add_executable(Batch_Fixed_Lag_Dynamics_NBEM app/Batch_Fixed_Lag_Dynamics_NBEM.cpp)
target_link_libraries(Batch_Fixed_Lag_Dynamics_NBEM quadrotor_dynamics ${Pangolin_LIBRARIES} ${YAML_CPP_LIBRARIES})


