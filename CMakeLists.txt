cmake_minimum_required(VERSION 3.22)
project(pacientecero)

set(CMAKE_CXX_STANDARD 20)

find_package(raylib CONFIG REQUIRED)

add_executable(pacientecero
    src/main.cpp
)

target_link_libraries(pacientecero PRIVATE raylib)
