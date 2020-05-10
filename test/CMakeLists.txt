#
# Tests configuration
#

include(FetchContent)

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

if(FIND_GTEST_PACKAGE)
    find_package(GTest REQUIRED)
else()
    FetchContent_Declare(
        googletest
        GIT_REPOSITORY https://github.com/google/googletest.git
        GIT_TAG master
        GIT_SHALLOW 1
    )

    FetchContent_GetProperties(googletest)

    if(NOT googletest_POPULATED)
        FetchContent_Populate(googletest)
        set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
        add_subdirectory(${googletest_SOURCE_DIR} ${googletest_BINARY_DIR} EXCLUDE_FROM_ALL)
    endif()

    add_library(GTest::Main ALIAS gtest_main)

    target_compile_features(gtest PUBLIC cxx_std_17)
    target_compile_features(gtest_main PUBLIC cxx_std_17)
    target_compile_features(gmock PUBLIC cxx_std_17)
    target_compile_features(gmock_main PUBLIC cxx_std_17)
endif()

if(WIN32)
    set(WINSOCK2 ws2_32)
elseif(NOT APPLE)
    find_library(LIBRT rt)
endif()

function(ADD_ANRTPLIB_TEST TEST_NAME TEST_SOURCE)
    add_executable(${TEST_NAME} ${TEST_SOURCE})

    target_link_libraries(
        ${TEST_NAME}
        PRIVATE
            anrtplib::anrtplib-static
            GTest::Main
            Threads::Threads
            ${LIBRT}
            ${WINSOCK2}
        )

    target_compile_definitions(${TEST_NAME} PRIVATE $<$<NOT:$<TARGET_EXISTS:anrtplib::anrtplib>>:ANRTPLIB_AS_LIB>)
    add_test(NAME uvw_${TEST_NAME} COMMAND $<TARGET_FILE:${TEST_NAME}>)
endfunction()

function(ADD_ANRTPLIB_DIR_TEST TEST_NAME TEST_SOURCE)
    ADD_ANRTPLIB_TEST(${TEST_NAME} ${TEST_SOURCE})
    string(TOUPPER "${TEST_NAME}" TEST_NAME_UPPER)
    set(TEST_DIR_VAR "TARGET_${TEST_NAME_UPPER}_DIR")
    file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/${TEST_NAME}_data)
    target_compile_definitions(${TEST_NAME} PRIVATE ${TEST_DIR_VAR}="${TEST_NAME}_data")
endfunction()

function(ADD_ANRTPLIB_LIB_TEST TEST_NAME TEST_SOURCE)
    ADD_ANRTPLIB_TEST(${TEST_NAME} ${TEST_SOURCE})
    target_link_libraries(${TEST_NAME} PRIVATE ${CMAKE_DL_LIBS})
endfunction()

# List of available targets

ADD_ANRTPLIB_TEST(main test.cpp)
