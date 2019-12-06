cmake_minimum_required(VERSION 3.5)
project(libmicrodns)

set(PROJECT_VERSION_MAJOR 0)
set(PROJECT_VERSION_MINOR 1)
set(PROJECT_VERSION_PATCH 0)
set(PROJECT_VERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH})

message(STATUS "operation system is ${CMAKE_SYSTEM}")

if (CMAKE_BUILD_TYPE MATCHES Debug)
    message("Debug compile")
    set(PROJECT_BINARY_DIR ${PROJECT_SOURCE_DIR}/debug)
else () #CMAKE_BUILD_TYPE MATCHES Release
    message("Release compile")
    set(PROJECT_BINARY_DIR ${PROJECT_SOURCE_DIR}/release)
endif ()
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR})
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR})

set(HAVE_INET_NTOP 0)
set(LIBS)
if (WIN32)
    set(HAVE_INET_NTOP 1)
    add_definitions(-DHAVE_STRUCT_POLLFD -D_SYS_POLL_H)
    set(LIBS ${LIBS} ws2_32 iphlpapi)
    set(HAVE_POLL 0)
    set(DEFINES "-D_UNICODE=1 -DUNICODE=1 -D_POSIX_C_SOURCE=200809L -D_BSD_SOURCE=1")
    if (MSVC)
        set(CMAKE_C_FLAGS "${DEFINES} -D_CRT_NONSTDC_NO_DEPRECATE -D_CRT_SECURE_NO_WARNINGS -D_CRT_SECURE_NO_DEPRECATE")
    elseif (MINGW)
        set(CMAKE_C_FLAGS "${DEFINES} -Wall -Wextra -Wsign-compare -Wstrict-aliasing -Wstrict-overflow -Wformat=2 -Wno-unused-parameter -pipe -Wcast-align -Wpointer-arith -Wmissing-prototypes -Wwrite-strings -Wlogical-op")
    endif ()
elseif (UNIX)
endif ()



add_definitions(-DHAVE_CONFIG_H)
configure_file(${PROJECT_SOURCE_DIR}/src/config.h.in ${CMAKE_CURRENT_BINARY_DIR}/src/config.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR}/src)

include_directories(${PROJECT_SOURCE_DIR}/include)
include_directories(${PROJECT_SOURCE_DIR}/compat)

add_library(compat STATIC compat/compat.c compat/inet.c compat/poll.c)

add_library(microdns STATIC src/mdns.c src/rr.c include/microdns/microdns.h include/microdns/rr.h)
target_link_libraries(microdns
        compat
        ${LIBS})

add_executable(listen examples/main.c)
target_link_libraries(listen microdns)

add_executable(announce examples/announce.c)
target_link_libraries(announce microdns)
