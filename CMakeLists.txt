cmake_minimum_required(VERSION 3.0)
project(dcadec)

set(DCADEC_MAJOR_VERSION 0)
set(DCADEC_MINOR_VERSION 0)
set(DCADEC_PATCH_VERSION 0)
set(DCADEC_VERSION ${DCADEC_MAJOR_VERSION}.${DCADEC_MINOR_VERSION}.${DCADEC_PATCH_VERSION} )

option(BUILD_SHARED_LIBS "Build shared library" OFF)
option(CONFIG_NDEBUG "Set -DNDEBUG linker flag" OFF)
option(CONFIG_SMALL "CONFIG_SMALL" OFF)

set(CMAKE_C_FLAGS  "-std=gnu99 -D_FILE_OFFSET_BITS=64 -Wall -Wextra -O3 -ffast-math -g -MMD ${CMAKE_C_FLAGS}")
if(CONFIG_NDEBUG)
    SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -DNDEBUG")
endif()

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

add_library(dcadec-static STATIC ${SRC_LIB})
set_target_properties(dcadec-static PROPERTIES VERSION ${DCADEC_VERSION} SOVERSION ${DCADEC_MAJOR_VERSION} OUTPUT_NAME "dcadec")

if(BUILD_SHARED_LIBS)
    add_library(dcadec-shared SHARED ${SRC_LIB})
    set_target_properties(dcadec-shared PROPERTIES VERSION ${DCADEC_VERSION} SOVERSION ${DCADEC_MAJOR_VERSION} OUTPUT_NAME "dcadec")

    INSTALL(TARGETS dcadec-shared
            LIBRARY DESTINATION "${CMAKE_INSTALL_PREFIX}/${LIB_INSTALL_DIR}")
endif()

add_executable(dcadec-cli dcadec.c)
add_executable(dcacut-cli dcacut.c)

target_link_libraries(dcadec-cli dcadec)
target_link_libraries(dcacut-cli dcadec)

set(BIN_INSTALL_DIR bin CACHE STRING "Install location of binaries")
set(LIB_INSTALL_DIR lib CACHE STRING "Install location of libraries")

INSTALL(DIRECTORY
        DESTINATION "${CMAKE_INSTALL_PREFIX}/include/libdcadec")

INSTALL(FILES ${INC_LIB}
        DESTINATION "${CMAKE_INSTALL_PREFIX}/include/libdcadec")

INSTALL(TARGETS dcadec-cli dcacut-cli dcadec-static
        RUNTIME DESTINATION "${CMAKE_INSTALL_PREFIX}/${BIN_INSTALL_DIR}"
        ARCHIVE DESTINATION "${CMAKE_INSTALL_PREFIX}/${LIB_INSTALL_DIR}"
        LIBRARY DESTINATION "${CMAKE_INSTALL_PREFIX}/${LIB_INSTALL_DIR}")

include(FindPkgConfig)
if(PKG_CONFIG_FOUND)
        configure_file("dcadec.pc.in" "dcadec.pc" @ONLY)
        install(FILES       "${CMAKE_CURRENT_BINARY_DIR}/dcadec.pc"
                DESTINATION "${CMAKE_INSTALL_PREFIX}/${LIB_INSTALL_DIR}/pkgconfig")
endif()

# uninstall target
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
