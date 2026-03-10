cmake_minimum_required(VERSION 3.16)
project(hello_simple LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_executable(hello_simple main.cpp)

if(MINGW)
    target_link_libraries(hello_simple PRIVATE stdc++exp)
endif()
