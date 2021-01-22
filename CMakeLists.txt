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

# Debug messages, should be removed
message("EVT_IS_MAIN_PROJECT: ${EVT_IS_MAIN_PROJECT}")
message("EVT_INSTALL: ${EVT_INSTALL}")
message("EVT_BUILD_SAMPLES: ${EVT_BUILD_SAMPLES}")
message("CMAKE_PROJECT_NAME: ${CMAKE_PROJECT_NAME}")
message("PROJECT_NAME: ${PROJECT_NAME}")

###############################################################################
# Install and expose library
###############################################################################
if(EVT_INSTALL)

    message("here")

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
