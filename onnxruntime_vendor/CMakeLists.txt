# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.

cmake_minimum_required(VERSION 3.5)
project(onnxruntime_vendor)

find_package(ament_cmake REQUIRED)
find_package(CUDA QUIET)

option(CUDA_SUPPORT "use CUDA support onnxruntime library" OFF)

# disable Cuda build on github ci
if(DEFINED ENV{GITHUB_WORKSPACE})
    message(STATUS "Detected Github CI, building without CUDA")
    set(CUDA_SUPPORT OFF) 
endif()

if(MSVC)
  set(ONNX_RUNTIME "Microsoft.ML.OnnxRuntime.DirectML.1.7.0")
  set(PACKAGE_URL "https://www.nuget.org/api/v2/package/Microsoft.ML.OnnxRuntime.DirectML/1.7.0")
  set(PACKAGE_SHA512 "2e5bd2c0ade72444d4efdfbd6a75571aaa72045769f9b5847186129c9e5e667ad080d5d2b9a12cce88c9eee68302be89cdb7030ccefa3d572e591b1c453c7340")
elseif(CUDA_SUPPORT)
  set(ONNX_RUNTIME "Microsoft.ML.OnnxRuntime.Gpu.1.7.1")
  set(PACKAGE_URL "https://www.nuget.org/api/v2/package/Microsoft.ML.OnnxRuntime.Gpu/1.7.1")
  set(PACKAGE_SHA512 "41112118007aae34fcc38100152df6e6fa5fc567e61aa4ded42a26d39751f1be7ec225c0d73799f065015e284f0fb9bd7e0835c733e9abad5b0243a391411f8d")
else()
  set(ONNX_RUNTIME "Microsoft.ML.OnnxRuntime.1.7.0")
  set(PACKAGE_URL "https://www.nuget.org/api/v2/package/Microsoft.ML.OnnxRuntime/1.7.0")
  set(PACKAGE_SHA512 "1fc15386bdfa455f457e50899e3c9c454aafbdc345799dcf4ecfd6990a9dbd8cd7f0b1f3bf412c47c900543c535f95aa1cb1e14e9851cb9b600c60a981f38a50")
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
        ${CMAKE_CURRENT_BINARY_DIR}/onnxruntime/runtimes/win-x64/native/onnxruntime.dll
        ${CMAKE_CURRENT_BINARY_DIR}/onnxruntime/runtimes/win-x64/native/onnxruntime.pdb
    DESTINATION bin
)

install(
    FILES
        ${CMAKE_CURRENT_BINARY_DIR}/onnxruntime/runtimes/win-x64/native/onnxruntime.lib
    DESTINATION lib
)

install(
    FILES
        ${CMAKE_CURRENT_BINARY_DIR}/onnxruntime/runtimes/linux-x64/native/libonnxruntime.so
    DESTINATION lib
    RENAME libonnxruntime.so.1.4.0
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
