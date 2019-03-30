cmake_minimum_required (VERSION 3.7)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/cmake)

project(tlspool C)

include (FeatureSummary)
find_package (ARPA2CM 0.5 QUIET NO_MODULE)
set_package_properties (ARPA2CM PROPERTIES
    DESCRIPTION "CMake modules for ARPA2 projects"
    TYPE REQUIRED
    URL "https://github.com/arpa2/arpa2cm/"
    PURPOSE "Required for the CMake build system for ${PROJECT}"
)
if (ARPA2CM_FOUND)
    set (CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${ARPA2CM_MODULE_PATH})
else()
    feature_summary (WHAT ALL)
    message (FATAL_ERROR "ARPA2CM is required.")
endif()


#
# Choices and Options
#

include(MacroEnsureOutOfSourceBuild)
include(MacroGitVersionInfo)

include(CTest)
include(FeatureSummary)
include(GNUInstallDirs)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

add_definitions(-fPIC)

enable_testing ()


#
# Subdirectories
#

add_subdirectory (mit.src)


