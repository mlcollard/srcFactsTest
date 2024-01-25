# @file CMakeLists.txt
# 
# CMake files for the srcfacts project

# Can be run by any version of CMake >= this
cmake_minimum_required(VERSION 3.28)

# project info
project(srcfacts)

# Use Release build type if not specified
if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

# srcfacts application
add_executable(srcfacts)

# srcfacts sources
target_sources(srcfacts PRIVATE srcfacts.cpp refillContent.cpp)
target_compile_features(srcfacts PRIVATE cxx_std_17)
set_target_properties(srcfacts PROPERTIES
    CXX_STANDARD_REQUIRED ON
    CXX_EXTENSIONS OFF
)

# libarchive dependency
find_package(LibArchive 3 REQUIRED)
target_link_libraries(srcfacts PRIVATE LibArchive::LibArchive)

# Turn on warnings
target_compile_options(srcfacts PRIVATE
     $<$<OR:$<CXX_COMPILER_ID:Clang>,$<CXX_COMPILER_ID:AppleClang>,$<CXX_COMPILER_ID:GNU>>: -Wall>
     $<$<CXX_COMPILER_ID:MSVC>: /W4>
)

# control TRACE: cmake . -DTRACE=ON|OFF
if(DEFINED TRACE)
    message(STATUS "TRACE is ${TRACE}")
    if(TRACE)
        target_compile_definitions(srcfacts PUBLIC TRACE)
    endif()
endif()

# Extract the demo input srcML file into the data directory
set(DATA_DIR "${CMAKE_CURRENT_BINARY_DIR}/data")
file(ARCHIVE_EXTRACT INPUT ${CMAKE_SOURCE_DIR}/demo.xml.zip DESTINATION ${DATA_DIR})

# Demo run command
add_custom_target(run
        COMMENT "Run demo"
        COMMAND $<TARGET_FILE:srcfacts> < ${DATA_DIR}/demo.xml
        DEPENDS srcfacts
        USES_TERMINAL
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
)

# Setup optional bigdata
set(BIGDATA_FILENAME "linux-6.6.xml.gz")
set(BIGDATA_URL "http://131.123.42.38/build/${BIGDATA_FILENAME}")
set(BIGDATA_FILE "${DATA_DIR}/${BIGDATA_FILENAME}")
if(DEFINED BIGDATA AND BIGDATA)
    message(STATUS "Downloading ${BIGDATA_FILENAME}")
    file(DOWNLOAD ${BIGDATA_URL} ${BIGDATA_FILE}
         SHOW_PROGRESS
         EXPECTED_MD5 59ad4bf8847024177d859e242f0e70e8)

    # run bigdata as input
    add_custom_target(run_bigdata
        COMMAND $<TARGET_FILE:srcfacts> < ${BIGDATA_FILE}
        USES_TERMINAL
    )

    # remove downloaded bigdata
    add_custom_target(clean_bigdata
        COMMAND ${CMAKE_COMMAND} -E rm -f ${BIGDATA_FILE}
        COMMAND ${CMAKE_COMMAND} -E echo "Set BIGDATA to OFF or cmake may download and extract it again"
        USES_TERMINAL
    )
endif()
