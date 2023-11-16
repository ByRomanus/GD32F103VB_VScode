# CMake file for FRDM-K22F_Simple
cmake_minimum_required(VERSION 3.15.3)

# Optional: print out extra messages to see what is going on. Comment it to have less verbose messages
set(CMAKE_VERBOSE_MAKEFILE ON)

# Path to toolchain file. This one has to be before 'project()' below
set(CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/arm-none-eabi-gcc.cmake)

# Setup project, output and linker file
project(GD32F103)
set(EXECUTABLE ${PROJECT_NAME}.elf)
set(LINKER_FILE ${CMAKE_SOURCE_DIR}/device/linker.ld)

enable_language(ASM C CXX)
set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

# Optional: issue a message to be sure it uses the correct toolchain file.
message(STATUS "CMAKE_TOOLCHAIN_FILE is: ${CMAKE_TOOLCHAIN_FILE}")

# List of source files
set(SRC_FILES
        src/main.cpp

        device/startup_gd32f10x_md.s
        device/system_gd32f10x.c
        device/src/gd32f10x_adc.c 
        device/src/gd32f10x_bkp.c 
        device/src/gd32f10x_can.c 
        device/src/gd32f10x_crc.c
        device/src/gd32f10x_dac.c
        device/src/gd32f10x_dbg.c
        device/src/gd32f10x_dma.c
        device/src/gd32f10x_enet.c
        device/src/gd32f10x_exmc.c 
        device/src/gd32f10x_exti.c 
        device/src/gd32f10x_fmc.c 
        device/src/gd32f10x_fwdgt.c 
        device/src/gd32f10x_gpio.c 
        device/src/gd32f10x_i2c.c 
        device/src/gd32f10x_misc.c 
        device/src/gd32f10x_pmu.c 
        device/src/gd32f10x_rcu.c 
        device/src/gd32f10x_rtc.c 
        device/src/gd32f10x_sdio.c 
        device/src/gd32f10x_spi.c 
        device/src/gd32f10x_timer.c 
        device/src/gd32f10x_usart.c 
        device/src/gd32f10x_wwdgt.c
        )

# Build the executable based on the source files
add_executable(${EXECUTABLE} ${SRC_FILES})

# List of compiler defines, prefix with -D compiler option
target_compile_definitions(${EXECUTABLE} PRIVATE
        -DGD32F103VBT6
        -DGD32F10X_MD
        )

# List of include directories
target_include_directories(${EXECUTABLE} PRIVATE
        src
        inc 
        device
        device/inc
        device/src
        CMSIS
        )

# Compiler options
target_compile_options(${EXECUTABLE} PRIVATE
        -mcpu=cortex-m3
        -mthumb
        -mfpu=auto
        -mfloat-abi=soft

        -fdata-sections
        -ffunction-sections

        -Wall
        -O0
        -g3
        )

# Linker options
target_link_options(${EXECUTABLE} PRIVATE
        -T${LINKER_FILE}
        -mcpu=cortex-m3
        -mthumb
        -mfpu=auto
        -mfloat-abi=soft
        -specs=nano.specs
        -specs=rdimon.specs
        -specs=nosys.specs
        -lc
        -lm
        -Wl,-Map=${PROJECT_NAME}.map,--cref
        -Wl,--gc-sections
        -Xlinker -print-memory-usage -Xlinker
        )

# Optional: Print executable size as part of the post build process
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND ${CMAKE_SIZE_UTIL} ${EXECUTABLE})

# Optional: Create hex, bin and S-Record files after the build
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -O srec --srec-len=64 ${EXECUTABLE} ${PROJECT_NAME}.s19
        COMMAND ${CMAKE_OBJCOPY} -O ihex ${EXECUTABLE} ${PROJECT_NAME}.hex
        COMMAND ${CMAKE_OBJCOPY} -O binary ${EXECUTABLE} ${PROJECT_NAME}.bin)
