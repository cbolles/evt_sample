###############################################################################
# Top level CMakeList for building the EVT common source code
###############################################################################
cmake_minimum_required(VERSION 3.15)

include(GNUInstallDirs)
include(CMakeDependentOption)

###############################################################################
# Project Setup
###############################################################################
project(EVT
    VERSION 0.0.1
    LANGUAGES CXX
)

add_library(${PROJECT_NAME} SHARED)

# Add sources
target_sources(${PROJECT_NAME} PRIVATE
    src/io/f3xx/GPIO.cpp
    src/dev/LED.cpp
)

# Check to see if this project is being built on its own, or as a dependency
# of another project
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
    set(EVT_IS_MAIN_PROJECT TRUE)
else()
    set(EVT_IS_MAIN_PROJECT FALSE)
endif()

###############################################################################
# Compiler settings
###############################################################################
# Verify that the ARM tools path is set
if(NOT DEFINED ENV{GCC_ARM_TOOLS_PATH})
    message(FATAL_ERROR 
        "Set your environment variables you frickin' hecker."
        "   --John Jenco"
    )
endif()

# Set C and C++ compiler
set(CMAKE_CXX_COMPILER "$ENV{GCC_ARM_TOOLS_PATH}/arm-none-eabi-g++")
set(CMAKE_C_COMPILER "$ENV{GCC_ARM_TOOLS_PATH}/arm-none-eabi-gcc")

###############################################################################
# Handle dependencies
###############################################################################
add_subdirectory(libs/HALf3/)

# Later this will be handled dynamically, either the HALf3 or the HALf4 will
# be linked to
target_link_libraries(${PROJECT_NAME}
    HALf3
)

###############################################################################
# Provide build options
###############################################################################
# Determine if the sample code should be built. Defaults to false if this
# project is not the main project.
cmake_dependent_option(EVT_BUILD_SAMPLES
    "Build the EVT core samples" ON
    "EVT_IS_MAIN_PROJECT" OFF
)

# Determine if the project should be installed and made accessible as a
# library. Defaults to true if this project is not the main project
cmake_dependent_option(EVT_INSTALL
    "Install the EVT library" OFF
    "NOT EVT_IS_MAIN_PROJECT" ON
)

# Determine if the sample code should be added. By default, if this is the
# main project then samples will be compiled, otherwise they will not.
cmake_dependent_option(EVT_BUILD_SAMPLES
    "Build samples" OFF
    "EVT_IS_MAIN_PROJECT" ON
)

###############################################################################
# Install and expose library
###############################################################################
if(EVT_INSTALL)

    # Expose headers
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
endif()

###############################################################################
# Handle adding samples
###############################################################################
if(EVT_BUILD_SAMPLES)
    # Add the sample code sub directory
    add_subdirectory(samples)
endif()
