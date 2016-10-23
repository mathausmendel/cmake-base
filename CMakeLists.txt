cmake_minimum_required(VERSION 2.8 FATAL_ERROR)
project(cmake-base)

set(CPACK_PACKAGE_CONTACT "Foo Bar")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "CMake base structure for standard C or C++ projects")
set(CPACK_PACKAGE_VERSION_MAJOR 1)
set(CPACK_PACKAGE_VERSION_MINOR 0)
set(CPACK_PACKAGE_VERSION_PATCH 0)
set(CPACK_PACKAGE_VERSION "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")

include(CPack)
include(CTest)

add_subdirectory(src)
add_subdirectory(test)
