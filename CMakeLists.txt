
###########################################################
#   Top level CMakeLists.txt file for the MOOS Project
#   pnewman@robots.ox.ac.uk
###########################################################


CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

if(COMMAND cmake_policy)
   cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

set(PROJECTNAME MOOSGeodesy)
project(${PROJECTNAME})

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lib)


set(CMakeScripts_MODULE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
set(CMAKE_MODULE_PATH ${CMakeScripts_MODULE_DIR})


# Turn on all compiler warnings
include(EnableAllWarnings)

# Bring in helper functions for dealing with CACHE INTERNAL variables
include(CacheInternalHelpers)

# We want to create dynamic libraries
set(BUILD_SHARED_LIBS true)

# Enable Doxygen build with 'make doxygen'
option(ENABLE_DOXYGEN "Enable a 'make doc' target for Doxygen documentation")
if (ENABLE_DOXYGEN)
    include(UseDoxygen)
endif()


###########  SETTING UP PREPROCESSOR ################
include(PlatformDefines)


#####################################################
# Toby Schneider's request for fPIC on amd64
IF("${CMAKE_SYSTEM_PROCESSOR}" STREQUAL "x86_64")
 SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC"  )
ENDIF("${CMAKE_SYSTEM_PROCESSOR}" STREQUAL "x86_64")


add_subdirectory(libMOOSGeodesy)


#########################
# Config Script Creation
#########################

# Provide a cmake config script for the build directory
export(
    TARGETS MOOSGeodesy
    NAMESPACE MOOS::
    FILE UseMOOSGeodesy.cmake
)

# Support existing projects that expect to find MOOSGeodesy_LIBRARIES and
# MOOSGeodesy_INCLUDE_DIRS variables.
set(MOOSGeodesy_LIBRARIES MOOS::MOOSGeodesy)

# Generate the config file for other projects to find the MOOS build dir
set(CONFIG_FILE "${PROJECT_BINARY_DIR}/MOOSGeodesyConfig.cmake")
configure_file(
    "cmake/ProjectConfig.cmake.in"
    "${CONFIG_FILE}"
    @ONLY
)

#########################
# Install target
#########################

# Provide a cmake config script for the install directory
set(PROJECT_CONFIG_PATH "lib/cmake/MOOSGeodesy")
install(
    EXPORT MOOSGeodesy
    NAMESPACE MOOS::
    FILE UseMOOSGeodesy.cmake
    DESTINATION ${PROJECT_CONFIG_PATH}
)

# Headers are all combined into one common 'include' dir when MOOS is installed.
# This path needs to be relative to the package config file itself.
set(MOOSGeodesy_INCLUDE_DIRS "\${CMAKE_CURRENT_LIST_DIR}/../../../include")
set(CONFIG_FILE "${PROJECT_BINARY_DIR}/MOOSGeodesyInstallConfig.cmake")
configure_file( "cmake/ProjectConfig.cmake.in" ${CONFIG_FILE} @ONLY )
install(
    FILES "${CONFIG_FILE}"
    RENAME MOOSGeodesyConfig.cmake
    DESTINATION "${PROJECT_CONFIG_PATH}"
)

# Save a version file in the project's install directory
install(FILES ${VERSION_FILE} DESTINATION "${PROJECT_CONFIG_PATH}")

option(ENABLE_EXPORT
  "Cause CMake to record this build so other projects can find it " ON)
if(ENABLE_EXPORT)
    if ("${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION}" GREATER 2.7)
        # Stick build location in the central CMake user package registry, so that
        # it may be easily found.
        message(STATUS "Location of this build will be exported. "
            "Set ENABLE_EXPORT to OFF if you don't want this and "
            "delete ~/.cmake/projects/MOOS "
        )
        export(PACKAGE ${PROJECTNAME})
    endif()
endif()
