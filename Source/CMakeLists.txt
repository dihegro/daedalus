cmake_minimum_required(VERSION 3.10)
project(DaedalusX64)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_BUILD_TYPE Release)
set(CMAKE_SYSTEM_PROCESSOR "aarch64")
# Detectando arquitetura e sistema
if(CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64|arm64")
    message(STATUS "Target: Linux aarch64 (S905X3)")

    # Ativar GLES
    option(DAEDALUS_GLES "OpenGLES Renderer" ON)
    option(DAEDALUS_GL "OpenGL Renderer" OFF)

    # SDL para Linux
    option(DAEDALUS_SDL "Enable SDL2" ON)
    option(DAEDALUS_POSIX "Enable POSIX features" ON)

    # Dynarec OFF por enquanto
    option(DAEDALUS_ENABLE_DYNAREC "Enable Dynarec" OFF)

    # Recursos úteis
    option(DAEDALUS_ACCURATE_TMEM "Accurate TMEM" ON)
    option(DAEDALUS_128BIT_MULT "128 Bit Multiplication" ON)
    option(DAEDALUS_ACCURATE_CVT "Accurate Floating Point Conversion" ON)
else()
    message(FATAL_ERROR "Este CMakeLists.txt foi modificado apenas para Linux aarch64 (S905X3)")
endif()

# Endian
include(TestBigEndian)
TEST_BIG_ENDIAN(IS_BIG_ENDIAN)
if(IS_BIG_ENDIAN)
    option(DAEDALUS_ENDIAN_BIG "Big Endian Processor" ON)
else()
    option(DAEDALUS_ENDIAN_LITTLE "Little Endian Processor" ON)
endif()

# Diretórios
include_directories("${PROJECT_SOURCE_DIR}/Source")

# SDL2
if(DAEDALUS_SDL)
    find_package(SDL2 REQUIRED)
    link_libraries(SDL2::SDL2)
endif()

# GLES
if(DAEDALUS_GLES)
    find_package(OpenGL REQUIRED)
    link_libraries(OpenGL::GL)
endif()

# PNG e Zlib
find_package(PNG REQUIRED)
link_libraries(PNG::PNG)

find_package(ZLIB REQUIRED)
link_libraries(ZLIB::ZLIB)

# Diretórios fonte
add_subdirectory(Core)
add_subdirectory(Debug)
add_subdirectory(DynaRec)
add_subdirectory(Graphics)
add_subdirectory(HLEAudio)
add_subdirectory(HLEGraphics)
add_subdirectory(Interface)
add_subdirectory(Input)
add_subdirectory(Math)
add_subdirectory(OSHLE)
add_subdirectory(RomFile)
add_subdirectory(System)
add_subdirectory(Utility)
add_subdirectory(UI)
add_subdirectory(SysPosix)
add_subdirectory(SysGLES)

# Executável
add_executable(daedalus)

target_link_libraries(daedalus PRIVATE ${daed_libs})

# Instalação
install(DIRECTORY ${PROJECT_BINARY_DIR}/../Data/ DESTINATION DaedalusX64)
install(DIRECTORY DESTINATION DaedalusX64/Roms)
install(DIRECTORY DESTINATION DaedalusX64/SaveGames)
install(DIRECTORY DESTINATION DaedalusX64/SaveStates)
