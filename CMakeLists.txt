cmake_minimum_required(VERSION 3.1 FATAL_ERROR)
cmake_policy(SET CMP0054 OLD)
cmake_policy(SET CMP0045 OLD)

include(CMakeParseArguments)
if (NOT COMMAND project_dependency)
  function(project_dependency DEP)
    cmake_parse_arguments(DL_ARGS "" "" "" ${ARGN})
    set(DL_ARGS_BUILD_DIR "${CMAKE_BINARY_DIR}/vendor/${DEP}")
    set(DL_ARGS_SOURCE_DIR "${CMAKE_SOURCE_DIR}/vendor/${DEP}")
    file(WRITE "${DL_ARGS_BUILD_DIR}/CMakeLists.txt"
      "cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)\n"
      "project(vendor-${DEP} NONE)\n"
      "include(ExternalProject)\n"
      "ExternalProject_Add(vendor-${DEP}\n"
      "${DL_ARGS_UNPARSED_ARGUMENTS}\nSOURCE_DIR \"${DL_ARGS_SOURCE_DIR}\"\n"
      "CONFIGURE_COMMAND \"\"\nBUILD_COMMAND \"\"\nINSTALL_COMMAND \"\"\nTEST_COMMAND \"\"\n"
      "UPDATE_COMMAND \"\"\nPATCH_COMMAND \"\")")
    execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}"
      -D "CMAKE_MAKE_PROGRAM:FILE=${CMAKE_MAKE_PROGRAM}" .
      RESULT_VARIABLE result OUTPUT_QUIET
      WORKING_DIRECTORY "${DL_ARGS_BUILD_DIR}")
    if (result)
      message(FATAL_ERROR "Config step for ${DEP} failed: ${result}")
    endif ()
    execute_process(COMMAND ${CMAKE_COMMAND} --build .
      RESULT_VARIABLE result OUTPUT_QUIET
      WORKING_DIRECTORY "${DL_ARGS_BUILD_DIR}")
    if (result)
      message(FATAL_ERROR "Build step for ${DEP} failed: ${result}")
    endif ()
    add_subdirectory(${DL_ARGS_SOURCE_DIR} ${DL_ARGS_BUILD_DIR}/build)
  endfunction()
endif ()

project(uty C)
project_dependency(ucc GIT_REPOSITORY https://github.com/uael/ucc.git)

set(uty_INC_DIR ${CMAKE_CURRENT_LIST_DIR}/include)
set(uty_TEST_DIR ${CMAKE_CURRENT_LIST_DIR}/test)

file(GLOB_RECURSE uty_HDRS ${uty_HDRS} ${uty_INC_DIR}/uty/*.h)
set(uty_HDR ${uty_INC_DIR}/uty.h)

add_library(uty INTERFACE)
target_include_directories(uty INTERFACE ${uty_INC_DIR})
target_link_libraries(uty INTERFACE ucc)

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
