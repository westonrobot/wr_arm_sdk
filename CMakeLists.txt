cmake_minimum_required(VERSION 3.10.2)

find_program(CCACHE_PROGRAM ccache)
if(CCACHE_PROGRAM)
    message(STATUS "Found ccache")
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
endif()

project(rm_arm_sdk VERSION 4.2.2)

## Project Options
option(BUILD_TESTING "Build tests for the package" OFF)

## Generate symbols for IDE indexer
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

## Additional cmake module path
set(CONAN_CMAKE_SETUP ${CMAKE_BINARY_DIR}/conan_paths.cmake)
if (EXISTS ${CONAN_CMAKE_SETUP} AND NOT IS_DIRECTORY ${CONAN_CMAKE_SETUP})
    include(${CONAN_CMAKE_SETUP})
endif()
set(USER_CMAKE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
list(APPEND CMAKE_MODULE_PATH "${USER_CMAKE_PATH}/modules")
list(APPEND CMAKE_PREFIX_PATH "/usr/lib/${CMAKE_SYSTEM_PROCESSOR}-linux-gnu/cmake")
list(APPEND CMAKE_PREFIX_PATH "/opt/weston_robot/lib/cmake")

## Set compiler to use c++ 11 features
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

## Choose build type
# set(CMAKE_BUILD_TYPE Release)
# set(CMAKE_BUILD_TYPE Debug)
set(default_build_type "Debug")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
      STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

## Use GNUInstallDirs to install libraries into correct
# locations on all platforms.
include(GNUInstallDirs)

## Put all binary files into /bin and libraries into /lib
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_BINDIR})

# Build tests
if(PROJECT_NAME STREQUAL CMAKE_PROJECT_NAME AND BUILD_TESTING)
    enable_testing()
    include(GoogleTest)
    set(BUILD_TESTS ON)
    message(STATUS "Tests will be built")
else()
    set(BUILD_TESTS OFF)
    message(STATUS "Tests will not be built")
endif()

find_package(Threads REQUIRED)

## Add library binaries
if(CMAKE_BUILD_TYPE MATCHES Release)
    set(LIB_RELEASE_TYPE "release")
    set(LIB_RELEASE_NAME "libRM_Service.so")
else()
    set(LIB_RELEASE_TYPE "debug")
    set(LIB_RELEASE_NAME "libRM_Serviced.so")
endif()


if(CMAKE_SYSTEM_PROCESSOR MATCHES "arm")
    set(LIB_ARCH_TYPE "arm64")
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64")
    set(LIB_ARCH_TYPE "amd64")
else()
    message(FATAL_ERROR "Unsupported architecture: ${CMAKE_SYSTEM_PROCESSOR}")
endif()

set(LIB_FULL_PATH ${CMAKE_CURRENT_SOURCE_DIR}/lib/${LIB_RELEASE_TYPE}/${LIB_ARCH_TYPE}/${LIB_RELEASE_NAME})
message(STATUS "Library path: ${LIB_FULL_PATH}")

add_library(rm_arm_sdk SHARED IMPORTED GLOBAL)
set_target_properties(rm_arm_sdk PROPERTIES 
    IMPORTED_LOCATION ${LIB_FULL_PATH})
set_target_properties(rm_arm_sdk PROPERTIES IMPORTED_NO_SONAME TRUE)
target_link_libraries(rm_arm_sdk INTERFACE 
    Threads::Threads)
target_link_directories(rm_arm_sdk INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/lib)
target_include_directories(rm_arm_sdk INTERFACE
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/rm_arm>
	$<INSTALL_INTERFACE:include>)

add_subdirectory(demo)

## Installation setup
message(STATUS "Project will be installed to ${CMAKE_INSTALL_PREFIX} with 'make install'")

# Offer the user the choice of overriding the installation directories
set(INSTALL_LIBDIR ${CMAKE_INSTALL_LIBDIR} CACHE PATH "Installation directory for libraries")
set(INSTALL_BINDIR ${CMAKE_INSTALL_BINDIR} CACHE PATH "Installation directory for executables")
set(INSTALL_INCLUDEDIR ${CMAKE_INSTALL_INCLUDEDIR} CACHE PATH "Installation directory for header files")
if(WIN32 AND NOT CYGWIN)
    set(DEF_INSTALL_CMAKEDIR CMake)
else()
    set(DEF_INSTALL_CMAKEDIR share/cmake/${PROJECT_NAME})
endif()
set(INSTALL_CMAKEDIR ${DEF_INSTALL_CMAKEDIR} CACHE PATH "Installation directory for CMake files")

# print installation path to user
foreach(p LIB BIN INCLUDE CMAKE)
    file(TO_NATIVE_PATH ${CMAKE_INSTALL_PREFIX}/${INSTALL_${p}DIR} _path)
    message(STATUS "Installation path for ${p}: ${_path}")
    unset(_path)
endforeach()

# # targets to install
# install(TARGETS rm_arm_sdk
#     EXPORT rm_arm_sdkTargets
#     LIBRARY DESTINATION lib
#     ARCHIVE DESTINATION lib
#     RUNTIME DESTINATION bin
#     INCLUDES DESTINATION include)

# install(DIRECTORY include/
#     DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# install(DIRECTORY lib/
#     DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# # export target configuration 
# include(CMakePackageConfigHelpers)
# write_basic_package_version_file(
#     rm_arm_sdkConfigVersion.cmake
#     VERSION "${${PROJECT_NAME}_VERSION_MAJOR}.${${PROJECT_NAME}_VERSION_MINOR}.${${PROJECT_NAME}_VERSION_PATCH}"
#     COMPATIBILITY SameMajorVersion)

# install(EXPORT rm_arm_sdkTargets
#     FILE rm_arm_sdkTargets.cmake
#     NAMESPACE westonrobot::
#     DESTINATION lib/cmake/rm_arm_sdk)

# configure_file(cmake/rm_arm_sdkConfig.cmake.in rm_arm_sdkConfig.cmake @ONLY)
# install(FILES "${CMAKE_CURRENT_BINARY_DIR}/rm_arm_sdkConfig.cmake"
#               "${CMAKE_CURRENT_BINARY_DIR}/rm_arm_sdkConfigVersion.cmake"
#         DESTINATION lib/cmake/rm_arm_sdk)

# Packaging support
set(CPACK_PACKAGE_VENDOR "Weston Robot")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Weston Robot RM Manipulator SDK")
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_PACKAGING_INSTALL_PREFIX "/opt/weston_robot")
# set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.md")

set(CPACK_GENERATOR "DEB")
set(CPACK_DEBIAN_FILE_NAME DEB-DEFAULT)
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Ruixiang Du") 
# set(CPACK_DEBIAN_PACKAGE_DEPENDS )
set(CPACK_SOURCE_IGNORE_FILES .git dist .*build.* \\\\.DS_Store)
include(CPack)