cmake_minimum_required(VERSION 3.13 FATAL_ERROR)
project(c4core
    DESCRIPTION "Multiplatform low-level C++ utilities"
    HOMEPAGE_URL "https://github.com/biojppm/c4core"
    LANGUAGES CXX)
include(./cmake/c4Project.cmake)
include(./compat.cmake)

c4_project(VERSION 0.2.6
    AUTHOR "Joao Paulo Magalhaes <dev@jpmag.me>")

option(C4CORE_INSTALL "create install target" ON)
option(C4CORE_WITH_FASTFLOAT "use fastfloat to parse floats" ON)
option(C4CORE_NO_DEBUG_BREAK "disable use of debug break even in debug builds" OFF)

set(C4CORE_SRC_FILES
    c4/allocator.hpp
    c4/base64.hpp
    c4/base64.cpp
    c4/blob.hpp
    c4/bitmask.hpp
    c4/charconv.hpp
    c4/c4_pop.hpp
    c4/c4_push.hpp
    c4/char_traits.cpp
    c4/char_traits.hpp
    c4/common.hpp
    c4/compiler.hpp
    c4/config.hpp
    c4/cpu.hpp
    c4/ctor_dtor.hpp
    c4/dump.hpp
    c4/enum.hpp
    c4/error.cpp
    c4/error.hpp
    c4/export.hpp
    c4/format.hpp
    c4/format.cpp
    c4/gcc-4.8.hpp
    c4/hash.hpp
    c4/language.hpp
    c4/language.cpp
    c4/memory_resource.cpp
    c4/memory_resource.hpp
    c4/memory_util.cpp
    c4/memory_util.hpp
    c4/platform.hpp
    c4/preprocessor.hpp
    c4/restrict.hpp
    c4/span.hpp
    c4/std/std.hpp
    c4/std/std_fwd.hpp
    c4/std/string.hpp
    c4/std/string_fwd.hpp
    c4/std/string_view.hpp
    c4/std/tuple.hpp
    c4/std/vector.hpp
    c4/std/vector_fwd.hpp
    c4/substr.hpp
    c4/substr_fwd.hpp
    c4/szconv.hpp
    c4/type_name.hpp
    c4/types.hpp
    c4/unrestrict.hpp
    c4/utf.hpp
    c4/utf.cpp
    c4/windows.hpp
    c4/version.hpp
    c4/version.cpp
    c4/windows_pop.hpp
    c4/windows_push.hpp
    c4/c4core.natvis
    #
    c4/ext/debugbreak/debugbreak.h
    c4/ext/rng/rng.hpp
    c4/ext/sg14/inplace_function.h
    )
if(C4CORE_WITH_FASTFLOAT)
    list(APPEND C4CORE_SRC_FILES
        c4/ext/fast_float.hpp
        c4/ext/fast_float_all.h
        )
endif()
set(C4CORE_AMALGAMATED ${C4CORE_SRC_DIR}/../src_singleheader/c4/c4core_all.hpp)
list(TRANSFORM C4CORE_SRC_FILES PREPEND "${C4CORE_SRC_DIR}/" OUTPUT_VARIABLE C4CORE_SRC_FILES_FULL)

add_custom_target(c4core-amalgamate
    python ${CMAKE_CURRENT_LIST_DIR}/tools/amalgamate.py ${C4CORE_AMALGAMATED}
    COMMENT "${CMAKE_CURRENT_LIST_DIR}/tools/amalgamate.py ${C4CORE_AMALGAMATED}"
    BYPRODUCTS ${C4CORE_AMALGAMATED}
    DEPENDS ${C4CORE_SRC_FILES_FULL}
    )

c4_add_library(c4core
    INC_DIRS
       $<BUILD_INTERFACE:${C4CORE_SRC_DIR}> $<INSTALL_INTERFACE:include>
    SOURCE_ROOT ${C4CORE_SRC_DIR}
    SOURCES ${C4CORE_SRC_FILES}
)
if(NOT C4CORE_WITH_FASTFLOAT)
    target_compile_definitions(c4core PUBLIC -DC4CORE_NO_FAST_FLOAT)
endif()
if(C4CORE_NO_DEBUG_BREAK)
    target_compile_definitions(c4core PUBLIC -DC4_NO_DEBUG_BREAK)
endif()


#-------------------------------------------------------

if(C4CORE_INSTALL)
    c4_install_target(c4core)
    c4_install_exports()
endif()

c4_add_dev_targets()

if(C4CORE_INSTALL)
    c4_pack_project(TYPE LIBRARY)
endif()


#-------------------------------------------------------
# clang-tidy

function(c4core_setup_clang_tidy rootdir)
    get_target_property(srcs c4core SOURCES)
    string(REPLACE "${rootdir}/" "./" srcs "${srcs}")
    set(exclude ./c4/ext/.* .natvis)
    foreach(e ${exclude})
        list(FILTER srcs EXCLUDE REGEX ${e})
    endforeach()
    set(cmd ${srcs} -p ${CMAKE_BINARY_DIR}
            "--config-file=${rootdir}/.clang-tidy"
            "--header-filter=${rootdir}/src/c4/.*pp|${rootdir}/src/c4/std/.*pp")
    add_custom_target(c4core-clang-tidy
        COMMAND ${CMAKE_COMMAND} -E cat ${CMAKE_BINARY_DIR}/compile_commands.json
        COMMAND ${CLANG_TIDY} --version
        COMMAND ${CLANG_TIDY} ${cmd} --dump-config
        COMMAND ${CLANG_TIDY} ${cmd} --list-checks
        COMMAND ${CLANG_TIDY} ${cmd}
        WORKING_DIRECTORY ${rootdir}
        VERBATIM
    )
endfunction()

find_program(CLANG_TIDY clang-tidy)
if(CLANG_TIDY)
    c4core_setup_clang_tidy(${CMAKE_CURRENT_LIST_DIR})
endif()
