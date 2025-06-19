# DaedalusX64 CMake Build File - S905X3/ARM64 Optimized
set(CMAKE_CXX_STANDARD 17)  # Alterado de 23 para 17 para compatibilidade

# Configurações específicas para ARM64
if(CMAKE_SYSTEM_PROCESSOR STREQUAL "aarch64" AND CMAKE_SYSTEM_NAME STREQUAL "Linux")
    message("ARM64 Linux Detected - Configurando para S905X3")
    
    # Configurações de otimização para Cortex-A55/A73
    add_compile_options(
        -march=armv8-a+crc+fp+simd
        -mtune=cortex-a73.cortex-a53
        -mno-outline-atomics
        -O3
        -fomit-frame-pointer
    )
    add_link_options(-Wl,-z,noexecstack)
    
    # Configurações específicas para GLES
    set(DAEDALUS_GLES ON CACHE BOOL "OpenGL ES Renderer" FORCE)
    set(DAEDALUS_GL OFF CACHE BOOL "OpenGL Renderer" FORCE)
    set(DAEDALUS_SDL ON CACHE BOOL "SDL Build" FORCE)
    set(DAEDALUS_POSIX ON CACHE BOOL "Posix specific options" FORCE)
    
    # Habilitar dynarec para ARM64
    set(DAEDALUS_ENABLE_DYNAREC ON CACHE BOOL "Enable Dynarec" FORCE)
    message("ARM64 Dynarec Enabled")
endif()

# Configurações básicas de build
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    message("Debug Build")
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION FALSE)
    add_compile_definitions(DAEDALUS_CONFIG_VERSION="Dev")
else()
    message("Release Build")
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
    add_compile_definitions(DAEDALUS_CONFIG_VERSION="Release")
    add_compile_definitions(NDEBUG)
endif()

# Configurações de plataforma
option(DAEDALUS_ACCURATE_TMEM "Accurate TMEM" ON)
option(DAEDALUS_128BIT_MULT "128 Bit Multiplication" ON)
option(DAEDALUS_COMPRESSED_ROM_SUPPORT "Enable Compressed ROM Support" ON)

# Configurações de renderização
if(DAEDALUS_GLES)
    find_package(OpenGL REQUIRED)
    find_library(GLESv2_LIBRARY GLESv2)
    find_path(GLES2_INCLUDE_DIR GLES2/gl2.h)
    
    if(NOT TARGET OpenGLES::GLES2)
        add_library(OpenGLES::GLES2 UNKNOWN IMPORTED)
        set_target_properties(OpenGLES::GLES2 PROPERTIES
            IMPORTED_LOCATION "${GLESv2_LIBRARY}"
            INTERFACE_INCLUDE_DIRECTORIES "${GLES2_INCLUDE_DIR}"
        )
    endif()
endif()

# Configurações do SDL
find_package(SDL2 REQUIRED)
find_package(SDL2_ttf)
link_libraries(SDL2::SDL2)
if(SDL2_ttf_FOUND)
    link_libraries(SDL2_ttf::SDL2_ttf)
endif()

# Bibliotecas essenciais
find_package(PNG REQUIRED)
find_package(ZLIB REQUIRED)

# Estrutura de diretórios
include_directories(BEFORE 
    ${PROJECT_SOURCE_DIR}/Source
    ${SDL2_INCLUDE_DIRS}
    ${GLES2_INCLUDE_DIR}
)

# Subdiretórios principais
add_subdirectory(Core)
add_subdirectory(DynaRec)
add_subdirectory(HLEGraphics)
add_subdirectory(System)

# Subdiretórios específicos para GLES
if(DAEDALUS_GLES)
    add_subdirectory(SysGLES)
endif()

# Executável principal
add_executable(daedalus ${plat_main})
target_link_libraries(daedalus PRIVATE
    ${daed_libs}
    ${default_libraries}
    OpenGLES::GLES2
    SDL2::SDL2
    ${SDL2_TTF_LIBRARIES}
    PNG::PNG
    ZLIB::ZLIB
)

# Instalação
install(DIRECTORY ${PROJECT_BINARY_DIR}/../Data/ DESTINATION DaedalusX64)
install(DIRECTORY DESTINATION DaedalusX64/Roms)
install(DIRECTORY DESTINATION DaedalusX64/SaveGames)
install(DIRECTORY DESTINATION DaedalusX64/SaveStates)
