cmake_minimum_required(VERSION 3.2)
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

if(NOT CONFIG_SMALL)
    set(SRC_LIB ${SRC_LIB}
        "libdcadec/dca_frame.c"
        "libdcadec/dca_stream.c"
        "libdcadec/dca_waveout.c"
        )
endif()

add_library(libdcadec ${SRC_LIB})
set_target_properties(libdcadec PROPERTIES VERSION ${DCADEC_VERSION} SOVERSION ${DCADEC_MAJOR_VERSION})

add_executable(dcadec dcadec.c)
add_executable(dcacut dcacut.c)

target_link_libraries(dcadec libdcadec)
target_link_libraries(dcacut libdcadec)

include(FindPkgConfig)
if(PKG_CONFIG_FOUND)
        configure_file("dcadec.pc.in" "dcadec.pc" @ONLY)
        install(FILES       "${CMAKE_CURRENT_BINARY_DIR}/dcadec.pc"
                DESTINATION "lib/pkgconfig")
endif()
