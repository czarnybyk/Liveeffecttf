#
# Copyright 2018 The Android Open Source Project
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
cmake_minimum_required(VERSION 3.4.1)
project(liveEffect LANGUAGES C CXX)

get_filename_component(SAMPLE_ROOT_DIR
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../.. ABSOLUTE)

### INCLUDE OBOE LIBRARY ###
set (OBOE_DIR ${SAMPLE_ROOT_DIR}/..)
set(JNI_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../jni)
add_subdirectory(${OBOE_DIR} ./oboe-bin)

add_library(liveEffect
    SHARED
        LiveEffectEngine.cpp
        FullDuplexStream.cpp
        jni_bridge.cpp
        ${SAMPLE_ROOT_DIR}/debug-utils/trace.cpp
        tflite-lib SHARED IMPORTED)
set_target_properties(tflite-lib
        PROPERTIES IMPORTED_LOCATION
        ${JNI_DIR}/${ANDROID_ABI}/libtfl.so)

include_directories( ${JNI_DIR} )
target_include_directories(liveEffect
    PRIVATE
        ${SAMPLE_ROOT_DIR}/debug-utils
        ${OBOE_DIR}/include)
target_link_libraries(liveEffect

    PRIVATE
        native-lib
        tflite-lib
        oboe
        android
        atomic
        log)

# Enable optimization flags: if having problems with source level debugging,
# disable -Ofast ( and debug ), re-enable it after done debugging.
target_compile_options(liveEffect PRIVATE -Wall -Werror "$<$<CONFIG:RELEASE>:-Ofast>")

