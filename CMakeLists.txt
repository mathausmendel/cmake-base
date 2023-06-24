cmake_minimum_required(VERSION 3.10 FATAL_ERROR)
project(cmake-base)

# Set the default C++ standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)

# Use pthreads if possible
set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
find_package(Threads)

if(CMAKE_USE_PTHREADS_INIT)
	set(CMAKE_THREAD_LIBS_INIT "pthread")
endif()

# Set the output directory for bin and lib
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")

# Disable developer warnings for CMake
set(CMAKE_SUPPRESS_DEVELOPER_WARNINGS 1 CACHE INTERNAL "No dev warnings")

# Disable Conan spitting a lot of logs
set(CONAN_CMAKE_SILENT_OUTPUT TRUE)

# Add our local modules to the path
list(APPEND CMAKE_MODULE_PATH ${CMAKE_BINARY_DIR})
list(APPEND CMAKE_PREFIX_PATH ${CMAKE_BINARY_DIR})
set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH};${PROJECT_SOURCE_DIR}/cmake")

# The default name of the generated component
set(CMAKE_INSTALL_DEFAULT_COMPONENT_NAME "default")

# Export all the compile commands
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# If ccache is installed, use to speed up the build
find_program(CCACHE_FOUND ccache HINTS /bin /usr/bin)
if(CCACHE_FOUND)
	set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
	set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
endif()

# Import conan
include(conan)

conan_cmake_configure(
	REQUIRES boost/1.82.0
	REQUIRES fmt/10.0.0
	IMPORTS "bin, *.dll -> ./bin"
	IMPORTS "lib, *.dylib* -> ./bin"
	IMPORTS "lib, *.so* -> ./bin"
	OPTIONS boost:shared=True
	GENERATORS cmake_find_package)

conan_cmake_autodetect(settings)

conan_cmake_install(PATH_OR_REFERENCE .
	BUILD missing
	BUILD b2
	REMOTE conancenter
	SETTINGS ${settings}
	SETTINGS build_type=${CMAKE_BUILD_TYPE}
	SETTINGS compiler.cppstd=gnu17)

# Set compilation flags
add_compile_options(-Wall
	-Wextra
	-Werror
	-pedantic-errors
	-fexceptions
	-ffunction-sections
	-fdata-sections
	-fasynchronous-unwind-tables
	-fno-delete-null-pointer-checks
	-fno-common
	-fno-omit-frame-pointer
	-fno-strict-aliasing
	-fno-semantic-interposition
	-fstack-protector-strong
	-fstack-clash-protection
	-flto=auto
	-fvisibility=hidden
	-fdiagnostics-color=auto
	-grecord-gcc-switches
	-g3
	-pipe
	-pthread)

# Set linker flags
add_link_options(-Wl,-O2
	-Wl,-z,noexecstack
	-Wl,-z,relro
	-Wl,-z,now
	-Wl,--no-as-needed
	-Wl,--gc-sections
	-Wl,--discard-all)

# Enable glibc assertions
add_definitions(-D_GLIBCXX_ASSERTIONS)

# Enable fortify by default
if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 12)
	# GCC 12 supports automatically initializing variables to zero
	add_compile_options(-ftrivial-auto-var-init=zero)

	# First we need to undef _FORTIFY_SOURCE as some systems
	# or libraries might have it set it previously.
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=3 ")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=3 ")
else()
	add_definitions(-D_FORTIFY_SOURCE=2)
endif()

# Use OpenMP if available
find_package(OpenMP)
if(OPENMP_FOUND)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
	set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()

set(Boost_USE_STATIC_LIBS OFF)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME OFF)

find_package(Boost COMPONENTS
	unit_test_framework
	REQUIRED)

add_subdirectory(src)
add_subdirectory(test)

set(CPACK_GENERATOR "RPM")
set(CPACK_PACKAGE_NAME "${PROJECT_NAME}")
set(CPACK_PACKAGE_VENDOR "Foo Bar LLC")
set(CPACK_PACKAGE_CONTACT "foo@bar.com")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "CMake base structure for standard C or C++ projects")
set(CPACK_PACKAGE_VERSION_MAJOR 1)
set(CPACK_PACKAGE_VERSION_MINOR 0)
set(CPACK_PACKAGE_VERSION_PATCH 0)
set(CPACK_PACKAGE_VERSION "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")
set(CPACK_COMPONENTS_ALL default)
set(CPACK_STRIP_FILES TRUE)

include(CPack)
include(CTest)
