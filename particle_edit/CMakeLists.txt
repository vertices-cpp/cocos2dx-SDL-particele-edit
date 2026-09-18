cmake_minimum_required(VERSION 3.17)
project(particle_editor CXX C)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(MSVC)
    add_compile_options(/utf-8)
endif()

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(ARCH_DIR "x64")
else()
    set(ARCH_DIR "x86")
endif()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

# 第三方库
add_subdirectory(third_party)

# ============================================================
# 主 exe：统一扫描 Classes + example_sdl2_sdlrenderer2
# ============================================================
# Classes 递归扫（将来可能有子目录）
file(GLOB_RECURSE PROJ_CLASSES CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/Classes/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/Classes/*.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/Classes/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/Classes/*.hpp"
)

# example_sdl2_sdlrenderer2 不递归（避免扫进 CMakeFiles/ 里的临时 cpp）
file(GLOB PROJ_MAIN CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/example_sdl2_sdlrenderer2/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/example_sdl2_sdlrenderer2/*.h"
)

set(PROJ_SOURCES ${PROJ_CLASSES} ${PROJ_MAIN})

add_executable(${PROJECT_NAME} ${PROJ_SOURCES})

target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/Classes
    ${CMAKE_CURRENT_SOURCE_DIR}/example_sdl2_sdlrenderer2
)

target_link_libraries(${PROJECT_NAME} PRIVATE
    sdl2_libs
    imgui
    tinyxml2
    ImGuiFileDialog
)

# ★ 全部塞进 Source Files，保留目录树
source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}"
             PREFIX "Source Files"
             FILES ${PROJ_SOURCES})

# 拷贝 SDL2 DLL
file(GLOB_RECURSE SDL2_DLLS
    "${CMAKE_CURRENT_SOURCE_DIR}/third_party/SDL2/*/lib/${ARCH_DIR}/*.dll"
)
if(SDL2_DLLS)
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            ${SDL2_DLLS}
            "$<TARGET_FILE_DIR:${PROJECT_NAME}>"
        COMMENT "Copying SDL2 DLLs..."
    )
endif()
