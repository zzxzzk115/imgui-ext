cmake_minimum_required(VERSION 3.14 FATAL_ERROR)

# create project
project(ImGuiExt
        VERSION 0.1
        DESCRIPTION "imgui-ext = imgui + 3rd party libraries + extension functions"
        LANGUAGES CXX)

# set options
option(IMGUIEXT_WITH_BACKEND_SDL2 "Enable ImGui SDL2 backend" OFF)
option(IMGUIEXT_WITH_BACKEND_GLFW "Enable ImGui GLFW backend" OFF)
option(IMGUIEXT_WITH_BACKEND_GL3 "Enable ImGui OpenGL3 backend" OFF)
option(IMGUIEXT_WITH_BACKEND_GL2 "Enable ImGui OpenGL2 backend" OFF)
option(IMGUIEXT_WITH_BACKEND_VULKAN "Enable ImGui Vulkan backend" OFF)
option(IMGUIEXT_WITH_BACKEND_DX11 "Enable ImGui DirectX11 backend" OFF)
option(IMGUIEXT_WITH_BACKEND_DX12 "Enable ImGui DirectX12 backend" OFF)
option(IMGUIEXT_WITH_BACKEND_WIN32 "Enable ImGui Win32 backend" OFF)
option(IMGUIEXT_WITH_BACKEND_METAL "Enable ImGui Metal backend" OFF)
option(IMGUIEXT_WITH_BACKEND_ALLEGRO5 "Enable ImGui Allegro5 backend" OFF)
option(IMGUIEXT_WITH_BACKEND_SFML "Enable ImGui SFML backend" OFF)

# C++ version: C++23
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# generate compile_commands.json
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# add dependencies
include(cmake/CPM.cmake)
include(cmake/common.cmake)

# add imgui_docking
CPMAddPackage(
        URI "gh:ocornut/imgui@1.91.9b-docking"
        OPTIONS "IMGUI_BUILD_EXAMPLES OFF" "IMGUI_BUILD_TESTS OFF"
)

set(IMGUI_BACKEND_SRCS "")
set(IMGUI_BACKEND_LIBS "")

# SDL2 backend
if (IMGUIEXT_WITH_BACKEND_SDL2)
    CPMAddPackage(
            URI "gh:libsdl-org/SDL#release-2.32.8"
            OPTIONS "SDL_BUILD_EXAMPLES OFF" "SDL_BUILD_TESTS OFF"
    )
    list(APPEND IMGUI_BACKEND_SRCS ${imgui_SOURCE_DIR}/backends/imgui_impl_sdl2.cpp)
    list(APPEND IMGUI_BACKEND_LIBS SDL2::SDL2)
endif ()

# GLFW backend
if (IMGUIEXT_WITH_BACKEND_GLFW)
    CPMAddPackage(
            URI "gh:glfw/glfw#3.4"
            OPTIONS "GLFW_BUILD_EXAMPLES OFF" "GLFW_BUILD_TESTS OFF" "GLFW_BUILD_DOCS OFF"
    )
    list(APPEND IMGUI_BACKEND_SRCS ${imgui_SOURCE_DIR}/backends/imgui_impl_glfw.cpp)
    list(APPEND IMGUI_BACKEND_LIBS glfw)
endif ()

# OpenGL backends
if (IMGUIEXT_WITH_BACKEND_GL3 OR IMGUIEXT_WITH_BACKEND_GL2)
    find_package(OpenGL REQUIRED)
    list(APPEND IMGUI_BACKEND_LIBS OpenGL::GL)
endif ()
if (IMGUIEXT_WITH_BACKEND_GL3)
    list(APPEND IMGUI_BACKEND_SRCS ${imgui_SOURCE_DIR}/backends/imgui_impl_opengl3.cpp)
endif ()
if (IMGUIEXT_WITH_BACKEND_GL2)
    list(APPEND IMGUI_BACKEND_SRCS ${imgui_SOURCE_DIR}/backends/imgui_impl_opengl2.cpp)
endif ()

# Vulkan backend
if (IMGUIEXT_WITH_BACKEND_VULKAN)
    find_package(Vulkan REQUIRED)
    list(APPEND IMGUI_BACKEND_SRCS ${imgui_SOURCE_DIR}/backends/imgui_impl_vulkan.cpp)
    list(APPEND IMGUI_BACKEND_LIBS Vulkan::Vulkan)
