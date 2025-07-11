cmake_minimum_required(VERSION 3.10 FATAL_ERROR)
project(lmplugin VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Check for required system dependencies
include(CheckIncludeFileCXX)
include(CheckCXXCompilerFlag)

# Check for C++17 support
CHECK_CXX_COMPILER_FLAG("-std=c++17" COMPILER_SUPPORTS_CXX17)
if(NOT COMPILER_SUPPORTS_CXX17)
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++17 support. Please use a different C++ compiler.")
endif()

# Find HDF5 with fallback options
find_package(HDF5 COMPONENTS CXX HL)
if(NOT HDF5_FOUND)
    find_package(PkgConfig QUIET)
    if(PKG_CONFIG_FOUND)
        pkg_check_modules(HDF5 QUIET hdf5)
    endif()
    if(NOT HDF5_FOUND)
        message(FATAL_ERROR "HDF5 library not found. Please install HDF5 development package.")
    endif()
endif()

# Allow user to directly set VMD_ROOT_DIR
set(VMD_ROOT_DIR "" CACHE PATH "Path to VMD runtime directory")

# Only try to auto-detect if VMD_ROOT_DIR is not set
if(NOT VMD_ROOT_DIR)
    find_program(VMD_EXECUTABLE NAMES vmd PATH_SUFFIXES bin DOC "Path to VMD start script")
    if(VMD_EXECUTABLE)
        file(STRINGS "${VMD_EXECUTABLE}" _VMD_ROOT_DIR REGEX "^[^#]*defaultvmddir[ \t]*=")
        string(REGEX REPLACE "(^.*=[^\"]*\"?|\"$)" "" _VMD_ROOT_DIR "${_VMD_ROOT_DIR}")
        set(VMD_ROOT_DIR "${_VMD_ROOT_DIR}" CACHE PATH "Path to VMD runtime directory" FORCE)
    else()
        set(VMD_ROOT_DIR "/usr/local/lib/vmd" CACHE PATH "Path to VMD runtime directory" FORCE)
    endif()
endif()

message(STATUS "VMD_ROOT_DIR: ${VMD_ROOT_DIR}")

find_path(VMD_PLUGIN_DIR "pdbplugin.so"
            PATHS 
                "${VMD_ROOT_DIR}/plugins/LINUXAMD64/molfile" 
                "${VMD_ROOT_DIR}/plugins/LINUX/molfile" 
                "${VMD_ROOT_DIR}/plugins/MACOSX86/molfile" 
                "${VMD_ROOT_DIR}/plugins/MACOSX86_64/molfile" 
                "${VMD_ROOT_DIR}/plugins/WIN32/molfile"
                "${VMD_ROOT_DIR}/plugins/WIN64/molfile"
            DOC
                "Path to VMD molfile plugin directory"
            NO_DEFAULT_PATH)

find_path(VMD_PLUGIN_INCLUDE_DIR "vmdplugin.h"
            PATHS 
                "${VMD_ROOT_DIR}/plugins/include"
            DOC
                "Path to VMD plugin header directory"
            NO_DEFAULT_PATH)

mark_as_advanced(VMD_ROOT_DIR VMD_EXECUTABLE)

if(VMD_PLUGIN_INCLUDE_DIR)
    message(STATUS "Found VMD molfile directory: `${VMD_PLUGIN_DIR}'")
    add_library(lmplugin SHARED "${CMAKE_CURRENT_SOURCE_DIR}/lmplugin.cpp")
    target_include_directories(lmplugin PUBLIC "${HDF5_INCLUDE_DIRS}" "${VMD_PLUGIN_INCLUDE_DIR}")
    target_link_libraries(lmplugin PUBLIC "${HDF5_HL_LIBRARIES}" "${HDF5_LIBRARIES}")
    target_compile_features(lmplugin PRIVATE cxx_std_17)
    set_target_properties(lmplugin
        PROPERTIES 
            PREFIX ""
            SUFFIX ${CMAKE_SHARED_LIBRARY_SUFFIX}
            INSTALL_RPATH "${CMAKE_PREFIX_PATH}/lib"
            BUILD_WITH_INSTALL_RPATH TRUE
            INSTALL_RPATH_USE_LINK_PATH TRUE)
    
    if(WIN32)
        set_target_properties(lmplugin PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
    endif()

    install(TARGETS lmplugin DESTINATION "${VMD_PLUGIN_DIR}")
else()
    message(WARNING "VMD not found. Please set \"VMD_PLUGIN_INCLUDE_DIR\" and \"VMD_PLUGIN_DIR\".")
endif()
