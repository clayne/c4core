if(NOT (CMAKE_VERSION VERSION_LESS "4.0.0"))
    c4_log(STATUS "setting CMAKE_POLICY_VERSION_MINIMUM=3.5 to build doctest")
    set(CMAKE_POLICY_VERSION_MINIMUM 3.5 CACHE STRING "" FORCE)
endif()
c4_setup_testing(DOCTEST)

c4_add_library(c4core-_libtest LIBRARY_TYPE STATIC
    SOURCES
        c4/test.hpp
        c4/main.cpp
        c4/libtest/test.cpp
        c4/libtest/archetypes.cpp
        c4/libtest/archetypes.hpp
        c4/libtest/supprwarn_push.hpp
        c4/libtest/supprwarn_pop.hpp
    LIBS c4core doctest
    DEFS
        # doctest does not deal well with template operator<< for substr
        # see https://github.com/onqtam/doctest/pull/431
        -DC4_SUBSTR_NO_OSTREAM_LSHIFT
    INC_DIRS ${CMAKE_CURRENT_LIST_DIR}
    FOLDER test
    )
if(C4CORE_DEFINED_FROM_SINGLEHEADER)
    target_compile_definitions(c4core-_libtest PUBLIC -DC4CORE_SINGLE_HEADER)
endif()

function(c4core_test name)
    c4_add_executable(c4core-test-${name}
        SOURCES ${ARGN}
        INC_DIRS ${CMAKE_CURRENT_LIST_DIR}
        LIBS c4core-_libtest
        FOLDER test)
    c4_add_test(c4core-test-${name} ARGS --duration=on --no-version=on)
endfunction()

c4core_test(version          test_version.cpp)
c4core_test(preprocessor     test_preprocessor.cpp)
c4core_test(type_name        test_type_name.cpp)
c4core_test(types            test_types.cpp)
c4core_test(szconv           test_szconv.cpp)
c4core_test(error            test_error.cpp)
c4core_test(error_exception  test_error_exception.cpp)
c4core_test(blob             test_blob.cpp)
c4core_test(memory_util      test_memory_util.cpp)
c4core_test(memory_resource  test_memory_resource.cpp)
c4core_test(allocator        test_allocator.cpp)
c4core_test(ctor_dtor        test_ctor_dtor.cpp)
c4core_test(chartraits       test_char_traits.cpp)
c4core_test(enum             test_enum.cpp)
c4core_test(bitmask          test_bitmask.cpp)
c4core_test(span             test_span.cpp)
c4core_test(substr           test_substr.cpp)
c4core_test(charconv         test_charconv.cpp test_numbers.hpp)
c4core_test(utf              test_utf.cpp)
c4core_test(format           test_format.cpp)
c4core_test(dump             test_dump.cpp)
c4core_test(base64           test_base64.cpp)
c4core_test(std_string       test_std_string.cpp)
c4core_test(std_vector       test_std_vector.cpp)
