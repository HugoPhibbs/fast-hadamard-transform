cmake_minimum_required(VERSION 3.18)
project(fht_cuda LANGUAGES CXX CUDA)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CUDA_ARCHITECTURES 86)

# CUDA, see https://stackoverflow.com/questions/66327073/how-to-find-and-link-cuda-libraries-using-cmake-3-15-and-later, and https://cmake.org/cmake/help/latest/module/FindCUDAToolkit.html
find_package(CUDAToolkit REQUIRED)

# Torch. See https://pytorch.org/cppdocs/installing.html#minimal-example
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")
set(Torch_DIR "/opt/torch/libtorch/share/cmake/Torch")
find_package(Torch REQUIRED)

# Adding in python3 headers 
set(Python3_EXECUTABLE "/usr/bin/python3")
find_package(Python3 COMPONENTS Interpreter Development REQUIRED)

file(GLOB SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/csrc/*.cpp" "${CMAKE_CURRENT_SOURCE_DIR}/csrc/*.cu")

add_library(fht_cuda STATIC
    ${SOURCES}
)

target_include_directories(fht_cuda PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/csrc
    ${TORCH_INCLUDE_DIRS}
    ${Python3_INCLUDE_DIRS}
)

target_link_libraries(fht_cuda PUBLIC ${TORCH_LIBRARIES})

target_compile_definitions(fht_cuda PRIVATE "-D_GLIBCXX_USE_CXX11_ABI=1") # match PyTorch ABI
target_compile_options(fht_cuda PRIVATE -O3)

target_compile_options(fht_cuda PRIVATE
    $<$<COMPILE_LANGUAGE:CUDA>:
        -O3
        --use_fast_math
        --expt-relaxed-constexpr
        --expt-extended-lambda
    >
)

set_target_properties(fht_cuda PROPERTIES CXX_STANDARD 17)