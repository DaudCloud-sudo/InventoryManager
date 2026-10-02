# Inventory Manager

A command-line stock-keeping tool written in C++17: add products, adjust quantities, find items by SKU, report low stock, and persist everything to a CSV file.

**Status:** in development. Milestone 0 (project skeleton and build) complete.

## Build

```
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build
```

Without CMake:

```
g++ -Wall -Wextra -Wpedantic -std=c++17 -Iinclude src/*.cpp -o inventory
```

## Test

```
ctest --test-dir build --output-on-failure
```

## Layout

| Path | Purpose |
|---|---|
| `include/` | Header files: declarations shared between source files |
| `src/` | Implementation files and the program entry point |
| `tests/` | Automated tests that call the logic directly |
| `data/` | The CSV file the program loads and saves |
| `docs/` | Design notes and complexity analysis |
| `SPEC.md` | The full requirements this project is built against |
