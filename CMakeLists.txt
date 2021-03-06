cmake_minimum_required(VERSION 2.6)
project(nanodbc)

option(NANODBC_USE_UNICODE "build with unicode support enabled" OFF)
option(NANODBC_HANDLE_NODATA_BUG "enable special handling for SQL_NO_DATA (required for vertica)" OFF)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror")

if(APPLE)
	set(CMAKE_MACOSX_RPATH ON)
endif()

add_definitions(--std=c++11)

########################################
## find unixODBC or iODBC config binary
########################################
find_program(ODBC_CONFIG odbc_config $ENV{ODBC_PATH}/bin /usr/bin /usr/local/bin PATHS)

if(NOT ODBC_CONFIG)
	find_program(ODBC_CONFIG iodbc-config $ENV{ODBC_PATH}/bin /usr/bin /usr/local/bin PATHS)
endif()

if(NOT ODBC_CONFIG)
	message(FATAL_ERROR "can not find odbc config program")
endif()

if(NANODBC_USE_UNICODE)
	message(STATUS "Unicode support: Enabled")
	add_definitions(-DNANODBC_USE_UNICODE)
else()
	message(STATUS "Unicode support: Disabled")
endif()

if(NANODBC_HANDLE_NODATA_BUG)
	message(STATUS "Handle SQL_NO_DATA bug: Enabled")
	add_definitions(-DNANODBC_HANDLE_NODATA_BUG)
else()
	message(STATUS "Handle SQL_NO_DATA bug: Enabled")
endif()

########################################
## get ODBC compile and link flags
########################################
execute_process(COMMAND ${ODBC_CONFIG} --libs OUTPUT_VARIABLE ODBC_LINK_FLAGS OUTPUT_STRIP_TRAILING_WHITESPACE)
execute_process(COMMAND ${ODBC_CONFIG} --cflags OUTPUT_VARIABLE ODBC_COMPILE_FLAGS OUTPUT_STRIP_TRAILING_WHITESPACE)

########################################
## optional NANODBC_USE_UNICODE
########################################
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} ${ODBC_LINK_FLAGS}")

########################################
## shared library
########################################
add_library(nanodbc SHARED src/nanodbc.cpp)
set_target_properties(nanodbc PROPERTIES
	COMPILE_FLAGS "${ODBC_COMPILE_FLAGS}"
	LIBRARY_OUTPUT_DIRECTORY "lib"
)

install(FILES src/nanodbc.h DESTINATION include)
install(TARGETS nanodbc LIBRARY DESTINATION lib)

########################################
## unit tests
########################################
enable_testing()
add_subdirectory(test)
add_test(NAME test COMMAND tests)
add_custom_target(check
	COMMAND ${CMAKE_CTEST_COMMAND} --force-new-ctest-process --output-on-failure
	DEPENDS tests)
