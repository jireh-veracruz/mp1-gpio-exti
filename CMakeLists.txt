cmake_minimum_required(VERSION 3.22)

# Set the cross-compilation toolchain
set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_SOURCE_DIR}/toolchain.cmake")

# Project name and languages
project(GPIO_EXTI_CM4 C ASM)
set(CMAKE_C_STANDARD 11)

# Collect all source files
# Application sources
file(GLOB APP_SOURCES
    "Src/*.c"
)

# HAL driver sources
set(HAL_SOURCES
    "Drivers/STM32MP1xx_HAL_Driver/Src/stm32mp1xx_hal_cortex.c"
    "Drivers/STM32MP1xx_HAL_Driver/Inc/stm32mp1xx_hal_def.h"
    "Drivers/STM32MP1xx_HAL_Driver/Src/stm32mp1xx_hal_dma_ex.c"
    "Drivers/STM32MP1xx_HAL_Driver/Src/stm32mp1xx_hal_dma.c"
    "Drivers/STM32MP1xx_HAL_Driver/Src/stm32mp1xx_hal_exti.c"
    "Drivers/STM32MP1xx_HAL_Driver/Src/stm32mp1xx_hal_gpio.c"
    "Drivers/STM32MP1xx_HAL_Driver/Src/stm32mp1xx_hal_hsem.c"
    "Drivers/STM32MP1xx_HAL_Driver/Src/stm32mp1xx_hal_i2c_ex.c"
    "Drivers/STM32MP1xx_HAL_Driver/Src/stm32mp1xx_hal_i2c.c"
    "Drivers/STM32MP1xx_HAL_Driver/Src/stm32mp1xx_hal_pwr_ex.c"
    "Drivers/STM32MP1xx_HAL_Driver/Src/stm32mp1xx_hal_pwr.c"
    "Drivers/STM32MP1xx_HAL_Driver/Src/stm32mp1xx_hal_rcc_ex.c"
    "Drivers/STM32MP1xx_HAL_Driver/Src/stm32mp1xx_hal_rcc.c"
    "Drivers/STM32MP1xx_HAL_Driver/Src/stm32mp1xx_hal.c"
)

# CMSIS system source
set(CMSIS_SOURCES
    "Drivers/CMSIS/Device/ST/STM32MP1xx/Source/system_stm32mp1xx.c"
)

# BSP sources
file(GLOB BSP_SOURCES
    "Drivers/BSP/STM32MP15xx_DISCO/stm32mp15xx_disco.c"
)

# Startup assembly
set(STARTUP_SOURCE "Src/startup_stm32mp157caax.s")

# Combine all sources
set(SOURCES
    ${APP_SOURCES}
    ${HAL_SOURCES}
    ${CMSIS_SOURCES}
    ${BSP_SOURCES}
    ${STARTUP_SOURCE}
)

# Create executable
add_executable(${PROJECT_NAME}.elf ${SOURCES})

# Include directories
target_include_directories(${PROJECT_NAME}.elf PRIVATE
    Inc
    Drivers/STM32MP1xx_HAL_Driver/Inc
    Drivers/CMSIS/Device/ST/STM32MP1xx/Include
    Drivers/CMSIS/Include
    Drivers/BSP/STM32MP15xx_DISCO
)

# Compile definitions
target_compile_definitions(${PROJECT_NAME}.elf PRIVATE
    STM32MP157Cxx
    USE_HAL_DRIVER
    CORE_CM4
)

# Compiler options (for Cortex-M4)
target_compile_options(${PROJECT_NAME}.elf PRIVATE
    -mcpu=cortex-m4
    -mthumb
    -mfpu=fpv4-sp-d16
    -mfloat-abi=hard
    -Wall -Wextra -Wpedantic
    -O2  # Optimization; use -Og for debug
    -g  # Debug symbols
)

# ASM options (must match C compiler for VFP)
set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -mcpu=cortex-m4 -mthumb -mfpu=fpv4-sp-d16 -mfloat-abi=hard")

# Linker options
target_link_options(${PROJECT_NAME}.elf PRIVATE
    -T${CMAKE_CURRENT_SOURCE_DIR}/STM32MP157CAAX_RAM.ld  # Linker script
    -mcpu=cortex-m4
    -mthumb
    -mfpu=fpv4-sp-d16
    -mfloat-abi=hard
    -specs=nano.specs
    -lc -lm
    -Wl,--gc-sections
    -Wl,-Map=${PROJECT_NAME}.map,--cref
)

# Post-build commands to generate .hex and .bin
add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:${PROJECT_NAME}.elf> ${PROJECT_NAME}.hex
    COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${PROJECT_NAME}.elf> ${PROJECT_NAME}.bin
    COMMAND ${CMAKE_SIZE} $<TARGET_FILE:${PROJECT_NAME}.elf>
)