endif ()

# DirectX11 backend
if (IMGUIEXT_WITH_BACKEND_DX11)
    list(APPEND IMGUI_BACKEND_SRCS ${imgui_SOURCE_DIR}/backends/imgui_impl_dx11.cpp)
    list(APPEND IMGUI_BACKEND_LIBS d3d11)
endif ()

# DirectX12 backend
if (IMGUIEXT_WITH_BACKEND_DX12)
    list(APPEND IMGUI_BACKEND_SRCS ${imgui_SOURCE_DIR}/backends/imgui_impl_dx12.cpp)
    list(APPEND IMGUI_BACKEND_LIBS d3d12 dxgi dxguid)
endif ()

# Win32 backend
if (IMGUIEXT_WITH_BACKEND_WIN32)
    list(APPEND IMGUI_BACKEND_SRCS ${imgui_SOURCE_DIR}/backends/imgui_impl_win32.cpp)
    list(APPEND IMGUI_BACKEND_LIBS user32 gdi32)
endif ()

# Metal backend (macOS/iOS)
if (IMGUIEXT_WITH_BACKEND_METAL)
    find_library(METAL_FRAMEWORK Metal)
    find_library(FOUNDATION_FRAMEWORK Foundation)
    list(APPEND IMGUI_BACKEND_SRCS ${imgui_SOURCE_DIR}/backends/imgui_impl_metal.mm)
    list(APPEND IMGUI_BACKEND_LIBS ${METAL_FRAMEWORK} ${FOUNDATION_FRAMEWORK})
endif ()

# Allegro5 backend
if (IMGUIEXT_WITH_BACKEND_ALLEGRO5)
    CPMAddPackage(
            URI "gh:liballeg/allegro5@5.2.10.1"
            OPTIONS "ALLEGRO_BUILD_EXAMPLES OFF" "ALLEGRO_BUILD_TESTS OFF" "ALLEGRO_BUILD_DOCS OFF"
    )
    list(APPEND IMGUI_BACKEND_SRCS ${imgui_SOURCE_DIR}/backends/imgui_impl_allegro5.cpp)
    list(APPEND IMGUI_BACKEND_LIBS allegro allegro_main)
endif ()

# SFML backend
if (IMGUIEXT_WITH_BACKEND_SFML)
    CPMAddPackage(
            URI "gh:SFML/SFML#3.0.1"
            OPTIONS "SFML_BUILD_EXAMPLES OFF" "SFML_BUILD_TESTS OFF" "SFML_BUILD_DOCS OFF"
    )
    list(APPEND IMGUI_BACKEND_SRCS ${imgui_SOURCE_DIR}/backends/imgui_impl_sfml.cpp)
    list(APPEND IMGUI_BACKEND_LIBS sfml-graphics sfml-window sfml-system)
endif ()

# https://github.com/cpm-cmake/CPM.cmake/issues/368
add_library(imgui STATIC
        ${imgui_SOURCE_DIR}/imgui.cpp
        ${imgui_SOURCE_DIR}/imgui_demo.cpp
        ${imgui_SOURCE_DIR}/imgui_draw.cpp
        ${imgui_SOURCE_DIR}/imgui_widgets.cpp
        ${imgui_SOURCE_DIR}/imgui_tables.cpp
        ${IMGUI_BACKEND_SRCS}
)
target_include_directories(imgui PUBLIC ${imgui_SOURCE_DIR})
target_link_libraries(imgui PUBLIC ${IMGUI_BACKEND_LIBS})
target_compile_definitions(imgui PUBLIC -DIMGUI_DISABLE_OBSOLETE_FUNCTIONS) # optional imgui setting
#set_target_properties(imgui PROPERTIES FOLDER third-party) # optoinal IDE dir

set(TARGET_NAME ImGuiExt)

# add library target
add_library(${TARGET_NAME} STATIC imgui_compiler.cpp)

# link libraries
target_link_libraries(${TARGET_NAME} PUBLIC imgui)
target_set_common_properties(${TARGET_NAME})