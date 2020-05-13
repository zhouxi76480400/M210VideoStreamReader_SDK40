cmake_minimum_required(VERSION 2.8)
project(VideoStreamReader)


include_directories(${DJIOSDK_INCLUDE_DIRS})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/common)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/hal)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/osal)

set(ARCH armv8)
add_definitions(-DDJIOSDK_HARDWARE_TYPE=2)

set(OSDK_CORE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../osdk-core)
set(CMAKE_MODULE_PATH ${OSDK_CORE_PATH}/cmake-modules)

message("-----------OSDK_LIB_HEADERS_DIR------------")
set(OSDK_LIB_HEADERS_DIR
        ${OSDK_CORE_PATH}/api/inc
        ${OSDK_CORE_PATH}/modules/inc/payload
        ${OSDK_CORE_PATH}/modules/inc/flight
        ${OSDK_CORE_PATH}/modules/inc/mop
        ${OSDK_CORE_PATH}/platform/inc
        ${OSDK_CORE_PATH}/logger/inc
        ${OSDK_CORE_PATH}/utility/inc
        ${OSDK_CORE_PATH}/linker/${ARCH}/inc
        ${OSDK_CORE_PATH}/modules/inc/filemgr
        )
include_directories(${OSDK_LIB_HEADERS_DIR})
message((${OSDK_LIB_HEADERS_DIR}))

message("-----------ADVANCED_SENSING_HEADERS_DIR------------")
set(ADVANCED_SENSING_SOURCE_ROOT ${OSDK_CORE_PATH}/advanced-sensing)
set(ADVANCED_SENSING_HEADERS_DIR
        ${ADVANCED_SENSING_SOURCE_ROOT}/api/inc
        ${ADVANCED_SENSING_SOURCE_ROOT}/platform/inc
        ${ADVANCED_SENSING_SOURCE_ROOT}/protocol/inc
        ${ADVANCED_SENSING_SOURCE_ROOT}/camera_stream/src
        ${ADVANCED_SENSING_SOURCE_ROOT}/ori-osdk-core/protocol/inc
        ${ADVANCED_SENSING_SOURCE_ROOT}/ori-osdk-core/hal/inc
        ${ADVANCED_SENSING_SOURCE_ROOT}/ori-osdk-core/platform/linux/inc
        )
include_directories(${ADVANCED_SENSING_HEADERS_DIR})
message((${ADVANCED_SENSING_HEADERS_DIR}))

find_package(LibUSB REQUIRED)
find_package(FFMPEG REQUIRED)

if(FFMPEG_FOUND)
    message( STATUS "Found FFmpeg ${FFMPEG_VERSION} installed in the system.")
    message( STATUS " - Includes: ${FFMPEG_INCLUDE_DIRS}")
    message( STATUS " - Libraries: ${FFMPEG_LIBRARIES}")
else()
    message("Cannot Find FFMPEG")
endif(FFMPEG_FOUND)

include_directories(${LIBUSB_1_INCLUDE_DIRS})
include_directories(${FFMPEG_INCLUDE_DIR})

link_libraries(advanced-sensing)
link_libraries(djiosdk-core)
link_libraries(${LIBUSB_1_LIBRARIES})
link_libraries(${FFMPEG_LIBRARIES})

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fPIC -pthread -g -O0")

FILE(GLOB SOURCE_FILES *.hpp *.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/common/dji_linux_environment.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/common/dji_linux_helpers.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/hal/*.c
        ${CMAKE_CURRENT_SOURCE_DIR}/osal/*.c
        ${LINKER_HEADER_SRC}
        )

find_package( OpenCV QUIET )
if (OpenCV_FOUND)
    message( "\n${PROJECT_NAME}...")
    message( STATUS "Found OpenCV installed in the system, will use it to display image in AdvancedSensing APIs")
    message( STATUS " - Includes: ${OpenCV_INCLUDE_DIRS}")
    message( STATUS " - Libraries: ${OpenCV_LIBRARIES}")
    add_definitions(-DOPEN_CV_INSTALLED)
else()
    message( STATUS "Did not find OpenCV in the system, image data is inside RecvContainer as raw data")
endif ()

set(HELPER_FUNCTIONS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/common)
add_executable(${PROJECT_NAME}
        ${SOURCE_FILES}
        ${HELPER_FUNCTIONS_DIR}/dji_linux_environment.cpp
        ${HELPER_FUNCTIONS_DIR}/dji_linux_helpers.cpp
        main.cpp
        )
target_include_directories(${PROJECT_NAME} PRIVATE ${OpenCV_INCLUDE_DIRS})

if (OpenCV_FOUND)
    target_link_libraries(${PROJECT_NAME}
            ${OpenCV_LIBS}
            )
endif ()