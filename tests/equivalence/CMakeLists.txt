cmake_minimum_required(VERSION 3.14)
set(AIOT_PATH $ENV{XMOS_AIOT_SDK_PATH})

include("${AIOT_PATH}/tools/cmake_utils/xmos_toolchain.cmake") 
enable_language(CXX XC C ASM)

set(APP_NAME app_equivalence)
project(${APP_NAME} VERSION 0.1.0)

set(INSTALL_DIR "${PROJECT_SOURCE_DIR}/bin")

add_executable(${APP_NAME})

set(XS3_MATH_PATH ${AIOT_PATH}/modules/lib_xs3_math/lib_xs3_math)
set(VTB_PATH ${AIOT_PATH}/modules/lib_voice_toolbox/lib_voice_toolbox)
set(DSP_PATH ${AIOT_PATH}/modules/lib_dsp/lib_dsp/lib_dsp)

if(NOT DEFINED ENV{MEL_FILTER_H_FILE})
    message(FATAL_ERROR "MEL_FILTER_H_FILE env var not set")
else()
    message(STATUS "using MEL_FILTER_H_FILE: $ENV{MEL_FILTER_H_FILE}")
endif()
 
set(BUILD_FLAGS
        "-Os"
        "-g"
        "-target=XCORE-AI-EXPLORER"
        "-D__XS2A__"
        "-report"
        "-fcmdline-buffer-bytes=256"
        "-DMEL_FILTER_H_FILE=\"$ENV{MEL_FILTER_H_FILE}\""
        )

set(APP_SRCS
        src/main.c
        ../../lib_audio_features/src/mel_filter.c
    )

set(APP_INCLUDES
        src
        ../../lib_audio_features/src
    )

file(GLOB_RECURSE XS3_MATH_VECT_C_SRCS ${XS3_MATH_PATH}/src/vect/*.c)
file(GLOB_RECURSE XS3_MATH_VECT_S_SRCS ${XS3_MATH_PATH}/src/vect/*.S)
file(GLOB_RECURSE XS3_MATH_BFP_C_SRCS ${XS3_MATH_PATH}/src/bfp/*.c)
file(GLOB_RECURSE XS3_MATH_BFP_S_SRCS ${XS3_MATH_PATH}/src/bfp/*.S)
file(GLOB_RECURSE XS3_MATH_ARCH_C_SRCS ${XS3_MATH_PATH}/src/arch/xcore/*.c)
file(GLOB_RECURSE XS3_MATH_ARCH_S_SRCS ${XS3_MATH_PATH}/src/arch/xcore/*.S)
set(XS3_MATH_SRCS
        ${XS3_MATH_VECT_C_SRCS}
        ${XS3_MATH_VECT_S_SRCS}
        ${XS3_MATH_BFP_C_SRCS}
        ${XS3_MATH_BFP_S_SRCS}
        ${XS3_MATH_ARCH_C_SRCS}
        ${XS3_MATH_ARCH_S_SRCS}
    )

set(VTB_SRCS
    )

set(DSP_SRCS
    )


target_include_directories(${APP_NAME}
  PRIVATE ${APP_INCLUDES}
  PRIVATE ${XS3_MATH_PATH}/api
  PRIVATE ${XS3_MATH_PATH}/src/arch/xcore
  PRIVATE ${VTB_PATH}/api
  PRIVATE ${DSP_PATH}/api
)

target_sources(${APP_NAME}
    PRIVATE ${APP_SRCS}
    PRIVATE ${XS3_MATH_SRCS}
    PRIVATE ${VTB_SRCS}
    PRIVATE ${DSP_SRCS}
)

target_compile_options(${APP_NAME}  PRIVATE ${BUILD_FLAGS})
target_link_options(${APP_NAME} PRIVATE ${BUILD_FLAGS})

install(TARGETS ${APP_NAME} DESTINATION ${INSTALL_DIR})

