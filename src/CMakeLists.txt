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


add_library(jumpstartedskeleton-lib JumpstartedSkeleton.cpp)
target_include_directories(jumpstartedskeleton-lib PUBLIC
    "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>"
    $<INSTALL_INTERFACE:include>
)

set_target_properties(jumpstartedskeleton-lib PROPERTIES OUTPUT_NAME jumpstartedskeleton)
install(TARGETS jumpstartedskeleton-lib EXPORT ${PROJECT_NAME}Config DESTINATION lib)

add_library(jumpstartedskeleton-cli-lib STATIC JumpstartedSkeleton-cli.cpp)
target_include_directories(jumpstartedskeleton-cli-lib INTERFACE "${CMAKE_CURRENT_LIST_DIR}")
target_link_libraries(jumpstartedskeleton-cli-lib jumpstartedskeleton-lib)

add_executable(jumpstartedskeleton main.cpp)
target_link_libraries(jumpstartedskeleton jumpstartedskeleton-cli-lib)
install(TARGETS jumpstartedskeleton DESTINATION bin)
