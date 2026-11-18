cmake_minimum_required(VERSION 3.16)
project(TimetableManagementSystem LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(core)
add_subdirectory(cli)
add_subdirectory(tests)

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/gui/CMakeLists.txt")
    add_subdirectory(gui)
endif()
