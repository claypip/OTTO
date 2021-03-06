# Version 3.8 is required for c++17
cmake_minimum_required(VERSION 3.8)
project(OTTO VERSION 0.0.1 LANGUAGES CXX C)

set(CMAKE_CXX_EXTENSIONS OFF)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_BUILD_TYPE Debug)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

option(OTTO_DEBUG_UI "Enable the imgui based debug ui" ON)
option(OTTO_BUILD_DOCS "Build documentation" OFF)
option(OTTO_USE_LIBCXX "Link towards libc++ instead of libstdc++. This is the default on OSX" ${APPLE})
option(OTTO_ENABLE_TIMERS "Enable debugging timers" OFF)
option(OTTO_RPI "Build for Raspberry Pi (native graphics)" OFF)

if (OTTO_USE_LIBCXX)
  message("Using libc++ instead of libstdc++")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
endif()

if (OTTO_ENABLE_TIMERS)
  add_definitions(-DOTTO_ENABLE_TIMERS=1)
else ()
  add_definitions(-DOTTO_ENABLE_TIMERS=0)
endif ()

if (OTTO_RPI)
  add_definitions(-DOTTO_UI_EGL=1)
  set(OTTO_DEBUG_UI OFF)
else()
  add_definitions(-DOTTO_UI_GLFW=1)
endif()

if (OTTO_DEBUG_UI)
  add_definitions(-DOTTO_DEBUG_UI=1)
else()
  add_definitions(-DOTTO_DEBUG_UI=0)
endif()

add_subdirectory(external)

# Enable warnings for local code
if(MSVC)
  if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
    string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
  endif()
elseif(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-long-long -Wpedantic -Wno-c++14-compat -Wno-psabi")
elseif(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-long-long -Wpedantic \
-Wno-c++11-compat -Wno-c++98-compat -Wno-c++14-compat -Wc++17-compat \
-Wno-shadow-field-in-constructor \
-Wno-documentation-unknown-command \
")
endif()

add_subdirectory(src)
add_subdirectory(test)
