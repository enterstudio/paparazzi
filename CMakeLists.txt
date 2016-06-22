cmake_minimum_required(VERSION 2.8)
project(tangram)

# helper functions
include(${PROJECT_SOURCE_DIR}/tangram-es/toolchains/utils.cmake)

# cmake output configuration
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# compiler options
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -fpermissive -g -std=c++1y")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-omit-frame-pointer")
set(CXX_FLAGS_DEBUG "-g -O0")
set(EXECUTABLE_NAME "tangramPaparazzi")

if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-gnu-zero-variadic-macro-arguments")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS}  -lc++ -lc++abi")
endif()

# compile definitions (adds -DPLATFORM_LINUX)
add_definitions(-DPLATFORM_LINUX)
set(PLATFORM_LINUX ON)
set(CORE_COMPILE_DEFS PLATFORM_LINUX)


if (CMAKE_COMPILER_IS_GNUCC)
  execute_process(COMMAND ${CMAKE_CXX_COMPILER} -dumpversion OUTPUT_VARIABLE GCC_VERSION)
  string(REGEX MATCHALL "[0-9]+" GCC_VERSION_COMPONENTS ${GCC_VERSION})
  list(GET GCC_VERSION_COMPONENTS 0 GCC_MAJOR)
  list(GET GCC_VERSION_COMPONENTS 1 GCC_MINOR)
  message(STATUS "Using gcc ${GCC_VERSION}")
  if (GCC_VERSION VERSION_GREATER 5.1)
    message(STATUS "USE CXX11_ABI")
    add_definitions("-D_GLIBCXX_USE_CXX11_ABI=1")
  endif()
endif()

# add sources and include headers
find_sources_and_include_directories(
  ${PROJECT_SOURCE_DIR}/src/*.h
  ${PROJECT_SOURCE_DIR}/src/*.cpp)

# add sources and include headers
find_sources_and_include_directories(
  ${PROJECT_SOURCE_DIR}/tangram-es/linux/src/urlWorker.*
  ${PROJECT_SOURCE_DIR}/tangram-es/linux/src/urlWorker.*)

# load core library
add_subdirectory(${PROJECT_SOURCE_DIR}/tangram-es/external)
add_subdirectory(${PROJECT_SOURCE_DIR}/tangram-es/core)
include_directories(${CORE_INCLUDE_DIRS})
include_directories(${CORE_LIBRARIES_INCLUDE_DIRS})

# add executable
add_executable(${EXECUTABLE_NAME} ${SOURCES})

# copy resources
add_resources(${EXECUTABLE_NAME} "${PROJECT_SOURCE_DIR}/tangram-es/scenes")
find_package(OpenGL REQUIRED)

# link libraries
target_link_libraries(${EXECUTABLE_NAME} ${CORE_LIBRARY})
target_link_libraries(${EXECUTABLE_NAME} ${CORE_LIBRARY} -lcurl glfw -ldl ${GLFW_LIBRARIES} ${OPENGL_LIBRARIES})
