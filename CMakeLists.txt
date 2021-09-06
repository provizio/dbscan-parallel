cmake_minimum_required(VERSION 3.0.0)
project(dbscan-parallel VERSION 0.1.0)

include(CTest)
enable_testing()

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

set(CMAKE_CXX_FLAGS "-std=c++17 -pthread -O3 -fPIC")

add_library(dbscan-parallel src/Caller.cpp)

target_include_directories(dbscan-parallel PUBLIC src)

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
