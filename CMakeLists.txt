cmake_minimum_required(VERSION 3.1 FATAL_ERROR)
cmake_policy(SET CMP0054 OLD)
cmake_policy(SET CMP0045 OLD)

if (NOT EXISTS ${CMAKE_CACHEFILE_DIR}/cmake)
  execute_process(
    COMMAND git clone https://github.com/uael/cmake-modules.git cmake
    RESULT_VARIABLE result OUTPUT_QUIET
    WORKING_DIRECTORY ${CMAKE_CACHEFILE_DIR}
  )
endif ()

set(CMAKE_MODULE_PATH "${CMAKE_CACHEFILE_DIR}/cmake")
include(ProjectDependency)

project(uty C)
project_dependency(ucc GIT_REPOSITORY https://github.com/uael/ucc.git)

set(uty_INC_DIR ${CMAKE_CURRENT_LIST_DIR}/include)
set(uty_TEST_DIR ${CMAKE_CURRENT_LIST_DIR}/test)

file(GLOB_RECURSE uty_HDRS ${uty_HDRS} ${uty_INC_DIR}/uty/*.h)
set(uty_HDR ${uty_INC_DIR}/uty.h)

add_library(uty INTERFACE)
target_include_directories(uty INTERFACE ${uty_INC_DIR})
target_link_libraries(uty INTERFACE ucc)

set(CMAKE_C_FLAGS "")
set(CMAKE_C_FLAGS_DEBUG "")
set(CMAKE_C_FLAGS_RELEASE "")

if (MSVC)
  target_compile_options(uty
    INTERFACE /MT$<$<CONFIG:Debug>:d> /Oy /Za
    INTERFACE /O$<$<CONFIG:Debug>:d>$<$<CONFIG:Release>:x>)
else ()
  target_compile_options(uty
    INTERFACE -Wall -Werror -Wextra -fomit-frame-pointer
    INTERFACE -Wno-missing-field-initializers
    INTERFACE -O$<$<CONFIG:Debug>:0 -g3>$<$<CONFIG:Release>:3>)
endif ()

get_directory_property(uty_PARENT PARENT_DIRECTORY)
if (NOT ${uty_PARENT})
  set(uty_DEVEL TRUE)
elseif (NOT ${uty_DEVEL})
  set(uty_DEVEL FALSE)
endif ()
if (${uty_DEVEL} AND EXISTS ${uty_TEST_DIR})
  if (NOT uty_PARENT)
    enable_testing()
  endif ()

  file(GLOB uty_TEST_SRCS ${uty_TEST_SRCS} ${uty_TEST_DIR}/*.c)
  foreach (uty_TEST_SRC ${uty_TEST_SRCS})
    get_filename_component(uty_TEST_NAME ${uty_TEST_SRC} NAME_WE)
    add_executable(test_${uty_TEST_NAME} ${uty_TEST_SRC})
    add_dependencies(test_${uty_TEST_NAME} uty)
    target_link_libraries(test_${uty_TEST_NAME} uty)
    add_test(${uty_TEST_NAME} test_${uty_TEST_NAME})
  endforeach ()
endif ()

install(FILES ${uty_HDRS}
  DESTINATION include/uty)
install(FILES ${uty_HDR}
  DESTINATION include)
