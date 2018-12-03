# ******************************************************************************
# Copyright 2017-2018 Intel Corporation
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ******************************************************************************

if("${CMAKE_SOURCE_DIR}" STREQUAL "${CMAKE_CURRENT_SOURCE_DIR}")
    message(FATAL_ERROR "Please call from nGraph !")
endif()

if (NOT NGRAPH_CPU_ENABLE OR NOT NGRAPH_DEX_ONLY)
    message(WARNING "Python API requires NGRAPH_CPU_ENABLE and NGRAPH_DEX_ONLY!")
    return()
endif()

find_package(PythonInterp 2.7 REQUIRED)
find_program(VIRTUALENV virtualenv)
if (NOT VIRTUALENV)
    message(FATAL_ERROR "Please install virtualenv!")
endif()

message(STATUS "Python:" ${PYTHON_EXECUTABLE})

foreach(_dep_lib ${NGRAPH_PYTHON_DIST_LIBS})
    message(STATUS "Python dep lib:" ${_dep_lib})
endforeach()

include(ExternalProject)

set(PYBIND11_SOURCE_DIR "${CMAKE_BINARY_DIR}/pybind11")

ExternalProject_Add(
        pybind11
        GIT_REPOSITORY "https://github.com/pybind/pybind11.git"
        GIT_TAG "v2.2.4"
        SOURCE_DIR ${PYBIND11_SOURCE_DIR}
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
)

set(SETUP_PY_IN "${CMAKE_SOURCE_DIR}/python/setup.py.in")
set(SETUP_PY    "${CMAKE_BINARY_DIR}/setup.py")
configure_file(${SETUP_PY_IN} ${SETUP_PY})

set(TOX_INI_IN "${CMAKE_SOURCE_DIR}/python/tox.ini.in")
set(TOX_INI    "${CMAKE_BINARY_DIR}/tox.ini")
configure_file(${TOX_INI_IN} ${TOX_INI})


add_custom_command(
        OUTPUT ${CMAKE_BINARY_DIR}/wheel.timestamp
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
        COMMAND ${VIRTUALENV} -p ${PYTHON_EXECUTABLE} .venv
        COMMAND ./.venv/bin/python setup.py bdist_wheel
        COMMAND touch wheel.timestamp
        DEPENDS pybind11 ngraph cpu_backend
)

add_custom_target(python_wheel DEPENDS ${CMAKE_BINARY_DIR}/wheel.timestamp)
