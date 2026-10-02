cmake_minimum_required(VERSION 3.16)
project(InventoryManager LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# All logic lives in a library so the app and the tests share one copy of it.
add_library(inventory_core
    src/inventory.cpp
    src/storage.cpp
)
target_include_directories(inventory_core PUBLIC include)

if(MSVC)
    target_compile_options(inventory_core PRIVATE /W4)
else()
    target_compile_options(inventory_core PRIVATE -Wall -Wextra -Wpedantic)
endif()

add_executable(inventory src/main.cpp)
target_link_libraries(inventory PRIVATE inventory_core)

enable_testing()
add_executable(inventory_tests tests/test_inventory.cpp)
target_link_libraries(inventory_tests PRIVATE inventory_core)
add_test(NAME inventory_tests COMMAND inventory_tests)
