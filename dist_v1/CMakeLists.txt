

##########################################################
#                 Template CMakeLists                    #
##########################################################
#                                                        #
# Use this CMakeLists file to create build environments  #
# for your C/C++ project                                 #
#                                                        #
# Put your source in the subfolder ./src and run CMake   #
# to configure the project for your platform.            #
#                                                        #
# If using Makefiles, be sure to set CMAKE_BUILD_TYPE to # 
# select different build configurations.                 #
# Possible values:                                       #
# - RELEASE - fully optimized for speed                  #
# - DEBUG - unoptimized, with debugging symbols          #
# - RELWITHDEBUGINFO - optimized, with debugging symbols #
# - MINSIZEREL - optimized for size                      #
# - [not set] - unoptimized                              #
##########################################################

# Minimum version of CMake required.
cmake_minimum_required(VERSION 2.8)

# Set the name of your project. Change the next two lines to match your current project name.
project(dist_v1)
set(EXE_NAME dist_v1)

## Collect all the sources in the src folder.
# Note that header files don't *need* to be included
# here, but if we don't, they will not show up in
# the Visual Studio explorer.
file(GLOB SOURCES 
  "${CMAKE_CURRENT_SOURCE_DIR}/*.cu"
  "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp"
  "${CMAKE_CURRENT_SOURCE_DIR}/*.c"
  "${CMAKE_CURRENT_SOURCE_DIR}/*.h"
  "${CMAKE_CURRENT_SOURCE_DIR}/*.cuh")

# Create the executable from all available sources
add_executable(${EXE_NAME} ${SOURCES})

# Set the startup project for Visual Studio
if(NOT CMAKE_VERSION VERSION_LESS "3.6.0")
  set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${EXE_NAME})
endif()