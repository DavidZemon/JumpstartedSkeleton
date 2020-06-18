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

find_package(GTest REQUIRED)
find_program(VALGRIND_EXE valgrind)

add_custom_target(test-all
    COMMAND ${CMAKE_CTEST_COMMAND} --output-on-failure
    WORKING_DIRECTORY "${PROJECT_BINARY_DIR}")

if (DEFINED ENV{CI_GTEST_OUTPUT_DIR})
    set(GTEST_OUTPUT_DIR "${CMAKE_BINARY_DIR}/$ENV{CI_GTEST_OUTPUT_DIR}")
else ()
    set(GTEST_OUTPUT_DIR "${CMAKE_BINARY_DIR}/gtest_output")
endif ()

function(create_test NAME LIB)
    set(XML_OUTPUT_ARG "--gtest_output=xml:${GTEST_OUTPUT_DIR}/${NAME}.xml")

    add_executable(${NAME} ${ARGN})
    target_link_libraries(${NAME} ${LIB} GTest::gmock_main)
    if (DEFINED CMAKE_CROSSCOMPILING_EMULATOR)
        add_test(NAME ${NAME}
            COMMAND ${CMAKE_CROSSCOMPILING_EMULATOR} "$<TARGET_FILE:${NAME}>" ${XML_OUTPUT_ARG})
    elseif (VALGRIND_EXE AND TEST_WITH_VALGRIND)
        # Valgrind isn't smart enough to create directories that don't exist
        file(MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/valgrind")
        add_test(NAME ${NAME}
            COMMAND "${VALGRIND_EXE}" --leak-check=full --leak-resolution=med  --track-origins=yes --vgdb=no
                --xml=yes "--xml-file=${CMAKE_BINARY_DIR}/valgrind/${NAME}-valgrind.xml" --error-exitcode=1
                "$<TARGET_FILE:${NAME}>" ${XML_OUTPUT_ARG})
    else ()
        add_test(NAME ${NAME}
            COMMAND "$<TARGET_FILE:${NAME}>" ${XML_OUTPUT_ARG})
    endif ()
    add_custom_target(test-${NAME}
        COMMAND ${CMAKE_CTEST_COMMAND} --output-on-failure -R "${NAME}"
        DEPENDS ${NAME})
    add_dependencies(test-all ${NAME})
endfunction()

create_test(JumpstartedSkeletonTest jumpstartedskeleton-lib JumpstartedSkeletonTest.cpp)
create_test(JumpstartedSkeleton-cliTest jumpstartedskeleton-cli-lib JumpstartedSkeleton-cliTest.cpp)
