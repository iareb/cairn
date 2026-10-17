cmake_minimum_required(VERSION 3.21)
project(cairn VERSION 0.1.0 LANGUAGES C)

set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

# Warning: target che non compila nulla, porta solo le opzioni.
# Ogni target lo collega con target_link_libraries(... PRIVATE cairn_warnings).
add_library(cairn_warnings INTERFACE)
if(MSVC)
    target_compile_options(cairn_warnings INTERFACE /W4 /WX)
else()
    target_compile_options(cairn_warnings INTERFACE -Wall -Wextra -Wpedantic -Werror)
endif()

# AddressSanitizer: attivato dal preset "asan". Vale per tutti i target,
# quindi qui si usano i comandi globali invece dei target_*.
option(CAIRN_ASAN "Compila con AddressSanitizer" OFF)
if(CAIRN_ASAN)
    if(MSVC)
        add_compile_options(/fsanitize=address)
        # Con MSVC il runtime di ASan è una DLL accanto a cl.exe, fuori dal PATH
        get_filename_component(CAIRN_MSVC_BIN "${CMAKE_C_COMPILER}" DIRECTORY)
        set(CAIRN_ASAN_DLL "${CAIRN_MSVC_BIN}/clang_rt.asan_dynamic-x86_64.dll")
    else()
        add_compile_options(-fsanitize=address,undefined -fno-omit-frame-pointer)
        add_link_options(-fsanitize=address,undefined)
    endif()
endif()

# Da chiamare per ogni eseguibile (cairn e test): copia accanto all'.exe
# la DLL di ASan, se serve. Negli altri casi non fa nulla.
function(cairn_copy_runtime target)
    if(CAIRN_ASAN_DLL)
        add_custom_command(TARGET ${target} POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CAIRN_ASAN_DLL}" $<TARGET_FILE_DIR:${target}>)
    endif()
endfunction()

# find_package(ZLIB REQUIRED)

# Libreria: da attivare quando esiste src/core/hash.c
# add_library(cairn_core STATIC
#     src/core/hash.c
# )
# target_include_directories(cairn_core PUBLIC src)        # anche i test vedono gli header
# target_link_libraries(cairn_core PRIVATE cairn_warnings)
# target_link_libraries(cairn_core PRIVATE ZLIB::ZLIB)     # dettaglio interno: chi usa cairn_core non vede gli header di zlib

add_executable(cairn src/main.c)
target_link_libraries(cairn PRIVATE cairn_warnings)
cairn_copy_runtime(cairn)
# target_link_libraries(cairn PRIVATE cairn_core)
target_compile_definitions(cairn PRIVATE CAIRN_VERSION="${PROJECT_VERSION}")   # in C: printf("%s\n", CAIRN_VERSION)

enable_testing()                                # Attiva CTest
# add_executable(test_hash test/test_hash.c)
# target_link_libraries(test_hash PRIVATE cairn_core cairn_warnings)
# cairn_copy_runtime(test_hash)
# add_test(NAME test_hash COMMAND test_hash)    # CTest considera fallito il test se l'exit code != 0
