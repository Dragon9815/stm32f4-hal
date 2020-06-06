cmake_minimum_required(VERSION 3.12)

set(HAL_COMPONENTS adc can cec cortex crc cryp dac dcmi dma dma2d eth flash
    flash_ramfunc fmpi2c gpio hash hcd i2c i2s irda iwdg ltdc
    nand nor pccard pcd pwr qspi rcc rng rtc sai sd sdram
    smartcard spdifrx spi sram tim uart usart wwdg)

set(LL_COMPONENTS adc crc dac dma dma2d exti fmc fsmc gpio i2c lptim pwr
    rcc rng rtc sdmmc spi tim usart usb utils)

set(HAL_REQUIRED_COMPONENTS cortex pwr rcc)

# Components that have _ex sources
set(HAL_EX_COMPONENTS adc cryp dac dcmi dma flash fmpi2c hash i2c i2s pcd
    pwr rcc rtc sai tim)

set(HAL_PREFIX stm32f4xx_)

set(HAL_HEADERS
    ${HAL_PREFIX}hal.h
    ${HAL_PREFIX}hal_def.h
)

set(HAL_SRCS
    ${HAL_PREFIX}hal.c
)

if(NOT STM32HAL_NEEDED_COMPONENTS)
    set(STM32HAL_NEEDED_COMPONENTS ${HAL_COMPONENTS})
    message(STATUS "No STM32HAL components selected, using all: ${STM32HAL_NEEDED_COMPONENTS}")
endif()

if(NOT STM32HAL_NEEDED_LL_COMPONENTS)
    set(STM32HAL_NEEDED_LL_COMPONENTS ${LL_COMPONENTS})
    message(STATUS "No STM32LL components selected, using all: ${STM32HAL_NEEDED_LL_COMPONENTS}")
endif()

# Append required components, if they are not in the list
foreach(cmp ${HAL_REQUIRED_COMPONENTS})
    list(FIND STM32HAL_NEEDED_COMPONENTS ${cmp} STM32HAL_FOUND_INDEX)
    if(${STM32HAL_FOUND_INDEX} LESS 0)
        list(APPEND STM32HAL_NEEDED_COMPONENTS ${cmp})
    endif()
endforeach()

# Parse all components and add to headers an sources
foreach(cmp ${STM32HAL_NEEDED_COMPONENTS})
    list(FIND HAL_COMPONENTS ${cmp} STM32HAL_FOUND_INDEX)
    if(${STM32HAL_FOUND_INDEX} LESS 0)
        message(FATAL_ERROR "Unknown STM32HAL component: ${cmp}. Available components: ${HAL_COMPONENTS}")
	else()
        list(APPEND HAL_HEADERS ${HAL_PREFIX}hal_${cmp}.h)
        list(APPEND HAL_SRCS ${HAL_PREFIX}hal_${cmp}.c)
        STRING(TOUPPER "${cmp}" cmp_upper)
        set("HAL_${cmp_upper}_MODULE_ENABLED" true)
    endif()
    list(FIND HAL_EX_COMPONENTS ${cmp} STM32HAL_FOUND_INDEX)
    if(NOT (${STM32HAL_FOUND_INDEX} LESS 0))
        list(APPEND HAL_HEADERS ${HAL_PREFIX}hal_${cmp}_ex.h)
        list(APPEND HAL_SRCS ${HAL_PREFIX}hal_${cmp}_ex.c)
    endif()
endforeach()

foreach(cmp ${STM32HAL_NEEDED_LL_COMPONENTS})
    list(FIND LL_COMPONENTS ${cmp} STM32HAL_FOUND_INDEX)
    if(${STM32HAL_FOUND_INDEX} LESS 0)
        message(FATAL_ERROR "Unknown STM32LL component: ${cmp}. Available components: ${LL_COMPONENTS}")
	else()
        list(APPEND LL_HEADERS ${HAL_PREFIX}ll_${cmp}.h)
        list(APPEND LL_SRCS ${HAL_PREFIX}ll_${cmp}.c)
    endif()
endforeach()

if(STM32HAL_USE_LL)
    list(APPEND HAL_DEFINES "USE_FULL_LL_DRIVER")
endif()

list(REMOVE_DUPLICATES HAL_HEADERS)
list(REMOVE_DUPLICATES HAL_SRCS)
list(REMOVE_DUPLICATES HAL_DEFINES)

list(TRANSFORM HAL_SRCS PREPEND ${CMAKE_CURRENT_SOURCE_DIR}/src/)
list(TRANSFORM LL_SRCS PREPEND ${CMAKE_CURRENT_SOURCE_DIR}/src/)

set(project_name stm32f4-hal)

project(${project_name})

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/stm32f4xx_hal_conf.h.in ${CMAKE_CURRENT_SOURCE_DIR}/include/stm32f4xx_hal_conf.h)

add_library(${project_name} STATIC ${HAL_SRCS} ${LL_SRCS})
target_compile_definitions(${project_name} PUBLIC ${HAL_DEFINES})
target_include_directories(${project_name} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_link_libraries(${project_name} stm32f4-cmsis)