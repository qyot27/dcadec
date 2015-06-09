cmake_minimum_required(VERSION 3.0)
project(dcadec)

# Project versioning
set(DCADEC_MAJOR_VERSION 0)
set(DCADEC_MINOR_VERSION 0)
set(DCADEC_PATCH_VERSION 0)
set(DCADEC_VERSION ${DCADEC_MAJOR_VERSION}.${DCADEC_MINOR_VERSION}.${DCADEC_PATCH_VERSION} )

# Define project options
option(BUILD_PROGRAMS "Build dcadec and dcacut cli applications" ON)
option(BUILD_DCADEC "Build dcadec cli application" ON)
option(BUILD_DCACUT "Build dcacut cli application" ON)
option(BUILD_SHARED_LIB "Build shared library" OFF)
option(BUILD_STATIC_LIB "Build static library" ON)
option(CONFIG_NDEBUG "Set -DNDEBUG linker flag" OFF)
option(CONFIG_SMALL "CONFIG_SMALL" OFF)

# Basic GCC compiler flags
set(CMAKE_C_FLAGS  "-std=gnu99 -D_FILE_OFFSET_BITS=64 -Wall -Wextra -O3 -ffast-math -g -MMD ${CMAKE_C_FLAGS}")
if(CONFIG_NDEBUG)
    SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -DNDEBUG")
endif()


# Define source lists
set(SRC_LIB
    "libdcadec/bitstream.c"
    "libdcadec/core_decoder.c"
    "libdcadec/dca_context.c"
    "libdcadec/dmix_tables.c"
    "libdcadec/exss_parser.c"
    "libdcadec/idct_fixed.c"
    "libdcadec/idct_float.c"
    "libdcadec/interpolator.c"
    "libdcadec/interpolator_fixed.c"
    "libdcadec/interpolator_float.c"
    "libdcadec/ta.c"
    "libdcadec/xll_decoder.c"
    )

set(INC_LIB
    "libdcadec/dca_context.h"
    )

if(NOT CONFIG_SMALL)
    set(SRC_LIB ${SRC_LIB}
        "libdcadec/dca_frame.c"
        "libdcadec/dca_stream.c"
        "libdcadec/dca_waveout.c"
        )

    set(INC_LIB ${INC_LIB}
        "libdcadec/dca_frame.h"
        "libdcadec/dca_stream.h"
        "libdcadec/dca_waveout.h"
    )
endif()


# Assembling static and shared libraries
add_library(dcadec-core OBJECT ${SRC_LIB})

if(BUILD_STATIC_LIB)
    add_library(dcadec-library-static STATIC $<TARGET_OBJECTS:dcadec-core>)
    set_target_properties(dcadec-library-static PROPERTIES VERSION ${DCADEC_VERSION} SOVERSION ${DCADEC_MAJOR_VERSION} OUTPUT_NAME "dcadec")
endif()

if(BUILD_SHARED_LIB)
    add_library(dcadec-library-shared SHARED $<TARGET_OBJECTS:dcadec-core>)
    set_target_properties(dcadec-library-shared PROPERTIES VERSION ${DCADEC_VERSION} SOVERSION ${DCADEC_MAJOR_VERSION} OUTPUT_NAME "dcadec")
endif()


# Assembling command-line applications
if(NOT BUILD_PROGRAMS)
    set(BUILD_DCADEC OFF)
    set(BUILD_DCACUT OFF)
endif()

if(BUILD_DCADEC)
    add_executable(dcadec-program-cli dcadec.c)
    set_target_properties(dcadec-program-cli PROPERTIES OUTPUT_NAME "dcadec")

    target_link_libraries(dcadec-program-cli dcadec)
endif()

if(BUILD_DCACUT)
    add_executable(dcacut-program-cli dcacut.c)
    set_target_properties(dcacut-program-cli PROPERTIES OUTPUT_NAME "dcacut")

    target_link_libraries(dcacut-program-cli dcadec)
endif()


# Set installation rules
set(BIN_INSTALL_DIR bin CACHE STRING "Install location of binaries")
set(LIB_INSTALL_DIR lib CACHE STRING "Install location of libraries")

INSTALL(DIRECTORY
        DESTINATION "${CMAKE_INSTALL_PREFIX}/include/libdcadec")

INSTALL(FILES ${INC_LIB}
        DESTINATION "${CMAKE_INSTALL_PREFIX}/include/libdcadec")

if(BUILD_DCADEC)
INSTALL(TARGETS dcadec-program-cli
        RUNTIME DESTINATION "${CMAKE_INSTALL_PREFIX}/${BIN_INSTALL_DIR}")
endif()

if(BUILD_DCACUT)
INSTALL(TARGETS dcacut-program-cli
        RUNTIME DESTINATION "${CMAKE_INSTALL_PREFIX}/${BIN_INSTALL_DIR}")
endif()

if(BUILD_STATIC_LIB)
    INSTALL(TARGETS  dcadec-library-static
            ARCHIVE DESTINATION "${CMAKE_INSTALL_PREFIX}/${LIB_INSTALL_DIR}")
endif()

if(BUILD_SHARED_LIB)
    if(WIN32)
        INSTALL(TARGETS dcadec-library-shared
                RUNTIME DESTINATION "${CMAKE_INSTALL_PREFIX}/${BIN_INSTALL_DIR}"
                LIBRARY DESTINATION "${CMAKE_INSTALL_PREFIX}/${LIB_INSTALL_DIR}"
                ARCHIVE DESTINATION "${CMAKE_INSTALL_PREFIX}/${LIB_INSTALL_DIR}")
    else()
        INSTALL(TARGETS dcadec-library-shared
                LIBRARY DESTINATION "${CMAKE_INSTALL_PREFIX}/${LIB_INSTALL_DIR}")
    endif()
endif()

include(FindPkgConfig)
if(PKG_CONFIG_FOUND)
        configure_file("dcadec.pc.in" "dcadec.pc" @ONLY)
        install(FILES       "${CMAKE_CURRENT_BINARY_DIR}/dcadec.pc"
                DESTINATION "${CMAKE_INSTALL_PREFIX}/${LIB_INSTALL_DIR}/pkgconfig")
endif()


# Set uninstallation rules
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
