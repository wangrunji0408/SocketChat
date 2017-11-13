cmake_minimum_required(VERSION 3.8)
project(Server)

set(CMAKE_CXX_STANDARD 11)

find_package(Protobuf REQUIRED)
include_directories(${Protobuf_INCLUDE_DIRS})
include_directories(${CMAKE_CURRENT_BINARY_DIR})
EXECUTE_PROCESS(
        COMMAND ${PROTOBUF_PROTOC_EXECUTABLE}
        --cpp_out=. protobuf/chat.proto
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src)

include_directories(src)
file(GLOB SOURCES
        src/protobuf/*
        ${PROTO_SRCS} ${PROTO_HDRS}
        src/*.h
        src/*.cpp)

add_executable(Server ${SOURCES})
target_link_libraries(Server ${Protobuf_LIBRARIES})

list(REMOVE_ITEM SOURCES src/main.cpp)
add_library(ServerLib SHARED ${SOURCES})


# Download and unpack googletest at configure time
configure_file(CMakeLists.txt.in googletest-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
        RESULT_VARIABLE result
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/googletest-download )
if(result)
    message(FATAL_ERROR "CMake step for googletest failed: ${result}")
endif()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
        RESULT_VARIABLE result
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/googletest-download )
if(result)
    message(FATAL_ERROR "Build step for googletest failed: ${result}")
endif()

# Prevent overriding the parent project's compiler/linker
# settings on Windows
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

# Add googletest directly to our build. This defines
# the gtest and gtest_main targets.
add_subdirectory(${CMAKE_BINARY_DIR}/googletest-src
        ${CMAKE_BINARY_DIR}/googletest-build)

# The gtest/gtest_main targets carry header search path
# dependencies automatically when using CMake 2.8.11 or
# later. Otherwise we have to add them here ourselves.
if (CMAKE_VERSION VERSION_LESS 2.8.11)
    include_directories("${gtest_SOURCE_DIR}/include")
endif()

file(GLOB TEST_SOURCES test/*.cpp test/*.h test/**/*.cpp)

# Now simply link against gtest or gtest_main as needed. Eg
add_executable(Test ${TEST_SOURCES})
target_link_libraries(Test ServerLib)
target_link_libraries(Test gtest gtest_main)
#target_link_libraries(Test gmock gmock_main)
#add_test(NAME example_test COMMAND example)