cmake_minimum_required(VERSION 3.4.1)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

set(PINE_SOURCES
        pine.cpp
        ruler.cpp
        android.cpp
        jni_bridge.cpp
        art/art_method.cpp
        art/thread.cpp
        art/jit.cpp
        trampoline/trampoline_installer.cpp
        utils/memory.cpp
        utils/scoped_memory_access_protection.cpp
        utils/elf_img.cpp
        utils/well_known_classes.cpp)

if (CMAKE_ANDROID_ARCH_ABI STREQUAL "armeabi-v7a")
    set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -arch armv7")
    set(PINE_SOURCES
            ${PINE_SOURCES}
            trampoline/arch/thumb2.cpp
            trampoline/arch/thumb2.S)
elseif(CMAKE_ANDROID_ARCH_ABI STREQUAL "arm64-v8a")
    set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -arch arm64")
    set(PINE_SOURCES
            ${PINE_SOURCES}
            trampoline/arch/arm64.cpp
            trampoline/arch/arm64.S)
elseif(CMAKE_ANDROID_ARCH_ABI STREQUAL "x86")
    set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -arch x86")
    set(PINE_SOURCES
            ${PINE_SOURCES}
            trampoline/arch/x86.cpp)
else()
    message(FATAL_ERROR "Unsupported cpu abi ${CMAKE_ANDROID_ARCH_ABI} (Only armeabi-v7a and arm64-v8a are now supported)!")
endif()

add_library(pine SHARED ${PINE_SOURCES})

add_library(dobby STATIC IMPORTED)

get_filename_component(current_source_dir ${CMAKE_CURRENT_SOURCE_DIR} ABSOLUTE)

set(external_dir "${current_source_dir}/../../../../external")
get_filename_component(export_dir ${external_dir} ABSOLUTE)
set_target_properties(dobby PROPERTIES IMPORTED_LOCATION ${external_dir}/dobby/${ANDROID_ABI}/libdobby.a)

find_library(log-lib log)

target_link_libraries(pine ${log-lib} dobby)

ENABLE_LANGUAGE(ASM)