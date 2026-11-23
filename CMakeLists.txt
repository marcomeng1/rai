cmake_minimum_required(VERSION 3.5)

project(rai_devel)

## variable default values
set(CMAKE_INSTALL_MESSAGE LAZY)
set(CMAKE_CXX_STANDARD 17 CACHE STRING "C++ version selection")
## dependencies are expected in an active pixi/conda environment (e.g. botop's pixi.toml)
if(DEFINED ENV{CONDA_PREFIX})
  list(PREPEND CMAKE_PREFIX_PATH "$ENV{CONDA_PREFIX}")
  if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set(CMAKE_INSTALL_PREFIX "$ENV{CONDA_PREFIX}" CACHE PATH "..." FORCE)
  endif()
elseif(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(CMAKE_INSTALL_PREFIX "$ENV{HOME}/.local" CACHE PATH "..." FORCE)
endif()
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
set(CMAKE_INSTALL_RPATH "\$ORIGIN;${CMAKE_INSTALL_PREFIX}/lib")

## options
option(USE_PHYSX "link to physx" ON)
option(USE_H5 "link to h5 library" ON)
option(USE_FREETYPE "link to freetype2 library" ON)
option(USE_OPENCV "link to opencv" OFF)
option(BUILD_TESTS "build tests" OFF)

## compile options
add_compile_options(
  -O3 -g -Wall -Wpedantic
  $<$<COMPILE_LANGUAGE:CXX>:-std=c++17>
  $<$<COMPILE_LANGUAGE:CXX>:-Wno-terminate>
  $<$<COMPILE_LANGUAGE:CXX>:-Wno-array-bounds>
  )

################################################################################

add_definitions(
  -DRAI_ROOT_PATH=\"${CMAKE_SOURCE_DIR}\"
  -DRAI_X11 -DRAI_LAPACK -DRAI_EIGEN -DRAI_PTHREAD
  -DRAI_GL
  -DRAI_GLFW -DRAI_ANN -DRAI_QHULL8 -DRAI_ASSIMP -DNO_CYCLE_DETECTION -DRAI_PNG -DRAI_PLY -DRAI_YAML
  -DRAI_GJK -DRAI_CCD -DRAI_FCL -DRAI_Lewiner -DRAI_VHACD
  )

################################################################################

include_directories(
  src
  )

file(GLOB SRC_Core src/Core/*.cpp)
file(GLOB SRC_Optim src/Optim/*.cpp)
file(GLOB SRC_Geo src/Geo/*.cpp)
file(GLOB SRC_Algo src/Algo/*.cpp)
file(GLOB SRC_Search src/Search/*.cpp)
file(GLOB SRC_PathAlgos src/PathAlgos/*.cpp)
file(GLOB SRC_Gui src/Gui/*.cpp)
file(GLOB SRC_Kin src/Kin/*.cpp)
file(GLOB SRC_KOMO src/KOMO/*.cpp)
file(GLOB SRC_Logic src/Logic/*.cpp)
file(GLOB SRC_LGP src/LGP/*.cpp)
file(GLOB SRC_Control src/Control/*.cpp)
file(GLOB SRC_DataGen src/DataGen/*.cpp)
file(GLOB SRC_Perception src/Perception/*.cpp)

file(GLOB SRC_AlgoLewiner src/Algo/Lewiner/*.cpp)
file(GLOB SRC_GeoGJK src/Geo/GJK/*.c)
file(GLOB SRC_GeoPly src/Geo/ply/*.c)
file(GLOB SRC_GeoCcd src/Geo/ccd_rai/*.c)
file(GLOB SRC_OptLBFGS src/Optim/liblbfgs/*.c)
file(GLOB SRC_OptCMA src/Optim/CMA/*.c)

add_library(raiobjs OBJECT
  ${SRC_Core}
  ${SRC_Optim}
  ${SRC_Geo}
  ${SRC_AlgoLewiner}
  ${SRC_GeoGJK}
  ${SRC_GeoPly}
  ${SRC_GeoCcd}
  ${SRC_OptLBFGS}
  ${SRC_OptCMA}
  ${SRC_Algo}
  ${SRC_Search}
  ${SRC_PathAlgos}
  ${SRC_Gui}
  ${SRC_Kin}
  ${SRC_KOMO}
  ${SRC_Logic}
  ${SRC_LGP}
  ${SRC_Control}
  ${SRC_DataGen}
  ${SRC_Perception}
  )

################################################################################

set_property(TARGET raiobjs PROPERTY POSITION_INDEPENDENT_CODE 1)
add_library(rai SHARED $<TARGET_OBJECTS:raiobjs>)
add_library(rai_static STATIC $<TARGET_OBJECTS:raiobjs>)

################################################################################

## all dependencies are collected in an interface target, so that include paths
## also reach the object library raiobjs
add_library(rai_deps INTERFACE)
target_link_libraries(raiobjs PUBLIC rai_deps)
target_link_libraries(rai PUBLIC rai_deps)
target_link_libraries(rai_static PUBLIC rai_deps)

find_package(PkgConfig REQUIRED)
find_package(Eigen3 3.3 REQUIRED NO_MODULE)
find_package(GLUT REQUIRED)
pkg_check_modules(DEPS REQUIRED IMPORTED_TARGET glfw3 glew glu yaml-0.1 libpng assimp ccd)
find_library(FCL_LIB NAMES fcl REQUIRED)  # not via pkg-config: fcl.pc forces -std=c++11
find_library(ANN_LIB NAMES ANN ann REQUIRED)
find_library(QHULL_LIB NAMES qhullstatic qhull REQUIRED)
find_library(LAPACK_LIB NAMES lapack REQUIRED)
find_library(BLAS_LIB NAMES blas REQUIRED)

target_link_libraries(rai_deps INTERFACE
  Eigen3::Eigen PkgConfig::DEPS GLUT::GLUT
  ${FCL_LIB} ${ANN_LIB} ${QHULL_LIB} ${LAPACK_LIB} ${BLAS_LIB})

if(USE_FREETYPE)
  find_package(Freetype REQUIRED)
  add_definitions(-DRAI_FREETYPE)
  target_link_libraries(rai_deps INTERFACE Freetype::Freetype)
endif()

if(USE_H5)
  find_package(HDF5 REQUIRED COMPONENTS C CXX)
  add_definitions(-DRAI_H5)
  target_include_directories(rai_deps INTERFACE ${HDF5_INCLUDE_DIRS})
  target_link_libraries(rai_deps INTERFACE ${HDF5_CXX_LIBRARIES} ${HDF5_C_LIBRARIES})
endif()

if(USE_PHYSX)
  find_path(PHYSX_INCLUDE_DIR PxPhysicsAPI.h REQUIRED)
  set(PHYSX_LIB_DIR "${PHYSX_INCLUDE_DIR}/../lib/PhysX")
  add_definitions(-DRAI_PHYSX -DNDEBUG -DPX_DISABLE_FLUIDS -DCORELIB -DPX32 -DLINUX)
  target_include_directories(rai_deps INTERFACE ${PHYSX_INCLUDE_DIR})
  foreach(px PhysXExtensions PhysX PhysXCooking PhysXCommon PhysXFoundation)
    target_link_libraries(rai_deps INTERFACE "${PHYSX_LIB_DIR}/lib${px}_static_64.a")
  endforeach()
endif()

if(USE_OPENCV)
  find_package(OpenCV REQUIRED COMPONENTS core highgui imgproc videoio)
  add_definitions(-DRAI_OPENCV)
  target_link_libraries(rai_deps INTERFACE ${OpenCV_LIBS})
endif()

################################################################################

get_property(_defs DIRECTORY PROPERTY COMPILE_DEFINITIONS)
message(STATUS "[rai] compiler flags: " "${_defs}")
message(STATUS "[rai] source dir: " "${CMAKE_SOURCE_DIR}")

################################################################################

add_executable(kinEdit bin/src_kinEdit/main.cpp)
target_link_libraries(kinEdit rai)

add_executable(meshTool bin/src_meshTool/main.cpp)
target_link_libraries(meshTool rai)

if(BUILD_TESTS)
file(GLOB tests test/*/*/main.cpp)
foreach(file ${tests})
  get_filename_component(path ${file} DIRECTORY)
  get_filename_component(name ${path} NAME)
  add_executable(${name} ${file})
  target_link_libraries(${name} rai)
endforeach( file ${tests} )
endif()

################################################################################

message(STATUS "[rai] installing headers and lib in " ${CMAKE_INSTALL_PREFIX}/include/rai|lib)

install(
  DIRECTORY src/
  DESTINATION ${CMAKE_INSTALL_PREFIX}/include/rai
  FILES_MATCHING
  PATTERN "*.h" PATTERN "*.ipp"
  PATTERN "*retired*" EXCLUDE
  )

install(
  TARGETS rai rai_static
  DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)

install(
  TARGETS kinEdit meshTool
  DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)

