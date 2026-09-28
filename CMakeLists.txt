cmake_minimum_required(VERSION 3.21)

project(ez3i VERSION 1.0.2) # extracted_z3_integer

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

option(BUILD_SHARED_LIBS
    "Build shared library (ON) or static library (OFF)"
    OFF
)

option(EZ3I_BUILD_TESTS
    "Build unit tests"
    ${PROJECT_IS_TOP_LEVEL}
)

option(EZ3I_INSTALL
    "Generate install targets and package configuration"
    ${PROJECT_IS_TOP_LEVEL}
)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

if(PROJECT_IS_TOP_LEVEL)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
endif()

# ---------------------------------------------------------------------------
# Library
# ---------------------------------------------------------------------------

add_library(${PROJECT_NAME}
    src/mpz.cpp
)

add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

# Public headers are located in:
#
#     include/ez3i/mpz.h
#
# Therefore consumers use:
#
#     #include <ez3i/mpz.h>
#
# BUILD_INTERFACE:
#     Used when ez3i is consumed directly from the source/build tree.
#
# INSTALL_INTERFACE:
#     Used after ez3i has been installed.
#
target_include_directories(${PROJECT_NAME}
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

target_compile_features(${PROJECT_NAME}
    PUBLIC
        cxx_std_20
)

# ---------------------------------------------------------------------------
# Compile definitions
# ---------------------------------------------------------------------------

if(BUILD_SHARED_LIBS)
    target_compile_definitions(${PROJECT_NAME}
        PRIVATE
            EZ3I_EXPORTS
    )
else()
    target_compile_definitions(${PROJECT_NAME}
        PUBLIC
            EZ3I_STATIC
    )
endif()

# ---------------------------------------------------------------------------
# Tests
# ---------------------------------------------------------------------------

if(EZ3I_BUILD_TESTS AND PROJECT_IS_TOP_LEVEL)
    enable_testing()
    add_subdirectory(tests)
endif()

# ---------------------------------------------------------------------------
# Installation / package configuration
# ---------------------------------------------------------------------------

if(EZ3I_INSTALL)

    # Install library and public headers.
    install(TARGETS ${PROJECT_NAME}
        EXPORT ez3iTargets
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )

    install(DIRECTORY include/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )

    # -----------------------------------------------------------------------
    # CMake package config
    # -----------------------------------------------------------------------

    set(ez3i_INSTALL_CMAKEDIR
        "${CMAKE_INSTALL_LIBDIR}/cmake/ez3i"
    )

    configure_package_config_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake/ez3iConfig.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/ez3iConfig.cmake"
        INSTALL_DESTINATION
            "${ez3i_INSTALL_CMAKEDIR}"
    )

    write_basic_package_version_file(
        "${CMAKE_CURRENT_BINARY_DIR}/ez3iConfigVersion.cmake"
        VERSION
            ${PROJECT_VERSION}
        COMPATIBILITY
            SameMajorVersion
    )

    # Export targets:
    #
    #     ez3i::ez3i
    #
    install(
        EXPORT ez3iTargets
        FILE
            ez3iTargets.cmake
        NAMESPACE
            ez3i::
        DESTINATION
            "${ez3i_INSTALL_CMAKEDIR}"
    )

    # Install:
    #
    #     ez3iConfig.cmake
    #     ez3iConfigVersion.cmake
    #
    install(
        FILES
            "${CMAKE_CURRENT_BINARY_DIR}/ez3iConfig.cmake"
            "${CMAKE_CURRENT_BINARY_DIR}/ez3iConfigVersion.cmake"
        DESTINATION
            "${ez3i_INSTALL_CMAKEDIR}"
    )

endif()
