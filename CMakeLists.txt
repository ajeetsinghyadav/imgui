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
)

# Optional demo file (can be excluded for smaller builds)
option(IMGUI_BUILD_DEMO "Build ImGui demo code" ON)
if(IMGUI_BUILD_DEMO)
    list(APPEND IMGUI_SOURCES imgui_demo.cpp)
endif()

# Create ImGui library
add_library(imgui STATIC ${IMGUI_SOURCES})

# Set include directories
target_include_directories(imgui
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<INSTALL_INTERFACE:include>
)

# Platform-specific backend sources
set(IMGUI_BACKEND_SOURCES "")

# OpenGL3 backend (for OpenGL and OpenGL ES)
if(VNE_ENABLE_OPENGL OR VNE_ENABLE_OPENGLES)
    list(APPEND IMGUI_BACKEND_SOURCES
        backends/imgui_impl_opengl3.cpp
    )
    target_compile_definitions(imgui PUBLIC IMGUI_IMPL_OPENGL_LOADER_CUSTOM)
endif()

# Metal backend (for macOS, iOS, visionOS)
if(VNE_ENABLE_METAL)
    if(APPLE)
        list(APPEND IMGUI_BACKEND_SOURCES
            backends/imgui_impl_metal.mm
        )
    endif()
endif()

# Vulkan backend
if(VNE_ENABLE_VULKAN)
    list(APPEND IMGUI_BACKEND_SOURCES
        backends/imgui_impl_vulkan.cpp
    )
endif()

# GLFW platform backend (for desktop platforms)
if(BUILD_GLFW AND (VNE_TARGET_PLATFORM IN_LIST DESKTOP_PLATFORMS))
    list(APPEND IMGUI_BACKEND_SOURCES
        backends/imgui_impl_glfw.cpp
    )
endif()

# Add backend sources to library if any
if(IMGUI_BACKEND_SOURCES)
    target_sources(imgui PRIVATE ${IMGUI_BACKEND_SOURCES})
endif()

# Link required libraries based on backends
if(VNE_ENABLE_OPENGL OR VNE_ENABLE_OPENGLES)
    # OpenGL is handled via GLAD or system loader
    # No explicit linking needed as ImGui uses function pointers
endif()

if(VNE_ENABLE_METAL AND APPLE)
    find_library(METAL_FRAMEWORK Metal)
    find_library(QUARTZCORE_FRAMEWORK QuartzCore)
    if(METAL_FRAMEWORK AND QUARTZCORE_FRAMEWORK)
        target_link_libraries(imgui PRIVATE ${METAL_FRAMEWORK} ${QUARTZCORE_FRAMEWORK})
    endif()
endif()

if(VNE_ENABLE_VULKAN)
    # Vulkan is header-only, no linking needed
endif()

if(BUILD_GLFW AND (VNE_TARGET_PLATFORM IN_LIST DESKTOP_PLATFORMS))
    if(TARGET glfw)
        target_link_libraries(imgui PRIVATE glfw)
    endif()
endif()

# Compiler-specific options
if(MSVC)
    target_compile_options(imgui PRIVATE /W4)
    # Disable specific warnings for ImGui
    target_compile_definitions(imgui PRIVATE _CRT_SECURE_NO_WARNINGS)
else()
    target_compile_options(imgui PRIVATE -Wall -Wextra)
    # Disable specific warnings for ImGui
    target_compile_options(imgui PRIVATE -Wno-unused-parameter -Wno-unused-variable)
endif()

# Set C++ standard
set_target_properties(imgui PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
)

# Create alias for namespace consistency
add_library(VertexNova::imgui ALIAS imgui)
