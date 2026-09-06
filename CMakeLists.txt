cmake_minimum_required (VERSION 4.4)

project (imgui-cmake
    VERSION 1.92.9
    LANGUAGES CXX
)

include (CMakeDependentOption)
include (FetchContent)

set (IMGUI_STATIC_DEFAULT ON)
set (IMGUI_SHARED_DEFAULT OFF)
set (IMGUI_BACKEND_SDL3_DEFAULT OFF)
set (IMGUI_BACKEND_SDLGPU3_DEFAULT OFF)

set (IMGUI_STATIC_AVAILABLE ON)
set (IMGUI_SHARED_AVAILABLE ON)
set (IMGUI_BACKEND_SDL3_AVAILABLE ON)
set (IMGUI_BACKEND_SDLGPU3_AVAILABLE ON)

cmake_dependent_option (IMGUI_STATIC  "Build a static version of the library" ${IMGUI_STATIC_DEFAULT} ${IMGUI_STATIC_AVAILABLE} OFF)
cmake_dependent_option (IMGUI_SHARED  "Build a shared version of the library" ${IMGUI_SHARED_DEFAULT} ${IMGUI_SHARED_AVAILABLE} OFF)
cmake_dependent_option (IMGUI_SDL3    "Build a sdl3 backend of the library" ${IMGUI_BACKEND_SDL3_DEFAULT} ${IMGUI_BACKEND_SDL3_AVAILABLE} OFF)
cmake_dependent_option (IMGUI_SDLGPU3 "Build a sdlgpu3 backend of the library" ${IMGUI_BACKEND_SDLGPU3_DEFAULT} ${IMGUI_BACKEND_SDLGPU3_AVAILABLE} OFF)

FetchContent_Declare (
  SDL3
  GIT_REPOSITORY https://github.com/libsdl-org/SDL
  GIT_TAG        release-3.4.16
)

FetchContent_MakeAvailable (SDL3)

set (IMGUI_SOURCES
    imgui.cpp
    imgui_demo.cpp
    imgui_draw.cpp
    imgui_tables.cpp
    imgui_widgets.cpp
)

if (IMGUI_SDL3)
    list (APPEND IMGUI_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/backends/imgui_impl_sdl3.cpp)
endif ()

if (IMGUI_SDLGPU3)
    list (APPEND IMGUI_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/backends/imgui_impl_sdlgpu3.cpp)
endif ()

if (IMGUI_STATIC)
    add_library (imgui_static STATIC ${IMGUI_SOURCES})
    set_target_properties (imgui_static PROPERTIES OUTPUT_NAME "imgui")
    target_include_directories (imgui_static PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/backends)
    if (IMGUI_SDL3 OR IMGUI_SDLGPU3)
        target_link_libraries(imgui_static PRIVATE SDL3::SDL3)
    endif ()
endif ()

if (IMGUI_SHARED)
    add_library (imgui_shared SHARED ${IMGUI_SOURCES})
    set_target_properties (imgui_shared PROPERTIES OUTPUT_NAME "imgui")
    target_include_directories (imgui_shared PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/backends)
    if (WIN32)
        target_compile_definitions (imgui_shared PUBLIC IMGUI_API=__declspec(dllexport))
    endif ()
    if (IMGUI_SDL3 OR IMGUI_SDLGPU3)
        target_link_libraries (imgui_shared PRIVATE SDL3::SDL3)
    endif ()
endif ()