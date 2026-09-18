#[[ 路径: D:\cplusplus\partilce_edit\third_party\CMakeLists.txt ]]

cmake_minimum_required(VERSION 3.17)
project(third_party_libs C CXX)

# 1. 包含第三方各个子模块的 CMakeLists.txt 配置
include(imgui-1.89.9/CMakeLists.txt)
include(ImGuiFileDialog/CMakeLists.txt)
include(tinyxml2/CMakeLists.txt)

# 2. 汇总所有子模块的源码与头文件
set(THIRD_PARTY_HEADER
    ${imgui_header}
    ${ImGuiFileDialog_header}
    ${tinyxml2_header}
)

set(THIRD_PARTY_SRC
    ${imgui_src}
    ${ImGuiFileDialog_src}
    ${tinyxml2_src}
)

set(ALL_THIRD_PARTY_FILES ${THIRD_PARTY_HEADER} ${THIRD_PARTY_SRC})

# 3. 创建统一的第三方静态库
add_library(third_party STATIC ${ALL_THIRD_PARTY_FILES})

# 4. 配置头文件搜索路径 (使用 PUBLIC 自动透传给主工程)
target_include_directories(third_party PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui-1.89.9
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui-1.89.9/backends
    ${CMAKE_CURRENT_SOURCE_DIR}/ImGuiFileDialog
    ${CMAKE_CURRENT_SOURCE_DIR}/tinyxml2
    ${CMAKE_CURRENT_SOURCE_DIR}/SDL2/SDL2-2.0.20/include
    ${CMAKE_CURRENT_SOURCE_DIR}/SDL2/SDL2_image-2.0.5/include
)

# 5. 自动根据编译目标架构（x86 / x64）选择正确的 SDL2 库路径
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(ARCH_DIR "x64")
else()
    set(ARCH_DIR "x86")
endif()

set(SDL2_LIB_DIR "${CMAKE_CURRENT_SOURCE_DIR}/SDL2/SDL2-2.0.20/lib/${ARCH_DIR}")
set(SDL2_IMAGE_LIB_DIR "${CMAKE_CURRENT_SOURCE_DIR}/SDL2/SDL2_image-2.0.5/lib/${ARCH_DIR}")

# 补充链接 Windows 原生依赖库
target_link_libraries(third_party PUBLIC
    ${SDL2_LIB_DIR}/SDL2.lib
    ${SDL2_LIB_DIR}/SDL2main.lib
    ${SDL2_IMAGE_LIB_DIR}/SDL2_image.lib 
)

# 6. 自动拷贝 SDL2 相关 DLL 到生成目录
file(GLOB_RECURSE SDL2_DLLS 
    "${CMAKE_CURRENT_SOURCE_DIR}/SDL2/*/lib/${ARCH_DIR}/*.dll"
)

if(SDL2_DLLS)
    add_custom_command(TARGET third_party POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            ${SDL2_DLLS}
            "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$<CONFIG>/"
        COMMAND_EXPAND_LISTS
        COMMENT "Copying all found SDL2 DLLs to executable folder..."
    )
endif()

# 7. 按物理磁盘目录配置 VS 虚拟筛选器
if(MSVC OR XCODE)
    source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}" FILES ${ALL_THIRD_PARTY_FILES})
endif()