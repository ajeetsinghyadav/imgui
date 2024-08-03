#==============================================================================
# Copyright (c) 2024 Ajeet Singh Yadav. All rights reserved.
# Licensed under the Apache License, Version 2.0 (the "License")
#
# Author:    Ajeet Singh Yadav
# Created:   JULY-2024
#
# Autodoc:   yes
#==============================================================================

# Minimum required CMake version
cmake_minimum_required(VERSION 3.16)

# Project declaration
project(imgui)

#==============================================================================
#                              Source and Headers                              #
#==============================================================================

# Define header files
set(INCLUDE_FILES
    imconfig.h
    imgui.h
    imgui_internal.h
    imstb_rectpack.h
    imstb_textedit.h
    imstb_truetype.h
)

# Define source files
set(SOURCE_FILES
    imgui.cpp
    imgui_draw.cpp
    imgui_widgets.cpp
    imgui_demo.cpp
    imgui_tables.cpp
)

#==============================================================================
#                                 Library                                      #
#==============================================================================

# Create a static library
add_library(imgui STATIC ${INCLUDE_FILES} ${SOURCE_FILES})

# Specify include directories for imgui
target_include_directories(imgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
