cmake_minimum_required(VERSION 2.8)

project(pthread)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release")
    message(STATUS "Build type not specified: Use Release by default.")
endif()

add_definitions(
   -DHAVE_CONFIG_H
   -DNDEBUG 
   -DUNICODE
   -D_UNICODE
)

if(MSVC)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MP /GS /GL /Wall /wd\"4820\" /wd\"4668\" /wd\"4255\" /Oi /Gy /Zc:wchar_t /Zi /Gm-")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MP /GS /GL /Wall /wd\"4820\" /wd\"4668\" /wd\"4255\" /Oi /Gy /Zc:wchar_t /Zi /Gm-")
endif()
if(NOT MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    if (${CMAKE_CXX_COMPILER_ID} STREQUAL "Clang")
        set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
    endif()
endif()

option(PTHREAD_BUILD_SHARED "Build pthread as a shared library." ON)

if (PTHREAD_BUILD_SHARED)
    message(STATUS "Build will generate a shared library. "
            "Use PTHREAD_BUILD_SHARED=0 to perform a static build")
    set(PTHREAD_LIBRARY_TYPE SHARED)
else()
    message(STATUS "Build will generate a static library.")
    set(PTHREAD_LIBRARY_TYPE STATIC)
    add_definitions(-DPTW32_STATIC_LIB)
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/src SOURCES)
list(REMOVE_ITEM SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/src/pthread.c)

add_library(${PROJECT_NAME} ${PTHREAD_LIBRARY_TYPE} ${SOURCES})
if(MSVC)
    target_link_libraries(${PROJECT_NAME} ws2_32)
endif()

file(GLOB HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)
install(
    FILES ${HEADERS} DESTINATION include/${PROJECT_NAME}
)

install(
    TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)
