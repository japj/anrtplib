#
# Setup libraries
#

function(add_anrtplib_library LIB_NAME)
    target_sources(
        ${LIB_NAME}
        PRIVATE
            anrtplib/anrtplib.cpp
    )

    set_target_properties(${LIB_NAME} PROPERTIES POSITION_INDEPENDENT_CODE 1)
    target_compile_definitions(${LIB_NAME} PUBLIC ANRTPLIB_AS_LIB)
    target_compile_features(${LIB_NAME} PUBLIC cxx_std_17)

    target_include_directories(
        ${LIB_NAME}
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
    )

    if(HAS_LIBCPP)
        target_compile_options(${LIB_NAME} BEFORE PUBLIC -stdlib=libc++)
    endif()
endfunction()

#
# Static library
#

add_library(anrtplib-static STATIC)
add_library(anrtplib::anrtplib-static ALIAS anrtplib-static)
set_target_properties(anrtplib-static PROPERTIES EXCLUDE_FROM_DEFAULT_BUILD 1)
add_anrtplib_library(anrtplib-static)

#
# Install targets
#

install(EXPORT anrtplib NAMESPACE anrtplib:: DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(TARGETS anrtplib-static EXPORT anrtplib ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})