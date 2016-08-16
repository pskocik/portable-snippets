cmake_minimum_required(VERSION 2.8.6)

project(portable-snippet-tests)

enable_testing()

set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

add_library(munit SHARED munit/munit.c)
find_package(ClockGettime)
if(ClockGettime_FOUND)
	set_property(TARGET munit
		APPEND PROPERTY COMPILE_DEFINITIONS MUNIT_ALLOW_CLOCK_GETTIME)
  target_link_libraries(munit ${ClockGettime_LIBRARIES})
endif()
set_property (TARGET munit PROPERTY C_STANDARD "99")

function(psnip_add_tests)
  set(options)
  set(oneValueArgs TARGET)
  set(multiValueArgs SOURCES TESTS)
  cmake_parse_arguments(PSNIP_TEST "${options}" "${oneValueArgs}" "${multiValueArgs}" ${ARGN})

  add_executable("${PSNIP_TEST_TARGET}" ${PSNIP_TEST_SOURCES})
  target_link_libraries("${PSNIP_TEST_TARGET}" munit)

  if(PSNIP_TESTS)
    foreach(TEST ${PSNIP_TEST_TESTS})
      add_test(NAME "${TEST}"
	COMMAND $<TARGET_FILE:${TARGET}> "${TEST}")
    endforeach(TEST)
  else()
    add_test(NAME "/${PSNIP_TEST_TARGET}"
      COMMAND $<TARGET_FILE:${PSNIP_TEST_TARGET}>)
  endif()
endfunction(psnip_add_tests)

psnip_add_tests(TARGET endian
  SOURCES
    endian.c
  TESTS
    "/endian/swap"
    "/endian/from-le"
    "/endian/from-be")

psnip_add_tests(TARGET atomic
  SOURCES
    atomic.c
  TESTS
    "/atomic")