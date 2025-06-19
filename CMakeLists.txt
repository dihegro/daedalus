cmake_minimum_required(VERSION 3.20)
project (DaedalusX64 LANGUAGES C CXX ASM)

message(aarch64 " Detected") 

INCLUDE(FindPkgConfig)

add_subdirectory(Source)
