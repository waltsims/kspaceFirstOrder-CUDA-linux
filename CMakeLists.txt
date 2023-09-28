cmake_minimum_required(VERSION 3.8 FATAL_ERROR)

project(cmake_and_cuda LANGUAGES C CXX CUDA)

set(CMAKE_VERBOSE_MAKEFILE ON)

find_package(ZLIB)
if (ZLIB_FOUND)
  message(STATUS "zlib found: ${ZLIB_INCLUDE_DIRS}, ${ZLIB_LIBRARIES}, ${ZLIB_VERSION}")
endif()

find_package(HDF5 COMPONENTS C CXX HL REQUIRED)
if (HDF5_FOUND)
  message(STATUS "HDF5 found: ${HDF5_INCLUDE_DIRS}, ${HDF5_LIBRARIES}, ${HDF5_HL_LIBRARIES}, ${HDF5_VERSION}, ${HDF5_DEFINITIONS}")
endif()

find_package(Threads)
find_package(OpenMP)

file(GLOB Containers  "Containers/*.cpp" "Containers/*.h")
file(GLOB getoptWin64  "getoptWin64/*.cpp" "getoptWin64/*.h")
file(GLOB Hdf5  "Hdf5/*.cpp" "Hdf5/*.h")
file(GLOB KSpaceSolver  "KSpaceSolver/*.cpp" "KSpaceSolver/*.h")
file(GLOB Logger  "Logger/*.cpp" "Logger/*.h")
file(GLOB MatrixClasses  "MatrixClasses/*.cpp" "MatrixClasses/*.h")
file(GLOB OutputStreams  "OutputStreams/*.cpp" "OutputStreams/*.h")
file(GLOB Parameters  "Parameters/*.cpp" "Parameters/*.h")
file(GLOB Utils  "Utils/*.cpp" "Utils/*.h")

file(GLOB CUDA_FILES "KSpaceSolver/*.cu" "Containers/*.cu" "MatrixClasses/*.cu" "OutputStreams/*.cu" "Parameters/*.cu")

set(CXXFILES "main.cpp" ${Containers} ${getoptWin64} ${Hdf5} ${KSpaceSolver} ${Logger} ${MatrixClasses} ${OutputStreams} ${Parameters} ${Utils} ${CUDA_FILES})

add_executable(kspaceFirstOrder-CUDA ${CXXFILES})

# Request that particles be built with -std=c++11
# As this is a public compile feature anything that links to particles
# will also build with -std=c++11
target_compile_features(kspaceFirstOrder-CUDA  PUBLIC cxx_std_11)

# We need to explicitly state that we need all CUDA files in the particle
# library to be built with -dc as the member functions could be called by
# other libraries and executables
set_target_properties(kspaceFirstOrder-CUDA  PROPERTIES  CUDA_SEPARABLE_COMPILATION ON )

set_target_properties(kspaceFirstOrder-CUDA  PROPERTIES  RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../kwave/bin/linux/" )
message(STATUS "runtime output directory is \"${CMAKE_RUNTIME_OUTPUT_DIRECTORY}\"")

# linking
target_link_directories(kspaceFirstOrder-CUDA
    PUBLIC 
        ${HDF5_INCLUDE_DIRS}
        ${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES} 
        ${CUDA_LIBRARIES})

target_include_directories(kspaceFirstOrder-CUDA 
    PUBLIC 
        Containers
        getoptWin64
        Hdf5
        KSpaceSolver ${CMAKE_SOURCE_DIR}/KSpaceSolver
        Logger
        MatrixClasses
        OutputStreams
        Parameters
        Utils
        ${HDF5_INCLUDE_DIRS}
        ${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES}
        ${CMAKE_SOURCE_DIR}
)

list(APPEND CUDA_NVCC_FLAGS "-gencode arch=compute_30,code=sm_30; -std=c++11")

target_link_libraries(kspaceFirstOrder-CUDA  z sz dl m 
fftw3f fftw3f_omp 
${HDF5_LIBRARIES} ${HDF5_HL_LIBRARIES} ${HDF5_CXX_LIBRARIES} ${HDF5_CXX_HL_LIBRARIES} 
fftw3 fftw3_threads Threads::Threads ${CMAKE_THREAD_LIBS_INIT} 
OpenMP::OpenMP_CXX
cudart
cufft)

message(STATUS "${CMAKE_SYSTEM_NAME}")
string(TOLOWER "${CMAKE_SYSTEM_NAME}" sys_name)
message(STATUS "${sys_name}" )

if("${CMAKE_SYSTEM_NAME}" STREQUAL "Linux")
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../kwave/bin/linux/")
elseif("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../kwave/bin/windows/")
endif()

message(STATUS "runtime output directory is \"${CMAKE_RUNTIME_OUTPUT_DIRECTORY}\"")
