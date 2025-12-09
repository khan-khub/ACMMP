cmake_minimum_required (VERSION 2.8)
project (ACMMP)

set(CUDA_TOOLKIT_ROOT_DIR "/usr/local/cuda")
find_package(CUDA 6.0 REQUIRED ) # For Cuda Managed Memory and c++11
find_package(OpenCV REQUIRED )


include_directories(${OpenCV_INCLUDE_DIRS})
include_directories(.)


# Detect CUDA architecture
if(NOT DEFINED CMAKE_CUDA_ARCHITECTURES)
    # Try to detect GPU compute capability
    execute_process(
        COMMAND nvidia-smi --query-gpu=compute_cap --format=csv,noheader
        OUTPUT_VARIABLE GPU_COMPUTE_CAP
        OUTPUT_STRIP_TRAILING_WHITESPACE
        ERROR_QUIET
    )
    
    if(GPU_COMPUTE_CAP)
        string(REPLACE "." "" GPU_ARCH "${GPU_COMPUTE_CAP}")
        message(STATUS "Detected GPU compute capability: ${GPU_COMPUTE_CAP} (sm_${GPU_ARCH})")
        set(CMAKE_CUDA_ARCHITECTURES ${GPU_ARCH})
    else()
        # Fallback to RTX 5080 default
        message(STATUS "Could not detect GPU, using sm_100 (RTX 50 series)")
        set(CMAKE_CUDA_ARCHITECTURES 100)
    endif()
endif()

set(CUDA_NVCC_FLAGS 
    ${CUDA_NVCC_FLAGS}
    -O3 
    --use_fast_math 
    --maxrregcount=128 
    --ptxas-options=-v 
    -std=c++11 
    --compiler-options -Wall
)


# set(CUDA_NVCC_FLAGS
#     ${CUDA_NVCC_FLAGS};
#     -O3
#     --use_fast_math
#     --maxrregcount=128
#     --ptxas-options=-v
#     -std=c++11
#     --compiler-options -Wall
    
#     # Modern GPU support
#     -gencode arch=compute_61,code=sm_61      # Pascal
#     -gencode arch=compute_75,code=sm_75      # Turing
# )


if(CMAKE_COMPILER_IS_GNUCXX)
    add_definitions(-std=c++11)
    add_definitions(-pthread)
    add_definitions(-Wall)
    add_definitions(-Wextra)
    add_definitions(-pedantic)
    add_definitions(-Wno-unused-function)
    add_definitions(-Wno-switch)
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3 -ffast-math -march=native") # extend release-profile with fast-math
endif()

find_package(OpenMP)
if (OPENMP_FOUND)
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()


# For compilation ...
# Specify target & source files to compile it from
cuda_add_executable(
    ACMMP
    main.h
    ACMMP.h
    ACMMP.cpp
    ACMMP.cu
    main.cpp
    )

# For linking ...
# Specify target & libraries to link it with
target_link_libraries(ACMMP
    ${OpenCV_LIBS}
    )
