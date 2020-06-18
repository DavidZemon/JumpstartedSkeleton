# Zero-Clause BSD
#
# Permission to use, copy, modify, and/or distribute this software for any purpose with or without fee is hereby
# granted.
#
# THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES WITH REGARD TO THIS SOFTWARE INCLUDING ALL
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY SPECIAL, DIRECT,
# INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN
# AN ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR
# PERFORMANCE OF THIS SOFTWARE.

cmake_minimum_required(VERSION 3.9)

file(STRINGS "version.txt" VERSION)
project(JumpstartedSkeleton VERSION ${VERSION})

# Configure SonarQube properties file
if (DEFINED ENV{CI_GTEST_OUTPUT_DIR})
    set(GTEST_OUTPUT_DIR "${CMAKE_BINARY_DIR}/$ENV{CI_GTEST_OUTPUT_DIR}")
else ()
    set(GTEST_OUTPUT_DIR "${CMAKE_BINARY_DIR}/gtest_output")
endif ()
# Don't use PROJECT_SOURCE_DIR in the properties file because paths get changed up when in Docker
file(RELATIVE_PATH RELATIVE_SOURCE_ROOT "${CMAKE_CURRENT_BINARY_DIR}" "${CMAKE_CURRENT_SOURCE_DIR}")
configure_file(sonar-project.properties.in sonar-project.properties @ONLY)

enable_testing()

option(BUILD_SHARED_LIBS "Build dynamic libraries when on, else static" ON)
option(TEST_WITH_VALGRIND "Run unit tests with valgrind (if CMAKE_CROSSCOMPILING_EMULATOR is defined, it will override this setting)" OFF)
option(HTML_COVERAGE_REPORT "When enabled, test coverage will be reported in HTML format for human consumption. When disable, coverage will be reported in XML for use with SonarQube." ON)

add_compile_options(-Werror -Wall -Wpedantic -Wconversion)
set(CMAKE_C_FLAGS_DEBUG   "${CMAKE_C_FLAGS_DEBUG} --coverage")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} --coverage")
set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--no-undefined")

find_program(CPPCHECK_EXE cppcheck)
if (CPPCHECK_EXE)
    message("-- Program cppcheck found: ${CPPCHECK_EXE}")
    set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
    add_custom_target(cppcheck
        COMMAND "${CPPCHECK_EXE}" --std=c++${CMAKE_CXX_STANDARD} --enable=all --quiet "--project=${PROJECT_BINARY_DIR}/compile_commands.json")
endif ()

find_program(CLANG_TIDY_EXE clang-tidy)
if (CLANG_TIDY_EXE)
    message("-- Program clang-tidy found: ${CLANG_TIDY_EXE}")
    set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
    set(CMAKE_C_CLANG_TIDY "${CLANG_TIDY_EXE}" --quiet "-header-filter=${PROJECT_SOURCE_DIR}/(include/Zemon|src|tests)/.*" -p "${PROJECT_BINARY_DIR}/compile_commands.json")
    set(CMAKE_CXX_CLANG_TIDY "${CMAKE_C_CLANG_TIDY}")
endif ()

add_subdirectory(src)
add_subdirectory(tests)

install(PROGRAMS scripts/JumpstartedSkeleton.init
        RENAME JumpstartedSkeleton
        DESTINATION /etc/rc.d/init.d
        COMPONENT system)

install(DIRECTORY "include/"
    DESTINATION include
    COMPONENT dev)
install(EXPORT ${PROJECT_NAME}Config
    DESTINATION lib/cmake/${PROJECT_NAME}
    NAMESPACE JumpstartedSkeleton::
    COMPONENT dev)

include(JumpstartedSkeletonCPack.cmake)
