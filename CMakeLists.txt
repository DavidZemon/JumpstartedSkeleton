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

option(BUILD_SHARED_LIBS "Build dynamic libraries when on, else static" ON)

add_compile_options(-Werror -Wall -Wpedantic -Wconversion)
set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--no-undefined")

add_subdirectory(src)

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
