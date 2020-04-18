cmake_minimum_required(VERSION 3.16)

if(TARGET tiny-js)
    return()
endif()

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

include(cotire)

set(project_name tiny-js)

project(tiny-js C CXX)

set(CMAKE_CXX_STANDARD 17 CACHE STRING "Set to 11, 14 or 17 to enable C++11, C++14 or C++17 builds, respectively." FORCE)
set(CMAKE_C_STANDARD 11 CACHE STRING "Set to 99 or 11 to enable C99 or C11 builds, respectively." FORCE)
set_property(GLOBAL PROPERTY CXX_STANDARD_REQUIRED ON)

add_compile_definitions("__STDC_WANT_LIB_EXT1__=1")

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

include_directories(${CMAKE_BINARY_DIR})

if(WIN32 OR WIN64)
    add_compile_definitions("WIN32_LEAN_AND_MEAN" "_CRT_SECURE_NO_WARNINGS" "NOMINMAX")
    if(MSVC)
		add_compile_options("/source-charset:utf-8" "/EHsc" "/Zc:threadSafeInit-")
	endif()
else()
    if(UNIX)
        if(APPLE)
            add_compile_options("-m64" "-fPIC" "-march=native")
            set(CMAKE_MACOSX_RPATH 1 CACHE STRING "CMAKE_MACOSX_RPATH" FORCE)
        else()
            add_compile_options("-fPIC")
            if(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86)|(X86)|(amd64)|(AMD64)")
                add_compile_options("-m64" "-march=westmere")
            endif()
        endif()
    endif()
endif()

file(GLOB TINY_JS_HEADER_FILES
	${CMAKE_CURRENT_LIST_DIR}/TinyJS.h
)

file(GLOB TINY_JS_SOURCE_FILES
	${CMAKE_CURRENT_LIST_DIR}/TinyJS.cpp
    ${CMAKE_CURRENT_LIST_DIR}/TinyJS_Functions.cpp
    ${CMAKE_CURRENT_LIST_DIR}/TinyJS_MathFunctions.cpp
)

add_library(tiny-js STATIC ${TINY_JS_HEADER_FILES} ${TINY_JS_SOURCE_FILES})
target_include_directories(tiny-js INTERFACE
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/>"
    $<INSTALL_INTERFACE:include/tiny-js>
)

add_executable(tiny-js-cli Script.cpp)

target_link_libraries(tiny-js-cli
    PUBLIC tiny-js
)

#############################################
# Unit tests

# enable CTest testing
enable_testing()

add_executable(tiny-js-tests run_tests.cpp)

target_link_libraries(tiny-js-tests
    PUBLIC tiny-js
)

add_custom_command(
	TARGET tiny-js-tests POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy_directory
			${CMAKE_CURRENT_SOURCE_DIR}/tests
			${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/tests)

add_test(NAME test_all COMMAND tiny-js-tests)