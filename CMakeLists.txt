cmake_minimum_required(VERSION 3.1) 
project(yolo-trt VERSION 1.0)            
enable_language(CUDA)

set(CMAKE_CXX_COMPILIER "/usr/bin/gcc")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -O3 -Wno-write-strings")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-rpath -Wl,$ORIGIN")

#cuda
find_package(CUDA 10.2 REQUIRED)
include_directories(${CUDA_INCLUDE_DIRS})
link_directories(/usr/local/cuda-10.2/lib64)

set(CUDA_NVCC_FLAGS --disable-warnings;-std=c++11;-O3;-gencode arch=compute_75, code=sm_75)
set(CUDA_LIBS cublas cudart curand cufft nvonnxparser nvinfer cuda)

#opencv
find_package(OpenCV REQUIRED)

# tensorRT 8 TensorRT是找不到的，需要set设置一下路径
set(TENSORRT_ROOT "/opt/TensorRT-8.5.1.7")
include_directories(${TENSORRT_ROOT}/include)
link_directories(${TENSORRT_ROOT}/lib)


#detector lib (cuda)
file(GLOB_RECURSE sources modules/*.hpp modules/*.cpp modules/*.h modules/*.cu extra/*.h)
add_library(detector SHARED ${sources})
target_include_directories(detector PRIVATE extra/ modules/ ${OpenCV_INCLUDE_DIRS} ${CUDA_INCLUDE_DIRS} ${OpenCV_LIBS} ${CUDA_LIBS} ${TENSORRT_LIBRARIES})
target_link_libraries(detector ${CUDA_LIBS}  ${TENSORRT_LIBRARIES}  "stdc++fs")


#sample
add_executable(yolo-trt samples/sample_detector.cpp)				  
target_include_directories(yolo-trt PRIVATE modules/ extra/)
target_link_libraries(yolo-trt detector ${OpenCV_LIBS} ${CUDA_LIBS} ${TENSORRT_LIBRARIES})



