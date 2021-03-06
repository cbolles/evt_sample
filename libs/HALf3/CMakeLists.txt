###############################################################################
# CMake file for exposing HAL as library.
###############################################################################
cmake_minimum_required(VERSION 3.15)

include(GNUInstallDirs)

message("${CMAKE_C_COMPILER}")

###############################################################################
# Project Setup
###############################################################################
project(HALf3
    VERSION 0.0.1
    LANGUAGES C ASM
)

add_library(${PROJECT_NAME} STATIC)

# Glob together all C source files
file(GLOB HALf3_C_SRC_FILES src/*.c)

# Handle assembly file
set_source_files_properties(
    src/startup_stm32f302x8.s 
    PROPERTIES COMPILE_FLAGS "-x assembler-with-cpp"
)

# Add linker script
set_target_properties(${TARGET_NAME} PROPERTIES 
    LINK_DEPENDS STM32F302C8Tx_FLASH.ld
)

# Add C and Assembly source files
target_sources(${PROJECT_NAME} PRIVATE 
    ${HALf3_C_SRC_FILES}
    src/startup_stm32f302x8.s
)

# Add headers privately.
# NOTE: This is a flat inclusion of the headers as that is how the STM HAL
# code expects the headers to be
target_include_directories(${PROJECT_NAME} PRIVATE include/HALf3/)

# Define more specifics about the F3 version
add_compile_definitions(STM32F302x8)

###############################################################################
# Compiler settings
###############################################################################
# Add flags for specific version of hardware
set(HALf3_HARDWARE_FLAGS "-mthumb -mcpu=cortex-m4")

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${HALf3_HARDWARE_FLAGS}")

###############################################################################
# Expose library
###############################################################################
target_include_directories(${PROJECT_NAME}
    PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/>
           $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

target_compile_definitions(${PROJECT_NAME} PRIVATE -D_EXPORT)

set_target_properties(${PROJECT_NAME} PROPERTIES DEBUG_POSTFIX "d")

install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}-config
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(
    EXPORT ${PROJECT_NAME}-config
    NAMESPACE ${PROJECT_NAME}::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
)

