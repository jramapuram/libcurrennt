PROJECT(currennt)
CMAKE_MINIMUM_REQUIRED (VERSION 2.8)

# OSX Does not need these [Verified on 10.10]
IF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libstdc++")
ELSEIF(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libstdc++")
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

FIND_PACKAGE (CUDA)
FIND_PACKAGE (Boost 1.55.0 REQUIRED COMPONENTS program_options system filesystem random thread  REQUIRED)

INCLUDE_DIRECTORIES (${Boost_INCLUDE_DIR})
LINK_DIRECTORIES (${Boost_LIBRARY_DIR})

SET (CMAKE_BUILD_TYPE Release)
SET (CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -arch compute_20")
SET (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-format-security")

# Find the required files
FILE (GLOB_RECURSE src_lib     currennt_lib/*.cpp currennt_lib/*.hpp     currennt_lib/*.h     currennt_lib/*.cu     currennt_lib/*.cuh)
FILE (GLOB_RECURSE src_trainer currennt/*.cpp     currennt/*.hpp         currennt/*.h         currennt/*.cu         currennt/*.cuh)

# Build a static lib
CUDA_ADD_LIBRARY(${PROJECT_NAME}_static STATIC ${src_lib})
CUDA_ADD_CUBLAS_TO_TARGET (${PROJECT_NAME}_static)
TARGET_LINK_LIBRARIES (${PROJECT_NAME}_static ${Boost_LIBRARIES} netcdf)

# Build the binary current
CUDA_ADD_EXECUTABLE (${PROJECT_NAME} ${src_trainer})
CUDA_ADD_CUBLAS_TO_TARGET (${PROJECT_NAME})
TARGET_LINK_LIBRARIES (${PROJECT_NAME} netcdf ${Boost_LIBRARIES} ${PROJECT_NAME}_static)