# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.

# Original code has been modified to support additional ONNX Runtime versions.

cmake_minimum_required(VERSION 3.8)
project(onnxruntime_vendor)

find_package(ament_cmake REQUIRED)

# Disable Cuda build on GitHub CI
if(DEFINED ENV{GITHUB_WORKSPACE})
    message("Detected GitHub CI, building without CUDA")
    option(CUDA_SUPPORT "Use CUDA support in ONNX Runtime library" OFF)
else()
    option(CUDA_SUPPORT "Use CUDA support in ONNX Runtime library" ON)
endif()

# Set default ONNX Runtime version
set(ONNXRUNTIME_VERSION "1.15.0" CACHE STRING "ONNX Runtime version to use")

# Check if ONNX Runtime version is provided as a command line argument
if(DEFINED ENV{ONNXRUNTIME_VERSION})
    set(ONNXRUNTIME_VERSION $ENV{ONNXRUNTIME_VERSION} CACHE STRING "ONNX Runtime version to use" FORCE)
endif()

if(CUDA_SUPPORT)
    message("Building with CUDA support")
    set(PACKAGE_URL "https://www.nuget.org/api/v2/package/Microsoft.ML.OnnxRuntime.Gpu/${ONNXRUNTIME_VERSION}")
    
    if(${ONNXRUNTIME_VERSION} STREQUAL "1.4.0")
        set(PACKAGE_SHA512 "c9c2ba5c594c92c1e426e9c53f9909e8851a41c99f48f8a369e082f8047d521b236f2fbb943e73975cbb45bd9957f20139c25959e50e1679dca9eeac08f73b31")
    elseif(${ONNXRUNTIME_VERSION} STREQUAL "1.6.0") 
        set(PACKAGE_SHA512 "8f838e54ffc888ec852a3ecc83919692bf28e354ab1bc5c2a8dad6b4fd5653870abb5c4d0f101d976ce5cdb9e35646264dabf12023155e3443fc6edf0355d41c")
    elseif(${ONNXRUNTIME_VERSION} STREQUAL "1.12.1")
        set(PACKAGE_SHA512 "23001237d1c9c49b28c02e1741431b06740fd86ff056fce766fd9eb62efd41eebe285d664eb6b66c8fb5079614149a15fd7bc011c40eda0ea230bd69a699553c")
    elseif(${ONNXRUNTIME_VERSION} STREQUAL "1.14.1")
        set(PACKAGE_SHA512 "19c62e2e94098d534a0ed18ba44e34e3461624f71d7fe6f708c81c21fc5c5e47ab5e87900c805a16d74ccb5655272ce2fb5cd6785df6fb9b9dac238bc025f95e")
    elseif(${ONNXRUNTIME_VERSION} STREQUAL "1.15.0")
        set(PACKAGE_SHA512 "4f9c57aa7288efcb908fa2d2d56f146b49a988c3494c2674cbf635f69231e410d33b2ea33dcc51299ab5181ef52fbcaa19078417d1ef5c2c877e66318994fe4f")
    else()
        message(FATAL_ERROR "Unsupported ONNX Runtime version")
    endif()
else()
    message("Building without CUDA support")
    set(PACKAGE_URL "https://www.nuget.org/api/v2/package/Microsoft.ML.OnnxRuntime/${ONNXRUNTIME_VERSION}")

    if(${ONNXRUNTIME_VERSION} STREQUAL "1.15.0")
        set(PACKAGE_SHA512 "0f3dd328e6e1174c5c5c703096029c43a8936d1eae7f9254f3ec6ccf9b7bbc502f047d870a8e8bab58c928a77d1c8dae5d6f0edc9bd04a41822733e1b47f6395")
    else()
        message(FATAL_ERROR "Unsupported ONNX Runtime version")
    endif()
endif()

file(DOWNLOAD
    ${PACKAGE_URL}
    ${CMAKE_CURRENT_BINARY_DIR}/onnxruntime.nuget
    EXPECTED_HASH SHA512=${PACKAGE_SHA512}
    SHOW_PROGRESS
)

file(MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/onnxruntime")

add_custom_target(ExtractOnnxruntime ALL
    ${CMAKE_COMMAND} -E
    tar xvzf "${CMAKE_CURRENT_BINARY_DIR}/onnxruntime.nuget"
    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/onnxruntime"
)

install(
    DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/onnxruntime/build/native/include/
    DESTINATION include
)

install(
    FILES
        ${CMAKE_CURRENT_BINARY_DIR}/onnxruntime/runtimes/linux-x64/native/libonnxruntime.so
    DESTINATION lib
    RENAME libonnxruntime.so
)

install(
    FILES
        ${CMAKE_CURRENT_BINARY_DIR}/onnxruntime/LICENSE.txt
        ${CMAKE_CURRENT_BINARY_DIR}/onnxruntime/ThirdPartyNotices.txt
    DESTINATION share/${PROJECT_NAME}
)

ament_package(
  CONFIG_EXTRAS "src/onnxruntime_vendor-extras.cmake"
)
