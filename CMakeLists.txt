cmake_minimum_required(VERSION 3.5)
project(lpeg VERSION 1.0 LANGUAGES C)

#Make sure that custom modules like FindRapidJSON are found
list(INSERT CMAKE_MODULE_PATH 0 ${CMAKE_SOURCE_DIR}/cmake)

##############################################
# Declare dependencies
find_package(LuaJIT REQUIRED MODULE)

##############################################
# Create target and set properties

add_library(lpeg
  src/lpcap.c
  src/lpcode.c
  src/lpprint.c
  src/lptree.c
  src/lpvm.c
)

add_library(LPEG::lpeg ALIAS lpeg)

target_include_directories(lpeg
  PUBLIC
    $<INSTALL_INTERFACE:lpeg>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${LUA_INCLUDE_DIR}
)

target_link_libraries(lpeg
  PUBLIC
    luajit
)

##############################################
# Installation instructions

include(GNUInstallDirs)
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/LPEG)

install(TARGETS lpeg
  EXPORT lpeg-targets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

#This is required so that the exported target has the name LPEG and not lpeg
set_target_properties(lpeg PROPERTIES EXPORT_NAME LPEG)

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

#Export the targets to a script
install(EXPORT lpeg-targets
  FILE
    LPEGTargets.cmake
  NAMESPACE
    LPEG::
  DESTINATION
    ${INSTALL_CONFIGDIR}
)

#Create a ConfigVersion.cmake file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/LPEGConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(${CMAKE_CURRENT_LIST_DIR}/cmake/LPEGConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/LPEGConfig.cmake
  INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)

#Install the config, configversion and custom find modules
install(FILES
  ${CMAKE_CURRENT_LIST_DIR}/cmake/FindLuaJIT.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/LPEGConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/LPEGConfigVersion.cmake
  DESTINATION ${INSTALL_CONFIGDIR}
)

##############################################
## Exporting from the build tree
configure_file(${CMAKE_CURRENT_LIST_DIR}/cmake/FindLuaJIT.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/FindLuaJIT.cmake
  COPYONLY)

export(EXPORT lpeg-targets FILE ${CMAKE_CURRENT_BINARY_DIR}/LPEGTargets.cmake NAMESPACE LPEG::)

#Register package in user's package registry
export(PACKAGE LPEG)
