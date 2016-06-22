cmake_minimum_required(VERSION 2.8)
project(tangram)

# helper functions
include(${PROJECT_SOURCE_DIR}/tangram-es/toolchains/utils.cmake)

# cmake output configuration
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# compiler options
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -fpermissive -g")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_CXX_FLAGS} -L/opt/vc/lib/ -lGLESv2 -lEGL -lbcm_host -lvchiq_arm -lvcos -lrt -lpthread")
set(CXX_FLAGS_DEBUG "-g -O0")
set(EXECUTABLE_NAME "tangramPaparazzi")

add_definitions(-DPLATFORM_RPI)

# check for c++11 compiler
execute_process(COMMAND ${CMAKE_CXX_COMPILER} -dumpversion OUTPUT_VARIABLE GCC_VERSION)

if("${CMAKE_CXX_COMPILER_ID}" MATCHES "GNU")
    if(NOT (GCC_VERSION VERSION_GREATER 4.8 OR GCC_VERSION VERSION_EQUAL 4.8))
        message(FATAL_ERROR "Please install g++ version 4.8 or greater")
    else()
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
    endif()
else()
    message(FATAL_ERROR "Please install a C++14 compatible compiler")
endif()

# add sources and include headers
find_sources_and_include_directories(
    ${PROJECT_SOURCE_DIR}/src/*.h 
    ${PROJECT_SOURCE_DIR}/src/*.cpp)

# add sources and include headers
find_sources_and_include_directories(
    ${PROJECT_SOURCE_DIR}/tangram-es/linux/src/urlWorker.*
    ${PROJECT_SOURCE_DIR}/tangram-es/linux/src/urlWorker.*)

# include headers for rpi-installed libraries
include_directories(/opt/vc/include/)
include_directories(/opt/vc/include/interface/vcos/pthreads)
include_directories(/opt/vc/include/interface/vmcs_host/linux)

# load core library
add_subdirectory(${PROJECT_SOURCE_DIR}/tangram-es/external)
add_subdirectory(${PROJECT_SOURCE_DIR}/tangram-es/core)
include_directories(${CORE_INCLUDE_DIRS})
include_directories(${CORE_LIBRARIES_INCLUDE_DIRS})

# add executable
add_executable(${EXECUTABLE_NAME} ${SOURCES})

# copy resources
add_resources(${EXECUTABLE_NAME} "${PROJECT_SOURCE_DIR}/tangram-es/scenes")

# link libraries
target_link_libraries(${EXECUTABLE_NAME} -lcurl)
target_link_libraries(${EXECUTABLE_NAME} ${CORE_LIBRARY})

## https://github.com/tangrams/tangram-es/blob/686a58043a7b8276add7521335b1ecef514712c7/toolchains/linux.cmake

# set for test in other cmake files
set(PLATFORM_LINUX ON)

# global compile options
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -std=c++1y")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-omit-frame-pointer")

if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-gnu-zero-variadic-macro-arguments")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS}  -lc++ -lc++abi")
endif()

if (CMAKE_COMPILER_IS_GNUCC)
  execute_process(COMMAND ${CMAKE_CXX_COMPILER} -dumpversion
    OUTPUT_VARIABLE GCC_VERSION)
  string(REGEX MATCHALL "[0-9]+" GCC_VERSION_COMPONENTS ${GCC_VERSION})
  list(GET GCC_VERSION_COMPONENTS 0 GCC_MAJOR)
  list(GET GCC_VERSION_COMPONENTS 1 GCC_MINOR)

  message(STATUS "Using gcc ${GCC_VERSION}")
  if (GCC_VERSION VERSION_GREATER 5.1)
    message(STATUS "USE CXX11_ABI")
    add_definitions("-D_GLIBCXX_USE_CXX11_ABI=1")
  endif()
endif()

check_unsupported_compiler_version()

# compile definitions (adds -DPLATFORM_LINUX)
set(CORE_COMPILE_DEFS PLATFORM_LINUX)

# if (USE_EXTERNAL_LIBS)
#   include(${EXTERNAL_LIBS_DIR}/core-dependencies.cmake)
#   include(${EXTERNAL_LIBS_DIR}/glfw.cmake)
# else()
 #  add_subdirectory(${PROJECT_SOURCE_DIR}/external)
 #endif()

# load core library
# add_subdirectory(${PROJECT_SOURCE_DIR}/core)

if(APPLICATION)

  set(EXECUTABLE_NAME "tangram")

  find_package(OpenGL REQUIRED)

  # add sources and include headers
  find_sources_and_include_directories(
    ${PROJECT_SOURCE_DIR}/linux/src/*.h
    ${PROJECT_SOURCE_DIR}/linux/src/*.cpp)

  add_executable(${EXECUTABLE_NAME} ${SOURCES})

  target_link_libraries(${EXECUTABLE_NAME}
    ${CORE_LIBRARY}
    -lcurl glfw
    # only used when not using external lib
    -ldl
    ${GLFW_LIBRARIES}
    ${OPENGL_LIBRARIES})

  add_resources(${EXECUTABLE_NAME} "${PROJECT_SOURCE_DIR}/scenes")

endif()
