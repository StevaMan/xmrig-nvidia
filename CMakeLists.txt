cmake_minimum_required(VERSION 3.0)
project(xmrig-nvidia)

option(WITH_AEON "CryptoNight-Lite support" ON)


include (CheckIncludeFile)


set(HEADERS
    src/3rdparty/align.h
    src/App.h
    src/Console.h
    src/Cpu.h
    src/interfaces/IClientListener.h
    src/interfaces/IConsoleListener.h
    src/interfaces/IJobResultListener.h
    src/interfaces/ILogBackend.h
    src/interfaces/IStrategy.h
    src/interfaces/IStrategyListener.h
    src/interfaces/IWorker.h
    src/log/ConsoleLog.h
    src/log/FileLog.h
    src/log/Log.h
    src/net/Client.h
    src/net/Job.h
    src/net/JobResult.h
    src/net/Network.h
    src/net/strategies/DonateStrategy.h
    src/net/strategies/FailoverStrategy.h
    src/net/strategies/SinglePoolStrategy.h
    src/net/SubmitResult.h
    src/net/Url.h
    src/Options.h
    src/Platform.h
    src/Summary.h
    src/version.h
    src/workers/CudaWorker.h
    src/workers/GpuThread.h
    src/workers/Handle.h
    src/workers/Hashrate.h
    src/workers/Workers.h
   )

set(HEADERS_CRYPTO
    src/crypto/c_blake256.h
    src/crypto/c_groestl.h
    src/crypto/c_jh.h
    src/crypto/c_keccak.h
    src/crypto/c_skein.h
    src/crypto/CryptoNight.h
    src/crypto/CryptoNight_p.h
    src/crypto/CryptoNight_test.h
    src/crypto/groestl_tables.h
    src/crypto/hash.h
    src/crypto/skein_port.h
   )

set(SOURCES
    src/App.cpp
    src/Console.cpp
    src/Cpu_stub.cpp
    src/log/ConsoleLog.cpp
    src/log/FileLog.cpp
    src/log/Log.cpp
    src/net/Client.cpp
    src/net/Job.cpp
    src/net/Network.cpp
    src/net/strategies/DonateStrategy.cpp
    src/net/strategies/FailoverStrategy.cpp
    src/net/strategies/SinglePoolStrategy.cpp
    src/net/Url.cpp
    src/nvidia/Health.h
    src/nvidia/NvmlApi.h
    src/Options.cpp
    src/Platform.cpp
    src/Summary.cpp
    src/workers/CudaWorker.cpp
    src/workers/GpuThread.cpp
    src/workers/Handle.cpp
    src/workers/Hashrate.cpp
    src/workers/Workers.cpp
    src/xmrig.cpp
   )

set(SOURCES_CRYPTO
    src/crypto/c_keccak.c
    src/crypto/c_groestl.c
    src/crypto/c_blake256.c
    src/crypto/c_jh.c
    src/crypto/c_skein.c
    src/crypto/soft_aes.c
    src/crypto/soft_aes.c
    src/crypto/CryptoNight.cpp
   )

if (WIN32)
    set(SOURCES_OS
        res/app.rc
        src/App_win.cpp
        src/Cpu_win.cpp
        src/Platform_win.cpp
        )

    add_definitions(/DWIN32)
    set(EXTRA_LIBS ws2_32 psapi iphlpapi userenv)
elseif (APPLE)
    set(SOURCES_OS
        src/App_unix.cpp
        src/Cpu_mac.cpp
        src/Platform_mac.cpp
        )
else()
    set(SOURCES_OS
        src/App_unix.cpp
        src/Cpu_unix.cpp
        src/Platform_unix.cpp
        )

    set(EXTRA_LIBS pthread)
endif()

add_definitions(/DXMRIG_NVIDIA_PROJECT)
add_definitions(/DUNICODE)
add_definitions(/DXMRIG_NO_LIBCPUID)
add_definitions(/D__STDC_FORMAT_MACROS)
#add_definitions(/DAPP_DEBUG)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake")

find_package(UV REQUIRED)

if ("${CMAKE_BUILD_TYPE}" STREQUAL "")
    set(CMAKE_BUILD_TYPE Release)
endif()

include(cmake/flags.cmake)
include(cmake/CUDA.cmake)

find_package(NVML)
if (NVML_FOUND)
    include_directories(${NVML_INCLUDE_DIR})

    set(SOURCES_NVML src/nvidia/NvmlApi.cpp)
else()
    add_definitions(/DXMRIG_NO_NVML)

    set(SOURCES_NVML src/nvidia/NvmlApi_stub.cpp)
endif()

CHECK_INCLUDE_FILE (syslog.h HAVE_SYSLOG_H)
if (HAVE_SYSLOG_H)
    add_definitions(/DHAVE_SYSLOG_H)
    set(SOURCES_SYSLOG src/log/SysLog.h src/log/SysLog.cpp)
endif()

if (NOT WITH_AEON)
    add_definitions(/DXMRIG_NO_AEON)
endif()

include_directories(src)
include_directories(src/3rdparty)
include_directories(src/3rdparty/jansson)
include_directories(${UV_INCLUDE_DIR})

add_subdirectory(src/3rdparty/jansson)

add_executable(${PROJECT_NAME} ${HEADERS} ${SOURCES} ${SOURCES_OS} ${SOURCES_NVML} ${HEADERS_CRYPTO} ${SOURCES_CRYPTO} ${SOURCES_SYSLOG})
target_link_libraries(${PROJECT_NAME} xmrig-cuda jansson ${UV_LIBRARIES} ${LIBS} ${EXTRA_LIBS} ${CPUID_LIB})
