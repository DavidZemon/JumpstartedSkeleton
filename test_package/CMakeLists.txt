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

project(JumpstartedSkeletonTestConan)

find_package(JumpstartedSkeleton REQUIRED)

set(CMAKE_CXX_STANDARD 11)
add_executable(${PROJECT_NAME} ${PROJECT_NAME})
target_link_libraries(${PROJECT_NAME} PUBLIC JumpstartedSkeleton::jumpstartedskeleton-lib)

enable_testing()
add_test(NAME ${PROJECT_NAME}
        COMMAND ${CMAKE_CROSSCOMPILING_EMULATOR} "$<TARGET_FILE:${PROJECT_NAME}>")
