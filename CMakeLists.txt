﻿CMAKE_MINIMUM_REQUIRED(VERSION 3.11)
SET(project_name raftcpp)
set(CMAKE_CXX_STANDARD 17)

PROJECT(${project_name})

SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules")

SET(ENABLE_ASIO_STANDALONE ON)
if(ENABLE_ASIO_STANDALONE)
	ADD_DEFINITIONS(-DASIO_STANDALONE)
else()
	FIND_PACKAGE(Boost 1.60 REQUIRED COMPONENTS system)
endif()
INCLUDE(CompilerOptions)
INCLUDE(ExternalProject)
INCLUDE(DoctestExternalProject)
INCLUDE(AsioExternalProject)
INCLUDE(Rest_rpcExternalProject)
INCLUDE(GflagsExternalProject)
INCLUDE(CustomeFunctions)
INCLUDE_DIRECTORIES(SYSTEM ${DOCTEST_INCLUDE_DIR})
INCLUDE_DIRECTORIES(SYSTEM thirdparty/logging)
INCLUDE_DIRECTORIES(src/)
INCLUDE_DIRECTORIES(includes)
INCLUDE_DIRECTORIES(
		SYSTEM ${ASIO_INCLUDE_DIR}
		SYSTEM ${REST_RPC_INCLUDE_DIR}
		SYSTEM ${MSGPACK_INCLUDE_DIR}
		SYSTEM ${MSGPACK_INCLUDE_DIR}
		SYSTEM ${GFLAGS_INCLUDE_DIR}
)

# common library
ADD_LIBRARY(common_lib
		src/common/config.cc
		src/common/timer.cc)
ADD_DEPENDENCIES(common_lib asio_ep)

# node library
ADD_LIBRARY(node_lib src/node/node.cc src/node/timer_manager.cc)
ADD_DEPENDENCIES(node_lib asio_ep rest_rpc_ep common_lib)
TARGET_LINK_LIBRARIES(node_lib common_lib)


# Test targets
if(UNIX)
	set(link_libs pthread)
else()
	set(link_libs )
endif()

RAFTCPP_DEFINE_TEST(test_nanolog SRC_FILES tests/nanolog_test.cc )
RAFTCPP_DEFINE_TEST(example_counter_server_main SRC_FILES examples/counter/counter_server_main.cc LINKS node_lib gflags)
RAFTCPP_DEFINE_TEST(test_timer SRC_FILES tests/timer_test.cc LINKS common_lib)
RAFTCPP_DEFINE_TEST(test_range SRC_FILES tests/range_test.cc LINKS common_lib)
RAFTCPP_DEFINE_TEST(test_config SRC_FILES tests/config_test.cc LINKS common_lib)
