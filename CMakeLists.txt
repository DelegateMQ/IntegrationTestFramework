# Example CMake command line to create project build files:
#
# cmake -B build . -DENABLE_IT=ON
#
# DelegateMQ is fetched from GitHub at the release tag DMQ_GIT_TAG when CMake
# configures. To build against a local DelegateMQ checkout instead, add
# -DFETCHCONTENT_SOURCE_DIR_DELEGATEMQ=<path>. To configure again without
# network access, add -DFETCHCONTENT_FULLY_DISCONNECTED=ON.

# Specify the minimum CMake version required
cmake_minimum_required(VERSION 3.16)

# Project name and language (C or C++)
project(IntegrationTestFramework VERSION 1.1.0 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Ensure all libraries use the dynamically linked runtime (/MDd for Debug, /MD for Release)
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL")

# Force GoogleTest to use the dynamic runtime libraries
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

# DelegateMQ build options, set before DelegateMQ is added
set(DMQ_ASSERTS "OFF")                        # ON for assert faults
set(DMQ_ALLOCATOR "OFF")                      # ON for fixed-block allocator
set(DMQ_LOG "OFF")                            # ON for spdlog output
set(DMQ_UTIL "ON")                            # ON for delegate utility classes
set(DMQ_THREAD "DMQ_THREAD_STDLIB")           # Set thread support library or none
set(DMQ_SERIALIZE "DMQ_SERIALIZE_SERIALIZE")  # Set serialization support library or none
set(DMQ_TRANSPORT "DMQ_TRANSPORT_NONE")       # Set transport support library or none

# DelegateMQ release to build against
set(DMQ_GIT_TAG "v3.0.0" CACHE STRING "DelegateMQ release tag (or branch) to fetch")

include(FetchContent)
FetchContent_Declare(
    delegatemq
    GIT_REPOSITORY https://github.com/DelegateMQ/DelegateMQ.git
    GIT_TAG        ${DMQ_GIT_TAG}
    GIT_SHALLOW    TRUE
)
FetchContent_MakeAvailable(delegatemq)

# Enable integration test build
set (ENABLE_IT "ON")

# Collect all .cpp and *.h source files in the current directory
file(GLOB SOURCES "*.cpp" "*.h")

# Platform-specific linker flags
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    # GCC/Clang: Use --no-as-needed to prevent the removal of unused libraries
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--no-as-needed")
endif()

# Define the IT_ENABLE macro when building integration tests
if (ENABLE_IT)
    add_compile_definitions(IT_ENABLE)
endif()

# Add subdirectories to include path
include_directories(
    ${CMAKE_SOURCE_DIR}/Logger/src
    ${CMAKE_SOURCE_DIR}/Port/src
)

# Add subdirectories to include path if building integration tests
if (ENABLE_IT)
    include_directories(
        ${CMAKE_SOURCE_DIR}/Logger/it
        ${CMAKE_SOURCE_DIR}/IntegrationTest
        ${CMAKE_SOURCE_DIR}/GoogleTest/googletest/include
    )
endif()

# Add an executable target
add_executable(IntegrationTestFrameworkApp ${SOURCES})

# Add subdirectories to build (product related code)
add_subdirectory(Logger/src)
add_subdirectory(Port/src)

# Add subdirectories to build (integration test related code)
if (ENABLE_IT)
    add_subdirectory(Logger/it)
    add_subdirectory(IntegrationTest)
    add_subdirectory(GoogleTest)
endif()

target_link_libraries(IntegrationTestFrameworkApp PRIVATE
    LoggerLib
    PortLib
    DelegateMQ::DelegateMQ
)

if (ENABLE_IT)
    target_link_libraries(IntegrationTestFrameworkApp PRIVATE
        Logger_ITLib
        IntegrationTestLib
        gtest
        gtest_main
    )
endif()
