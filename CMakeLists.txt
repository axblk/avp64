##############################################################################
#                                                                            #
# Copyright 2020 Lukas Jünger                                                #
#                                                                            #
# This software is licensed under the MIT license.                           #
# A copy of the license can be found in the LICENSE file at the root         #
# of the source tree.                                                        #
#                                                                            #
#############################################################################

cmake_minimum_required(VERSION 3.6)
project(avp64)
option(BUILD_TESTS "Build unit tests" OFF)

include(ExternalProject)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror")
set(CMAKE_EXE_LINKER_FLAGS -rdynamic)

set(OCX_QEMU_ARM_HOME $ENV{OCX_QEMU_HOME})
if(NOT EXISTS ${OCX_QEMU_ARM_HOME})
    set(OCX_QEMU_ARM_HOME ${CMAKE_CURRENT_SOURCE_DIR}/deps/ocx-qemu-arm)
endif()

set(VCML_HOME $ENV{VCML_HOME})
if(NOT EXISTS ${VCML_HOME})
    set(VCML_HOME ${CMAKE_CURRENT_SOURCE_DIR}/deps/vcml)
endif()

message(STATUS "Found OCX at " ${OCX_HOME})
message(STATUS "Found OCX-QEMU-ARM at " ${OCX_QEMU_ARM_HOME})
message(STATUS "Found VCML at " ${VCML_HOME})

add_subdirectory(${OCX_QEMU_ARM_HOME} ocx-qemu-arm)
add_subdirectory(${VCML_HOME} vcml EXCLUDE_FROM_ALL)

set(src "${CMAKE_CURRENT_SOURCE_DIR}/src")
set(inc "${CMAKE_CURRENT_SOURCE_DIR}/include"
        "${CMAKE_CURRENT_SOURCE_DIR}/deps/ocx-qemu-arm/ocx/include")

set(sources
    ${src}/arm64_cpu.cpp
    ${src}/system.cpp
    ${src}/main.cpp)

add_executable(avp64 ${sources})

target_include_directories(avp64 PRIVATE ${inc})

target_link_libraries(avp64 vcml)
target_link_libraries(avp64 ${CMAKE_DL_LIBS})
target_link_libraries(avp64 -pthread -lelf)

install(TARGETS avp64 DESTINATION bin)
install(DIRECTORY config/ DESTINATION config)

if(BUILD_TESTS)
    message("Building AVP64 tests")
    enable_testing()
    add_subdirectory(tests)
endif()