cmake_minimum_required(VERSION 3.14)
project(xovi VERSION 0.1.1 LANGUAGES C ASM)

option(XOVI_SANDBOX "Build a sandbox executable instead of an LD_PRELOAD library." OFF)

set(XOVI_BASE_SOURCES
   src/main.c
   src/dynamiclinker.c
   src/hash.c
   )

if(NOT XOVI_SANDBOX)
    add_library(xovi SHARED ${XOVI_BASE_SOURCES})
    # remove the 'lib' prefix from shared libraries to match existing behavior
    set_target_properties(xovi PROPERTIES PREFIX "")
else()
    add_executable(xovi ${XOVI_BASE_SOURCES})
    target_compile_definitions(xovi PRIVATE DEBUG DEBUGFUNC)
endif(NOT XOVI_SANDBOX)

target_compile_definitions(xovi PRIVATE _GNU_SOURCE)

if(${CMAKE_SYSTEM_PROCESSOR} STREQUAL aarch64)
    target_sources(xovi PRIVATE
                  src/trampolines/aarch64/aarch64.c
                  src/trampolines/aarch64/untrampoline.S
                  )
    target_compile_definitions(xovi PRIVATE AARCH64)
elseif(${CMAKE_SYSTEM_PROCESSOR} STREQUAL armv6l)
    target_sources(xovi PRIVATE
                  src/trampolines/arm32/arm32.c
                  src/trampolines/arm32/untrampoline.S
                  )
    target_compile_definitions(xovi PRIVATE ARM)
else()
    message(FATAL_ERROR "xovi doesn't support '${CMAKE_SYSTEM_PROCESSOR}' architecture")
endif(${CMAKE_SYSTEM_PROCESSOR} STREQUAL aarch64)
