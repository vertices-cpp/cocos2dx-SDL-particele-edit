#[[ 路径: D:\cplusplus\partilce_edit\CMakeLists.txt ]]

cmake_minimum_required(VERSION 3.17)
project(particle_editor C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# -------------------------------------------------------------
# 关键：全局强制 MSVC 将所有源文件按 UTF-8 读取，彻底解决 C4819 和 C2001
# -------------------------------------------------------------
if (MSVC)
    add_compile_options(/utf-8)
endif()

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

add_subdirectory(third_party)

set(PROJ_CLASSES
    Classes/Mat3.cpp
    Classes/Mat3.h
    Classes/ParticleConfig.cpp
    Classes/ParticleConfig.h
    Classes/ParticleExamples.cpp
    Classes/ParticleExamples.h
    Classes/ParticleManager.cpp
    Classes/ParticleManager.h
    Classes/ParticleSystem.cpp
    Classes/ParticleSystem.h
    Classes/Texture2D.cpp
    Classes/Texture2D.h
    Classes/Type.cpp
    Classes/Type.h
)

set(PROJ_MAIN
    example_sdl2_sdlrenderer2/main.cpp
)

add_executable(${PROJECT_NAME}
    ${PROJ_CLASSES}
    ${PROJ_MAIN}
)

target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/Classes
    ${CMAKE_CURRENT_SOURCE_DIR}/example_sdl2_sdlrenderer2
)

target_link_libraries(${PROJECT_NAME} PRIVATE
    third_party
)

if(MSVC OR XCODE)
    source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}" 
        FILES 
            ${PROJ_CLASSES}
            ${PROJ_MAIN}
    )
endif()

# -------------------------------------------------------------
# 资源同步配置 (放到根目录，准确识别工程和目录)
# -------------------------------------------------------------
set(GAME_RES_DIR "${CMAKE_CURRENT_SOURCE_DIR}/Resources")
set(SYNC_SCRIPT "${CMAKE_CURRENT_SOURCE_DIR}/third_party/scripts/sync_folder.py")

if(EXISTS "${GAME_RES_DIR}" AND EXISTS "${SYNC_SCRIPT}")
    find_package(Python2 REQUIRED COMPONENTS Interpreter)

    add_custom_target(SYNC_RESOURCE ALL
        COMMAND "${Python2_EXECUTABLE}"
            "${SYNC_SCRIPT}"
            -s "${GAME_RES_DIR}"
            -d "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$<CONFIG>/Resources"
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
        COMMENT "Syncing Resources using sync_folder.py..."
    )

    # 确保主程序构建时触发资源同步目标
    add_dependencies(${PROJECT_NAME} SYNC_RESOURCE)
endif()
