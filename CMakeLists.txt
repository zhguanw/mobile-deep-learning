cmake_minimum_required(VERSION 2.8.7)
project(mobile-deep-learning)

if(NOT MSVC)
include_directories(include /usr/local/include)
endif(NOT MSVC)

set(CMAKE_VERBOSE_MAKEFILE on)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY build)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY build)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY build)
set(MDL_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})

if(MSVC)
    # Add path for custom modules such as pthread library and protoc library
    set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${MDL_SOURCE_DIR}/cmake")
    find_package(PTHREAD REQUIRED)
    find_package(PROTOC REQUIRED)

    if(PTHREAD_FOUND)
        include_directories(${PTHREAD_INCLUDE_DIR})
    endif(PTHREAD_FOUND)

    if(PROTOC_FOUND)
        include_directories(${PROTOC_INCLUDE_DIR})
    endif(PROTOC_FOUND)
endif(MSVC)

if(IS_IOS)
    message(status " Platform is IOS ")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fobjc-abi-version=2 -fobjc-arc -isysroot ${CMAKE_OSX_SYSROOT}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fobjc-abi-version=2 -fobjc-arc -std=gnu++11 -stdlib=libc++ -isysroot ${CMAKE_OSX_SYSROOT}")
endif(IS_IOS)

if(UNIX AND NOT APPLE)
    message(status " Platform is Unix or Linux ")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -pthread")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread -std=gnu++11")
    #set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fobjc-abi-version=2 -fobjc-arc")
    #set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fobjc-abi-version=2 -fobjc-arc -std=gnu++11 -stdlib=libstdc++")
endif(UNIX AND NOT APPLE)

# Add all headers and source files.
file(GLOB_RECURSE MDL_CPP src/*.cpp third-party/*.cpp)
file(GLOB_RECURSE MDL_H *.h)
list(REMOVE_ITEM MDL_CPP main.cpp)
if(ANDROID)
#else(ANDROID)
    list(REMOVE_ITEM MDL_H ${CMAKE_CURRENT_SOURCE_DIR}/include/mdl_jni.h)
    list(REMOVE_ITEM MDL_CPP ${CMAKE_CURRENT_SOURCE_DIR}/src/mdl_jni.cpp)
endif(ANDROID)

# include headers
include_directories(include/)
include_directories(third-party/)

# gen so
ADD_LIBRARY(mdl SHARED ${MDL_CPP} ${MDL_H})

# gen static
ADD_LIBRARY(mdl-static STATIC ${MDL_CPP} ${MDL_H})

# gen test
if(UNIX AND NOT APPLE)
    ADD_EXECUTABLE(mdlTest test/main.cpp)
    target_link_libraries(mdlTest mdl-static)
endif(UNIX AND NOT APPLE)

if(IS_IOS)
    ADD_EXECUTABLE(mdlTest test/main.cpp)
    target_link_libraries(mdlTest mdl-static)
endif(IS_IOS)

if((UNIX OR MSCV ) AND NOT APPLE)
    add_subdirectory(tools)
    add_definitions(-DMDL_MAC="false")
elseif(IS_MAC)
    add_subdirectory(tools)
    add_definitions(-DMDL_MAC="true")
elseif(V7)
    add_definitions(-DMDL_V7="true")
elseif(V8)
    add_definitions(-DMDL_V8="true")
endif(V8)



