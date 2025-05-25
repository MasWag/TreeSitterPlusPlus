cmake_minimum_required(VERSION 3.15 FATAL_ERROR)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED TRUE)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)

project(TreeSitterPlusPlus VERSION 0.1.0)

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)
set(CMAKE_INSTALL_CONFIGDIR "${CMAKE_INSTALL_LIBDIR}/cmake/tree-sitter")

option(ENABLE_TESTS "Enable unit tests" OFF)

function(CHECKOUT proj tag)
    if (NOT EXISTS "${CMAKE_CURRENT_BINARY_DIR}/${proj}")
        message(STATUS "Cloning ${proj} ${tag}")
        execute_process(
            COMMAND git clone -b ${tag} https://github.com/tree-sitter/${proj}
            WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
            RESULT_VARIABLE ret
            OUTPUT_QUIET
            ERROR_QUIET
        )
        if (NOT ret EQUAL 0)
            message(ERROR "Error cloning ${proj}")
        endif()
    endif()
endfunction()

checkout(tree-sitter v0.25.4)
checkout(tree-sitter-c v0.24.1)
checkout(tree-sitter-c-sharp v0.23.1)
checkout(tree-sitter-cpp v0.23.4)
checkout(tree-sitter-go v0.23.4)
checkout(tree-sitter-java v0.23.5)
checkout(tree-sitter-javascript v0.23.1)
checkout(tree-sitter-python v0.23.6)
checkout(tree-sitter-rust v0.24.0)
checkout(tree-sitter-typescript v0.23.2)

# Patch the compatibility issue of tree-sitter-typescript
execute_process(
  COMMAND sed -i.bak
          "s:tree_sitter/parser:parser:"
          "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-typescript/common/scanner.h"
)

add_library(Tree-Sitter
    src/cursor.cpp
    src/lang.cpp
    src/node.cpp
    src/parser.cpp
    src/query.cpp
    src/tree.cpp
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter/lib/src/lib.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-c/src/parser.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-c-sharp/src/parser.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-c-sharp/src/scanner.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-cpp/src/parser.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-cpp/src/scanner.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-go/src/parser.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-java/src/parser.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-javascript/src/parser.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-javascript/src/scanner.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-python/src/parser.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-python/src/scanner.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-rust/src/parser.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-rust/src/scanner.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-typescript/typescript/src/parser.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-typescript/typescript/src/scanner.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-typescript/tsx/src/parser.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-typescript/tsx/src/scanner.c"
)

target_include_directories(Tree-Sitter
    PRIVATE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/tree-sitter/lib/src>
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/tree-sitter/lib/include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
)

set_target_properties(Tree-Sitter PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
)

if(ENABLE_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()

configure_package_config_file(
    ${CMAKE_CURRENT_LIST_DIR}/tree-sitter-config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-config.cmake
    INSTALL_DESTINATION
        ${CMAKE_INSTALL_CONFIGDIR}
)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-config-version.cmake"
    COMPATIBILITY
        SameMajorVersion
)

install(DIRECTORY
        ${CMAKE_CURRENT_BINARY_DIR}/tree-sitter/lib/include/
        ${CMAKE_CURRENT_SOURCE_DIR}/include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    FILES_MATCHING
    PATTERN "*.h"
)

install(TARGETS Tree-Sitter
    EXPORT Tree-Sitter
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(FILES
        "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-config.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-config-version.cmake"
    DESTINATION
        ${CMAKE_INSTALL_CONFIGDIR}
)

install(EXPORT Tree-Sitter
    DESTINATION ${CMAKE_INSTALL_CONFIGDIR}
    NAMESPACE Tree-Sitter::
    FILE tree-sitter-targets.cmake
)
