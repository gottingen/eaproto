#
# Copyright 2023 The titan-search Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
####################################################################################
# generate by carbin package manager tools for titan-search
# https://github.com/gottingen/carbin
####################################################################################
cmake_minimum_required(VERSION 3.15)

project(eaproto CXX)

set(PROJECT_DESCRIPTION "Hercules is an ahead of time compiler for a subset of the Python language framework.")
set(PROJECT_VERSION_MAJOR 0)
set(PROJECT_VERSION_MINOR 1)
set(PROJECT_VERSION_PATCH 6)
set(PROJECT_VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}")
set(CARBIN_VERSION 0.2.0)
list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/carbin_cmake)
list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)
#################################################################
#
# modify the CARBIN_DEPS_PREFIX to you prefix if needed.
# for example when you install you deps to "/opt/handsome/guy
# modify as set(CARBIN_DEPS_PREFIX /opt/handsome/guy)
#################################################################
set(CARBIN_DEPS_PREFIX ${PROJECT_SOURCE_DIR}/carbin)
#######################################################################
# warning
# in carbin_cmake directory, caution to modify files in that dir
######################################################################
include(carbin_module)

##################################################################
#
# your cmake directory ${PROJECT_SOURCE_DIR}/cmake, do things here
#################################################################
include(user_deps)
include(user_cxx_config)

include(carbin_cc_proto)
#file(GLOB PROTO_FILES "${PROJECT_SOURCE_DIR}/elasticann/proto/*.proto")
set(PROTO_FILES
        eaproto/common/error_code.proto
        eaproto/common/optype.proto
        eaproto/common/version.proto
        eaproto/db/base_subscribe.proto
        eaproto/db/binlog.proto
        eaproto/db/common.proto
        eaproto/db/console.proto eaproto/db/event.proto
        eaproto/db/expr.proto
        eaproto/db/fc.proto
        eaproto/db/meta.interface.proto
        eaproto/db/meta.struct.proto
        eaproto/db/plan.proto
        eaproto/db/reverse.proto
        eaproto/db/statistics.proto
        eaproto/db/store.interface.proto
        eaproto/db/store.struct.proto
        eaproto/db/test_decode.proto
        eaproto/discovery/optype.proto
        eaproto/discovery/discovery.struct.proto
        eaproto/discovery/discovery.interface.proto
        eaproto/ops/config_struct.proto
        eaproto/ops/dict_struct.proto
        eaproto/ops/plugin_struct.proto
        eaproto/ops/ops.struct.proto
        eaproto/ops/ops.interface.proto
        eaproto/raft/raft.proto
        eaproto/router/router.interface.proto
)

file(MAKE_DIRECTORY ${PROJECT_BINARY_DIR}/output/include/eaproto)
set(PROTOC_FLAGS ${PROTOC_FLAGS} -I${PROTOBUF_INCLUDE_DIR} -I${PROJECT_SOURCE_DIR})

carbin_cc_proto(PROTO_HDRS PROTO_SRCS ${PROJECT_BINARY_DIR}
        ${PROJECT_BINARY_DIR}/output/include
        ${PROJECT_SOURCE_DIR}
        "${PROTO_FILES}")

carbin_cc_library(
        NAMESPACE eaproto
        NAME eaproto
        SOURCES ${PROTO_SRCS}
        COPTS ${CARBIN_DEFAULT_COPTS}
        DEPS
        ${CARBIN_DYLINK}
        PUBLIC
)

####################################################################
# belows are auto, edit it be cation
####################################################################
if (CARBIN_BUILD_TEST)
    #add_subdirectory(tests)
endif ()

if (CARBIN_BUILD_BENCHMARK)
    #add_subdirectory(benchmark)
endif ()

if (CARBIN_BUILD_EXAMPLES)
    #add_subdirectory(examples)
endif ()

##############################################
# header installing
#############################################
install(DIRECTORY ${PROJECT_NAME}
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        FILES_MATCHING
        PATTERN "*.inc"
        PATTERN "*.h"
)

# Layout. This works for all platforms:
#   * <prefix>/lib*/cmake/<PROJECT-NAME>
#   * <prefix>/lib*/
#   * <prefix>/include/
set(config_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

# Configuration
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

# Include module with fuction 'write_basic_package_version_file'
include(CMakePackageConfigHelpers)

# Configure '<PROJECT-NAME>ConfigVersion.cmake'
# Use:
#   * PROJECT_VERSION
write_basic_package_version_file(
        "${version_config}" COMPATIBILITY SameMajorVersion
)

# Configure '<PROJECT-NAME>Config.cmake'
# Use variables:
#   * TARGETS_EXPORT_NAME
#   * PROJECT_NAME
configure_package_config_file(
        "carbin_cmake/package/carbin_config.cmake.in"
        "${project_config}"
        INSTALL_DESTINATION "${config_install_dir}"
)

#Targets:
#  * <prefix>/lib/libbar.a
#  * <prefix>/lib/libbaz.a
#  * header location after install: <prefix>/include/foo/Bar.hpp
#  * headers can be included by C++ code `#include <foo/Bar.hpp>`


# Config
#   * <prefix>/lib/cmake/Foo/FooConfig.cmake
#   * <prefix>/lib/cmake/Foo/FooConfigVersion.cmake
install(
        FILES
        "${project_config}" "${version_config}"
        DESTINATION
        "${config_install_dir}"
)

# Config
#   * <prefix>/lib/cmake/Foo/FooTargets.cmake

install(
        EXPORT
        "${TARGETS_EXPORT_NAME}"
        NAMESPACE
        "${namespace}"
        DESTINATION
        "${config_install_dir}"
)
