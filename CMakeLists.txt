#[==[
 * This file is part of UAHThesis.
 * Copyright (C) 2024 UAH - Thomas Tarter
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU Affero General Public License as
 * published by the Free Software Foundation, either version 3 of the
 * License, or (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
 * GNU Affero General Public License for more details.
 *
 * You should have received a copy of the GNU Affero General Public License
 * along with this program. If not, see <http://www.gnu.org/licenses/>.
]==]

#[===[
# sudo dnf install libacl-devel -y
#  cd /opt/software/anaconda3/bin/
#./conda create --name py310 python=3.10
#./conda activate py310
# In Linux: /home/ttarter/.conda/envs/py310
# In Windows C:/ProgramData/Local/Local/anaconda3/envs/py310
]===]

# Set the minimum CMake version required
cmake_minimum_required(VERSION 3.10)

# Set the project name and version
project(MyProject VERSION 1.0)

# Specify the C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Set installation directories
set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/install)

# Include ExternalProject module
include(ExternalProject)
# Create the external project for Iceoryx
ExternalProject_Add(
        IceOryx
        GIT_REPOSITORY https://github.com/eclipse-iceoryx/iceoryx.git
        GIT_TAG v2.0.6
        SOURCE_SUBDIR iceoryx_meta
        CMAKE_ARGS
            -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
            -DBUILD_SHARED_LIBS=ON
        BINARY_DIR ${CMAKE_BINARY_DIR}/external_build

)
SET(iceoryx_binding_c_DIR   ${CMAKE_INSTALL_PREFIX}/lib64/cmake/iceoryx_binding_c CACHE INTERNAL "Cyclone DIR")
SET(iceoryx_hoofs_DIR       ${CMAKE_INSTALL_PREFIX}/lib64/cmake/iceoryx_hoofs CACHE INTERNAL "Cyclone DIR")
SET(iceoryx_posh_DIR        ${CMAKE_INSTALL_PREFIX}/lib64/cmake/iceoryx_posh CACHE INTERNAL "Cyclone DIR")

# Fetch CycloneDDS
ExternalProject_Add(
        CycloneDDS
        GIT_REPOSITORY https://github.com/eclipse-cyclonedds/cyclonedds.git
        GIT_TAG releases/0.10.x
        CMAKE_ARGS
            -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
)
SET(CycloneDDS_DIR ${CMAKE_INSTALL_PREFIX}/lib64/cmake/CycloneDDS CACHE INTERNAL "Cyclone DIR")
add_dependencies(CycloneDDS IceOryx)


# Fetch CycloneDDS
ExternalProject_Add(
        CycloneDDS-CXX
        GIT_REPOSITORY https://github.com/eclipse-cyclonedds/cyclonedds-cxx.git
        GIT_TAG releases/0.10.x
        CMAKE_ARGS
        -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
)
add_dependencies(CycloneDDS-CXX CycloneDDS IceOryx)
SET(CycloneDDS-CXX_DIR ${CMAKE_INSTALL_PREFIX}/lib64/cmake/CycloneDDS-CXX CACHE INTERNAL "Cyclone DIR")

include_directories(${CMAKE_INSTALL_PREFIX}/include)
include_directories(${CMAKE_INSTALL_PREFIX}/include/ddscxx)
link_directories(${CMAKE_INSTALL_PREFIX}/lib4)

#MESSAGE(STATUS "CMAKE_INSTALL_PREFIX => ${CMAKE_INSTALL_PREFIX}")
#add_custom_target(generated_helloworlddata
#        COMMAND ${CMAKE_COMMAND}
#        -S ${CMAKE_SOURCE_DIR}/partythird/Cyclonedds
#        -B ${CMAKE_SOURCE_DIR}/partythird/Cyclonedds/build
#        -DCycloneDDS-CXX_DIR=${CycloneDDS-CXX_DIR}
#        -DCycloneDDS_DIR=${CycloneDDS_DIR}
#        -Diceoryx_binding_c_DIR=${iceoryx_binding_c_DIR}
#        -Diceoryx_hoofs_DIR=${iceoryx_hoofs_DIR}
#        -Diceoryx_posh_DIR=${iceoryx_posh_DIR}
#        -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
#        COMMAND ${CMAKE_COMMAND} --build ${CMAKE_SOURCE_DIR}/partythird/Cyclonedds/build --target install
#        COMMENT "Loading another CMakeLists.txt file"
#)
#add_dependencies(generated_helloworlddata CycloneDDS CycloneDDS-CXX)


SET(CUDA_PATH           /usr/local/cuda CACHE STRING "Description of the variable")
SET(CUDACXX             /usr/local/cuda/bin CACHE STRING "Direcotry of nvcc")
SET(CMAKE_CUDA_COMPILER /usr/local/cuda/bin/nvcc CACHE STRING "Path to of nvcc")

SET(PYBIND11_PYTHON_VERSION 3.10  CACHE STRING "Python Root")
SET(PYTHON_EXECUTABLE /home/ttarter/.conda/envs/py310/bin/python3  CACHE STRING "Python Root")

#SET(PYBIND11_FINDPYTHON True  CACHE STRING "Python Root")

#set(Python_ROOT_DIR         /home/ttarter/.conda/envs/py310 CACHE STRING "Python Root")
#set(Python_FIND_STRATEGY       LOCATION CACHE STRING "Python Root")


#set(PYTHON_EXECUTABLE     /home/ttarter/.conda/envs/py310/bin/python3 CACHE STRING "Python Executable")
#SET(PYTHON_INCLUDE_DIRS    /home/ttarter/.conda/envs/py310/include/python3.10 CACHE STRING "Python Include files")
#SET(Python_LIBRARY        /home/ttarter/.conda/envs/py310/lib CACHE STRING "Path to of nvcc")


#message(STATUS $ENV{CUDACXX})
#set(ENV{CUDACXX} "${CUDACXX}")
#message(STATUS $ENV{CUDACXX})
#
#message(STATUS $ENV{CMAKE_CUDA_COMPILER})
#set(ENV{CMAKE_CUDA_COMPILER} "${CMAKE_CUDA_COMPILER}")
#message(STATUS $ENV{CMAKE_CUDA_COMPILER})

add_custom_target(UAHThesis
        COMMAND ${CMAKE_COMMAND}
        -S ${CMAKE_SOURCE_DIR}/UAHThesis
        -B ${CMAKE_BINARY_DIR}/UAHThesis-build
        -DCycloneDDS-CXX_DIR=${CycloneDDS-CXX_DIR}
        -DCycloneDDS_DIR=${CycloneDDS_DIR}
        -Diceoryx_binding_c_DIR=${iceoryx_binding_c_DIR}
        -Diceoryx_hoofs_DIR=${iceoryx_hoofs_DIR}
        -Diceoryx_posh_DIR=${iceoryx_posh_DIR}
        -DPYBIND11_PYTHON_VERSION=${PYBIND11_PYTHON_VERSION}
        -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE}
        -DCMAKE_CUDA_COMPILER=${CMAKE_CUDA_COMPILER}
        -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
        COMMAND ${CMAKE_COMMAND} --build ${CMAKE_BINARY_DIR}/UAHThesis-build --target install
        COMMENT "Loading another CMakeLists.txt file"
)
add_dependencies(UAHThesis CycloneDDS-CXX CycloneDDS IceOryx)


#find_package(iceoryx_posh CONFIG REQUIRED)
#find_package(iceoryx_hoofs CONFIG REQUIRED)
# Define the main executable
