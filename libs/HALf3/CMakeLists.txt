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

add_library(${PROJECT_NAME} SHARED)

# Glob together all C source files
file(GLOB SRC_FILES src/*.c)

# Add C source
target_sources(${PROJECT_NAME} PRIVATE ${SRC_FILES})

# Add headers privately.
# NOTE: This is a flat inclusion of the headers as that is how the STM HAL
# code expects the headers to be
target_include_directories(${PROJECT_NAME} PRIVATE include/HALf3/)

# Define more specifics about the F3 version
add_compile_definitions(STM32F302x8)

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

