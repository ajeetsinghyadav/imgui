#==============================================================================
# Copyright (c) 2025 Ajeet Singh Yadav. All rights reserved.
# Licensed under the Apache License, Version 2.0 (the "License")
#
# Author:    Ajeet Singh Yadav
# Created:   December 2025
#
# Autodoc:   yes
#==============================================================================

cmake_minimum_required(VERSION 3.16)

# Core ImGui source files
set(IMGUI_SOURCES
    imgui.cpp
    imgui_draw.cpp
    imgui_tables.cpp
    imgui_widgets.cpp
    imgui_demo.cpp
)

if(VNE_ENABLE_OPENGL OR VNE_ENABLE_OPENGLES)
    list(APPEND IMGUI_SOURCES backends/imgui_impl_opengl3.cpp)
endif()

if(VNE_ENABLE_METAL)
    list(APPEND IMGUI_SOURCES backends/imgui_impl_metal.mm)
endif()

if(VNE_ENABLE_VULKAN)
    list(APPEND IMGUI_SOURCES backends/imgui_impl_vulkan.cpp)
endif()

# Create ImGui library
add_library(imgui STATIC ${IMGUI_SOURCES})

# Set include directories for the library
target_include_directories(imgui
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<INSTALL_INTERFACE:include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/backends>
        $<INSTALL_INTERFACE:include/backends>
)

# Link required libraries based on backends
if(VNE_ENABLE_OPENGL OR VNE_ENABLE_OPENGLES)
    find_package(OpenGL)
    if(OpenGL_FOUND)
        target_link_libraries(imgui PRIVATE OpenGL::GL)
    endif()
endif()

if(VNE_ENABLE_METAL AND APPLE)
    find_library(METAL_FRAMEWORK Metal)
    find_library(QUARTZCORE_FRAMEWORK QuartzCore)
    if(METAL_FRAMEWORK AND QUARTZCORE_FRAMEWORK)
        target_link_libraries(imgui PRIVATE ${METAL_FRAMEWORK} ${QUARTZCORE_FRAMEWORK})
    endif()
    # Enable ARC for Metal backend file to fix bridge cast warnings
    set_source_files_properties(backends/imgui_impl_metal.mm PROPERTIES
        COMPILE_FLAGS "-fobjc-arc"
    )
endif()

# Compiler-specific options
if(MSVC)
    target_compile_options(imgui PRIVATE /W4)
    target_compile_definitions(imgui PRIVATE _CRT_SECURE_NO_WARNINGS)
else()
    target_compile_options(imgui PRIVATE -Wall -Wextra)
    target_compile_options(imgui PRIVATE -Wno-unused-parameter -Wno-unused-variable)
    target_compile_options(imgui PRIVATE -Wno-arc-bridge-casts-disallowed-in-nonarc)
endif()

# Set C++ standard
set_target_properties(imgui PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
)

# Create alias for namespace consistency
add_library(VertexNova::imgui ALIAS imgui)
