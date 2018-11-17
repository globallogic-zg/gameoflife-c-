set (PROJECT_NAME "Coderetreat 2018")

message (STATUS "### ${PROJECT_NAME} C++14 building system ###")

if (${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
  message (FATAL_ERROR "Create out-of-source tree first! From some other directory call, eg.:")
  message (FATAL_ERROR "    cmake -G \"Unix Makefiles\" ../source_code/")
endif()

# needed definitions
set (CMAKE_VER    "3.0.2")
cmake_minimum_required(VERSION ${CMAKE_VER})
project(${PROJECT_NAME})

# for ExternalProject_* functions
include (${CMAKE_ROOT}/Modules/ExternalProject.cmake)

# git & googletest
find_package(Git)

if (GIT_FOUND)
  set (GTEST_EXT "googletest")
  set (GTEST_GIT "https://github.com/google/googletest.git")

  # force shared crt
  ExternalProject_Add(${GTEST_EXT} GIT_REPOSITORY ${GTEST_GIT}
                          CMAKE_ARGS "-Dgtest_force_shared_crt=ON"
                          INSTALL_COMMAND ""
                          LOG_DOWNLOAD ON
                          LOG_CONFIGURE ON
                          LOG_BUILD ON)

  # Specify include dir
  ExternalProject_Get_Property(${GTEST_EXT} source_dir)
  set(GTEST_INCLUDE_PATH ${source_dir}/googletest/include)

  # Library
  ExternalProject_Get_Property(${GTEST_EXT} binary_dir)
  set(GTEST_LIBRARY_PATH ${binary_dir}/lib)
  set(GTEST_LIBRARY gtest)
  # set test libs names
  if (WIN32)
    set (TEST_LIBS "gtestd.lib" "gtest_maind.lib")
  else (WIN32)
    set (TEST_LIBS "libgtest.a" "libgtest_main.a" "pthread")
  endif (WIN32)

  # set includes & libraries
  include_directories (${GTEST_INCLUDE_PATH})
  link_directories (${GTEST_LIBRARY_PATH})
  enable_testing()

  set (CTEST_OUTPUT_ON_FAILURE ON)

  add_executable(runUnitTests ${CMAKE_SOURCE_DIR}/main-test.cpp)

  target_link_libraries(runUnitTests ${TEST_LIBS})

else (GIT_FOUND)
  message (FATAL_ERROR "Git package not found. Get one from https://git-scm.com/downloads")
endif (GIT_FOUND)