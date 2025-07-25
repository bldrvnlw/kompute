cmake_minimum_required(VERSION 3.20)

set(LOGGER_SOURCES Logger.cpp)

add_library(kp_logger ${LOGGER_SOURCES})

# Define log levels in code
add_compile_definitions(KOMPUTE_LOG_LEVEL_TRACE=0)
add_compile_definitions(KOMPUTE_LOG_LEVEL_DEBUG=1)
add_compile_definitions(KOMPUTE_LOG_LEVEL_INFO=2)
add_compile_definitions(KOMPUTE_LOG_LEVEL_WARN=3)
add_compile_definitions(KOMPUTE_LOG_LEVEL_ERROR=4)
add_compile_definitions(KOMPUTE_LOG_LEVEL_CRITICAL=5)
add_compile_definitions(KOMPUTE_LOG_LEVEL_OFF=6)

if(KOMPUTE_OPT_BUILD_PYTHON AND KOMPUTE_OPT_USE_SPDLOG)
    message(FATAL_ERROR "'KOMPUTE_OPT_BUILD_PYTHON' is incompatible with 'KOMPUTE_OPT_USE_SPDLOG'. To continue set either one option to 'OFF'.")
endif()

if(KOMPUTE_OPT_ANDROID_BUILD AND KOMPUTE_OPT_USE_SPDLOG)
    message(FATAL_ERROR "'KOMPUTE_OPT_ANDROID_BUILD' is incompatible with 'KOMPUTE_OPT_USE_SPDLOG'. To continue set either one option to 'OFF'.")
endif()

if(${KOMPUTE_OPT_LOG_LEVEL} STREQUAL "Trace")
    set(KOMPUTE_OPT_LOG_LEVEL TRACE)
    message(STATUS "Using log level Trace")
elseif(${KOMPUTE_OPT_LOG_LEVEL} STREQUAL "Debug")
    set(KOMPUTE_OPT_LOG_LEVEL DEBUG)
    message(STATUS "Using log level Debug")
elseif(${KOMPUTE_OPT_LOG_LEVEL} STREQUAL "Info")
    set(KOMPUTE_OPT_LOG_LEVEL INFO)
    message(STATUS "Using log level Info")
elseif(${KOMPUTE_OPT_LOG_LEVEL} STREQUAL "Warn")
    set(KOMPUTE_OPT_LOG_LEVEL WARN)
    message(STATUS "Using log level Warn")
elseif(${KOMPUTE_OPT_LOG_LEVEL} STREQUAL "Error")
    set(KOMPUTE_OPT_LOG_LEVEL ERROR)
    message(STATUS "Using log level Error")
elseif(${KOMPUTE_OPT_LOG_LEVEL} STREQUAL "Critical")
    set(KOMPUTE_OPT_LOG_LEVEL CRITICAL)
    message(STATUS "Using log level Critical")
elseif(${KOMPUTE_OPT_LOG_LEVEL} STREQUAL "Off")
    set(KOMPUTE_OPT_LOG_LEVEL OFF)
    message(STATUS "Using log level Off")
elseif(${KOMPUTE_OPT_LOG_LEVEL} STREQUAL "Default")
    set(KOMPUTE_OPT_LOG_LEVEL $<IF:$<CONFIG:Debug>,DEBUG,INFO>)
    message(STATUS "Setting KOMPUTE_OPT_LOG_LEVEL to according to the build type")
else()
    message(FATAL_ERROR "Log level '${KOMPUTE_OPT_LOG_LEVEL}' unknown, use -DKOMPUTE_OPT_LOG_LEVEL={Trace, Debug, Info, Warn, Error, Critical, Off, Default} to set it to a correct value.")
endif()

# Always make sure we define the Kompute log level independent of the Spdlog log level
target_compile_definitions(kp_logger INTERFACE KOMPUTE_OPT_ACTIVE_LOG_LEVEL=KOMPUTE_LOG_LEVEL_${KOMPUTE_OPT_LOG_LEVEL})

# Link depending on how the logger should be setup
if(NOT KOMPUTE_OPT_LOG_LEVEL_DISABLED)
    if(KOMPUTE_OPT_USE_SPDLOG)
        target_link_libraries(kp_logger PUBLIC spdlog::spdlog)
        target_compile_definitions(spdlog INTERFACE SPDLOG_ACTIVE_LEVEL=SPDLOG_LEVEL_${KOMPUTE_OPT_LOG_LEVEL})
        target_compile_definitions(kp_logger INTERFACE SPDLOG_ACTIVE_LEVEL=SPDLOG_LEVEL_${KOMPUTE_OPT_LOG_LEVEL})
        message(STATUS "setting SPDLOG_ACTIVE_LEVEL to SPDLOG_LEVEL_${KOMPUTE_OPT_LOG_LEVEL}")

        if(KOMPUTE_OPT_SPDLOG_ASYNC_MODE)
            target_compile_definitions(kp_logger INTERFACE KOMPUTE_SPDLOG_ASYNC_LOGGING=1)
        endif()
    else()
        target_link_libraries(kp_logger PUBLIC fmt::fmt)
    endif()
endif()

if(KOMPUTE_OPT_BUILD_PYTHON)
    target_link_libraries(kp_logger PUBLIC pybind11::headers)
endif()

# Nothing will be installed, this will only export kp_logger to komputeTargets
add_library(kompute::logger ALIAS kp_logger)
if(KOMPUTE_OPT_INSTALL)
    install(TARGETS kp_logger
        EXPORT komputeTargets)
endif()