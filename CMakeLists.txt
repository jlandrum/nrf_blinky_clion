cmake_minimum_required(VERSION 3.6)
project(main C ASM)

include(CMakeForceCompiler)

set (CC /usr/local/bin/arm-none-eabi-gcc)
set (CXX /usr/local/bin/arm-none-eabi-g++)
set (OBJCOPY /usr/local/bin/arm-none-eabi-objcopy)
set (SIZE /usr/local/bin/arm-none-eabi-size)

set(CMAKE_C_COMPILER ${CC})
set(CMAKE_CXX_COMPILER ${CXX})
set(CMAKE_ASM_COMPILER ${CC})

set(CMAKE_SYSTEM_NAME Generic)

# NRF5x Configuration
set(NRF_SDK_ROOT /usr/local/lib/nrfsdk)
set(NRF_FAMILY NRF52)
include(nrfcmake.txt)

set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_C_STANDARD 99)

set(SOURCE_FILES main.c
        ${__NRF_SOURCE_FILES}
        ${G_NRF_SOURCE_CORE}
)

include_directories(
        ./
        ./bsp/
        ${G_NRF_INCLUDE_CORE})

set(CMAKE_C_FLAGS ${__NRF_C_FLAGS})
set(CMAKE_ASM_FLAGS ${__NRF_ASM_FLAGS})
set(CMAKE_EXE_LINKER_FLAGS  ${__NRF_LINKER_FLAGS})

add_executable(main ${SOURCE_FILES})

set_target_properties(
        main
        PROPERTIES
            OUTPUT_NAME "build"
            SUFFIX ".out"
)

add_custom_target(size
        DEPENDS main
        COMMAND ${SIZE} build.out
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR})

add_custom_target(createhex
        DEPENDS size main
        COMMAND ${OBJCOPY} -O ihex build.out build.hex
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR})

add_custom_target(flash
        DEPENDS createhex size main
        COMMAND nrfjprog --program build.hex -f ${__NRF_FAMILY} --sectorerase
        COMMAND nrfjprog --reset -f ${__NRF_FAMILY}
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR})

add_custom_target(flash_softdevice_nrf52_132_2
        COMMAND nrfjprog --program support/s132_nrf52_2.0.1_softdevice.hex -f ${__NRF_FAMILY} --sectorerase
        COMMAND nrfjprog --reset -f ${__NRF_FAMILY}
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})

add_custom_target(flash_softdevice_nrf52_132_3
        COMMAND nrfjprog --program support/s132_nrf52_3.0.0_softdevice.hex -f ${__NRF_FAMILY} --sectorerase
        COMMAND nrfjprog --reset -f ${__NRF_FAMILY}
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})