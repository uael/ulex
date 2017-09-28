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

set(CMAKE_C_STANDARD 99)
set(BUILD_SHARED_LIBS OFF)

project(ulex C)
project_dependency(ufs GIT_REPOSITORY https://github.com/uael/ufs.git)
project_dependency(uev GIT_REPOSITORY https://github.com/uael/uev.git)

set(ulex_INC_DIR ${CMAKE_CURRENT_LIST_DIR}/include)
set(ulex_SRC_DIR ${CMAKE_CURRENT_LIST_DIR}/src)
set(ulex_TEST_DIR ${CMAKE_CURRENT_LIST_DIR}/test)

file(GLOB_RECURSE ulex_SRCS ${ulex_SRCS} ${ulex_SRC_DIR}/*.c)
file(GLOB_RECURSE ulex_HDRS ${ulex_HDRS} ${ulex_INC_DIR}/ulex/*.h)
set(ulex_HDR ${ulex_INC_DIR}/ulex.h)

add_library(ulex ${ulex_SRCS} ${ulex_HDRS} ${ulex_HDR})
target_include_directories(ulex PUBLIC ${ulex_INC_DIR})
target_link_libraries(ulex PUBLIC ufs uev)

if (MSVC)
  target_compile_options(ulex
    PRIVATE /MT$<$<CONFIG:Debug>:d> /Oy
    PRIVATE /O$<$<CONFIG:Debug>:d>$<$<CONFIG:Release>:x>)
  set_target_properties(ulex
    PROPERTIES LINK_FLAGS /nodefaultlib:msvcrt.lib)
else ()
  target_compile_options(ulex
    PRIVATE -Wall -Werror -Wextra -fomit-frame-pointer
    PRIVATE -Wno-missing-field-initializers
    PRIVATE -O$<$<CONFIG:Debug>:0 -g3>$<$<CONFIG:Release>:3>)
endif ()

get_directory_property(ulex_PARENT PARENT_DIRECTORY)
if (NOT ${ulex_PARENT})
  set(ulex_DEVEL TRUE)
elseif (NOT ${ulex_DEVEL})
  set(ulex_DEVEL FALSE)
endif ()
if (${ulex_DEVEL} AND EXISTS ${ulex_TEST_DIR})
  project_dependency(cute GIT_REPOSITORY https://github.com/uael/cute.git)
  if (NOT ulex_PARENT)
    enable_testing()
  endif ()

  file(GLOB ulex_TEST_SRCS ${ulex_TEST_SRCS} ${ulex_TEST_DIR}/*.c)
  foreach (ulex_TEST_SRC ${ulex_TEST_SRCS})
    get_filename_component(ulex_TEST_NAME ${ulex_TEST_SRC} NAME_WE)
    add_executable(test_${ulex_TEST_NAME} ${ulex_TEST_SRC})
    add_dependencies(test_${ulex_TEST_NAME} ulex cute)
    target_link_libraries(test_${ulex_TEST_NAME} ulex cute)
    add_test(${ulex_TEST_NAME} test_${ulex_TEST_NAME})
  endforeach ()
endif ()

install(FILES ${ulex_HDRS}
  DESTINATION include/ulex)
install(FILES ${ulex_HDR}
  DESTINATION include)