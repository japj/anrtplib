# CMakeList.txt : Top-level CMake project file, do global configuration
# and include sub-projects here.
#
cmake_minimum_required (VERSION 3.8)

#
# Project configuration
#
set(ANRTPLIB_VERSION_MAJOR 0)
set(ANRTPLIB_VERSION_MINOR 1)
set(ANRTPLIB_VERSION_PATCH 0)

project(
    anrtplib
    VERSION ${ANRTPLIB_VERSION_MAJOR}.${ANRTPLIB_VERSION_MINOR}.${ANRTPLIB_VERSION_PATCH}
    DESCRIPTION "Another RTP Library"
    HOMEPAGE_URL "https://github.com/japj/anrtplib"
    LANGUAGES C CXX
)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug)
endif()

message("*")
message("* ${PROJECT_NAME} v${PROJECT_VERSION} (${CMAKE_BUILD_TYPE})")
message("* Copyright (c) 2020 Jeroen Janssen <jeroen.janssen@gmail.com>")
message("*")

option(USE_LIBCPP "Use libc++ by adding -stdlib=libc++ flag if availbale." ON)

if(NOT WIN32 AND USE_LIBCPP)
    include(CheckCXXSourceCompiles)
    include(CMakePushCheckState)

    cmake_push_check_state()

    set(CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS} -stdlib=libc++")

    check_cxx_source_compiles("
        #include<type_traits>
        int main() { return std::is_same_v<int, char>; }
    " HAS_LIBCPP)

    if(NOT HAS_LIBCPP)
        message(WARNING "The option USE_LIBCPP is set (by default) but libc++ is not available. The flag will not be added to the target.")
    endif()

    cmake_pop_check_state()
endif()

include(GNUInstallDirs)

add_subdirectory(src)

file(GLOB HEADERS src/anrtplib/*.h src/anrtplib/*.hpp)

#
# Install targets
#

install(
    FILES ${HEADERS}
    COMPONENT ${PROJECT_NAME}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/anrtplib
    PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ
)

install(
    FILES src/anrtplib.hpp
    COMPONENT ${PROJECT_NAME}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ
)

#
# Pkg-Config
#

#if(UNIX)
#    set(includedir ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_INCLUDEDIR})
#    set(libdir ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR})
#    set(prefix ${CMAKE_INSTALL_PREFIX})
#    configure_file(libuvw-static.pc.in ${CMAKE_CURRENT_BINARY_DIR}/libuvw-static.pc @ONLY)
#    install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libuvw-static.pc
#            DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
#endif()

### Testing

option(BUILD_TESTING "Enable testing with gtest." ON)

if(BUILD_TESTING)
    option(FIND_GTEST_PACKAGE "Enable finding gtest package." OFF)

    enable_testing()
    add_subdirectory(test)
endif()

