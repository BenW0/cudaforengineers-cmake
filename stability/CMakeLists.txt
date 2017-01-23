

##########################################################
#               Template CUDA CMakeLists                 #
#             with OpenGL, GLUT, and GLEW                #
##########################################################
#                                                        #
# Use this CMakeLists file to create build environments  #
# for your C/C++ and CUDA project with GL interop.       #
#                                                        #
# Put your source in the subfolder ./src and run CMake   #
# to configure the project for your platform. The process#
# will use the ../libs folder, which contains source and #
# binaries for FreeGLUT and GLEW, as well as some CMake  #
# scripts.                                               #
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
project(stability)
set(EXE_NAME stability)

## Some compilation options you may want to change
set(CUDA_COMPUTE_CAPABILITY "30") 
		# Set the compute capability to 3.0 (valid values are 20, 30, 32, 35, 37, 50, 52, 53, 60, 61, 62)
set(CUDA_FAST_MATH TRUE)  # GPU code only (does not affect CPU)
set(CUDA_VERBOSE_PTX FALSE
		CACHE BOOL "Show Verbose Kernel Info During Compilation")
set(CUDA_VERBOSE_BUILD FALSE
		CACHE BOOL "Print out the commands run while compiling the CUDA source files.")


# Include some scripts we will need to configure CUDA and find local copies of GLUT and GLEW
set(MY_CMAKE_SCRIPTS ${CMAKE_CURRENT_SOURCE_DIR}/../libs)
include(${MY_CMAKE_SCRIPTS}/ParallelUtils.cmake)
list(APPEND CMAKE_MODULE_PATH "${MY_CMAKE_SCRIPTS}")


## Turn on CUDA
enable_cuda_support()

## Try to find the OpenGL, GLUT, and GLEW Libraries. 
# First use the default CMake finder in case they are
# installed on the system, then fall back on local copies (Windows only)
find_package(OpenGL REQUIRED)
find_package(GLUT QUIET)
if(NOT GLUT_FOUND)
	find_package(GLUTlocal REQUIRED)
endif()
find_package(GLEW QUIET)
if(NOT GLEW_FOUND)
	find_package(GLEWlocal REQUIRED)
endif()

## Collect all the sources in the current folder.
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
cuda_add_executable(${EXE_NAME} ${SOURCES})

# Set the startup project for Visual Studio
if(NOT CMAKE_VERSION VERSION_LESS "3.6.0")
  set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${EXE_NAME})
endif()

# link with shared common headers
cuda_include_directories(${CMAKE_CURRENT_SOURCE_DIR}/../include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/../include)

# link with OpenGL
cuda_include_directories(${OPENGL_INCLUDE_DIR})
include_directories(${OPENGL_INCLUDE_DIR})
target_link_libraries(${EXE_NAME} ${OPENGL_LIBRARIES})

# link with GLUT
include_directories(${GLUT_INCLUDE_DIR})
target_link_libraries(${EXE_NAME} ${GLUT_LIBRARIES}) 

# link with GLEW
include_directories(${GLEW_INCLUDE_DIRS})
target_link_libraries(${EXE_NAME} ${GLEW_LIBRARIES})

# Copy the dynamic libraries to the compilation folder as a post-build step (Windows-only in case we are using local libraries)
if(WIN32)
	if(DEFINED GLUT_RUNTIME_LIBRARY)
		add_custom_command(TARGET ${EXE_NAME} POST_BUILD
				COMMAND ${CMAKE_COMMAND} -E copy_if_different
					"${GLUT_RUNTIME_LIBRARY}"
					"\"\$\(OutDir\)/\"")
	endif()
	if(DEFINED GLEW_RUNTIME_LIBRARY)
		add_custom_command(TARGET ${EXE_NAME} POST_BUILD
				COMMAND ${CMAKE_COMMAND} -E copy_if_different
					"${GLEW_RUNTIME_LIBRARY}"
					"\"\$\(OutDir\)/\"")
	endif()
endif()