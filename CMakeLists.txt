
cmake_minimum_required(VERSION 3.5)
project(neuro_game_sdk_websocketpp)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

include(FetchContent)

set(FETCHCONTENT_QUIET FALSE)

add_library(nlohmann_json INTERFACE)
target_include_directories(nlohmann_json INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/include")

find_package(Boost QUIET COMPONENTS system)
if(NOT Boost_FOUND)
    set(BOOST_INCLUDE_LIBRARIES system asio)
    set(BOOST_ENABLE_CMAKE ON)
    FetchContent_Declare(
            Boost
            URL https://github.com/boostorg/boost/releases/download/boost-1.86.0/boost-1.86.0-cmake.tar.xz
    )
    FetchContent_MakeAvailable(Boost)
endif()

if(TARGET Boost::system)
    set(NEURO_BOOST_SYSTEM_TARGET Boost::system)
elseif(TARGET Boost::boost)
    set(NEURO_BOOST_SYSTEM_TARGET Boost::boost)
else()
    add_library(Boost::boost INTERFACE IMPORTED)
    target_include_directories(Boost::boost INTERFACE ${Boost_INCLUDE_DIRS})
    set(NEURO_BOOST_SYSTEM_TARGET Boost::boost)
endif()

if(TARGET Boost::asio)
    set(NEURO_BOOST_ASIO_TARGET Boost::asio)
else()
    set(NEURO_BOOST_ASIO_TARGET ${NEURO_BOOST_SYSTEM_TARGET})
endif()

if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/websocketpp/websocketpp")
    message(FATAL_ERROR "websocketpp submodule is missing. Run: git submodule update --init --recursive")
endif()

add_library(websocketpp INTERFACE)
target_include_directories(websocketpp INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/websocketpp")

add_executable(NeuroRPS Test/RockPaperScissors.cpp)
target_link_libraries(NeuroRPS PRIVATE nlohmann_json ${NEURO_BOOST_SYSTEM_TARGET} ${NEURO_BOOST_ASIO_TARGET} websocketpp)
