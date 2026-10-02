# Inventory Manager — Project Spec

**Type:** command-line application, C++17, single user, file-backed
**Start when:** you have finished Part X (STL) of the guide
**Budget:** 10–14 one-hour sessions
**This file is the contract.** It says *what* the program must do and how you'll know it's right. It deliberately does not say how to write it — that's your half.

---

## 1. What you're building, and why this one

A small stock-keeping tool for a shop: add products, adjust quantities as stock arrives and sells, find things fast, report what's running low, and keep everything on disk between runs.

It's the right first real project because it forces five things a practice exercise never does:

1. **A data-structure choice with consequences** — the wrong container makes one requirement slow or one requirement awkward.
2. **Persistence** — data must survive the program closing, and survive a damaged file.
3. **Hostile input** — every prompt will be given garbage, and nothing may crash.
4. **Separation of logic from I/O** — so the logic can be tested without a human typing.
5. **A real build, real tests, real version history.**

---

## 2. What you need before starting

From the guide: Parts III–X (structure, types, operators, control flow, functions, memory model, strings, STL).

**Three things the guide does not cover — learn these first, one short session:**

| Topic | Header | What you need from it |
|---|---|---|
| File streams | `<fstream>` | Opening a file for reading and for writing; checking whether the open succeeded |
| Reading whole lines | `<string>` (`std::getline`) | Reading a line including its spaces; why mixing `>>` and `getline` leaves a stray newline behind |
| Splitting a line | `<sstream>` (`std::stringstream`) | Turning one text line into separate fields using a delimiter |

Also `struct`: a `class` whose members are public by default. Use it for plain data records.

---

## 3. The data

### 3.1 One item

| Field | Type to use | Rules |
|---|---|---|
| SKU | text | The unique key. 1–16 characters, only `A–Z`, `0–9`, and `-`. Stored uppercase. |
| Name | text | 1–40 characters. May contain spaces. **May not contain a comma** (see file format). |
| Category | text | 1–20 characters. No commas. |
| Quantity | whole number | 0 or more. Never negative. |
| Unit price | **whole number of cents** | 0 or more. `8999` means 89.99. |
| Reorder level | whole number | 0 or more. Item is "low" when quantity ≤ reorder level. |

**Why cents and not a `double`:** `0.1 + 0.2` is not `0.3` in floating point. Money stored as a `double` drifts, and a total over thousands of items will be wrong by pennies. Every real payments system stores the smallest currency unit as an integer. Do the same.

**Think about overflow now:** quantity × price, summed across the whole inventory. Decide which integer type holds that total safely, and write down why.

### 3.2 The collection — your first design decision

You must meet all of these at once:

| Operation | Required cost |
|---|---|
| Find an item by SKU | O(1) average |
| Add an item | O(1) average |
| List everything in SKU order | O(n log n) or better |
| Remove an item | you choose — but you must state it and defend it |

No single STL container gives you all four for free. Before writing any code, write half a page in your design notes: which container(s), what each one costs for each row above, and what you're trading away. **This write-up is a graded part of the project.** There is more than one defensible answer; there are also a couple of tempting wrong ones.

---

## 4. What the program does

### 4.1 Commands

| # | Command | Behaviour |
|---|---|---|
| 1 | Add item | Prompt for every field. Reject a SKU that already exists. |
| 2 | List all | Table of every item, sorted by SKU. Shows a `LOW` marker where quantity ≤ reorder level. |
| 3 | Find by SKU | Show one item, or say it doesn't exist. Lookup is case-insensitive (`kb-104` finds `KB-104`). |
| 4 | Search by name | Show every item whose name contains the typed text, case-insensitive. `key` matches `Mechanical Keyboard`. |
| 5 | Adjust stock | Take a SKU and a signed change. `+10` receives stock, `-3` sells it. Reject any change that would take quantity below 0. |
| 6 | Change price | Take a SKU and a new price. |
| 7 | Remove item | Take a SKU. Ask for confirmation before removing. |
| 8 | Low-stock report | Every item where quantity ≤ reorder level, sorted by SKU. |
| 9 | Total value | Sum of quantity × unit price across everything, printed as currency. |
| S | Save | Write to disk now. |
| 0 | Quit | If there are unsaved changes, ask whether to save first. |

### 4.2 What a session looks like

```
=== Inventory Manager ===   12 items loaded from data/inventory.csv

 1) Add item           6) Change price
 2) List all           7) Remove item
 3) Find by SKU        8) Low-stock report
 4) Search by name     9) Total value
 5) Adjust stock       S) Save      0) Quit
> 5
SKU: kb-104
Change (+ receive / - sell): -3
KB-104  Mechanical Keyboard    qty 7 -> 4    LOW (reorder at 5)

> 5
SKU: kb-104
Change (+ receive / - sell): -9
Rejected: only 4 in stock. Nothing changed.

> 9
Total inventory value: 14,382.50
```

Exact wording and layout are yours. The behaviour is not.

### 4.3 Price entry

The user types a normal decimal price; you store cents.

| Typed | Stored | |
|---|---|---|
| `12` | 1200 | |
| `12.5` | 1250 | |
| `12.50` | 1250 | |
| `0.99` | 99 | |
| `12.345` | — | reject: more than two decimal places |
| `-1` | — | reject: negative |
| `abc`, `12.x`, empty | — | reject |

**Do not convert through a `double` to do this.** Work on the text. This is a proper logic problem — do the pseudocode and a hand trace for each row above before you write it.

---

## 5. Persistence

### 5.1 File format

Plain CSV at `data/inventory.csv`. First line is a header. One item per line. Fields in this order:

```
sku,name,category,quantity,price_cents,reorder_level
KB-104,Mechanical Keyboard,Peripherals,7,8999,5
MS-220,Wireless Mouse,Peripherals,31,2499,10
CB-USB3,USB-C Cable 2m,Cables,0,799,25
```

### 5.2 Loading — must never crash, whatever the file contains

| Situation | Required behaviour |
|---|---|
| File doesn't exist | Start with an empty inventory. Say so. Not an error. |
| File exists but is empty | Start empty. |
| Header line missing or wrong | Refuse to load, explain, start empty. Do not guess. |
| A line has the wrong number of fields | Skip that line, print a warning **with the line number**, keep loading the rest. |
| A number field isn't a number, or is negative | Same: skip, warn with line number, continue. |
| Same SKU appears twice | Keep the first, skip the second, warn with line number. |
| Blank line | Ignore silently. |

At the end of loading, report: how many loaded, how many skipped.

### 5.3 Saving

Rewrites the whole file, sorted by SKU, header first. After a save followed by a restart, **List all** must print exactly what it printed before.

---

## 6. Rules for the code itself

These are what separate a project someone would hire you for from a tutorial exercise.

**Structure**

- **Logic and I/O are separate.** The code that adds, finds, adjusts and removes items must not contain a single `cin` or `cout`. It takes values in and reports results out. Only the menu layer talks to the user. This is the rule that makes section 8 possible — if you can't test it without typing, the structure is wrong.
- Operations that can fail report **why** they failed, not just true/false. "Not found", "duplicate SKU", "insufficient stock" and "invalid value" are different outcomes and the caller must be able to tell them apart.
- No global variables.
- No function longer than about 40 lines. If it is, it's doing two jobs.

**Modern C++**

- No raw arrays. No `new` or `delete`. No C-style strings.
- Pass anything bigger than a number by `const` reference unless you mean to change it.
- Anything that doesn't change is `const`.

**Build**

- Compiles with **zero warnings** under `-Wall -Wextra -Wpedantic -std=c++17`.

---

## 7. Project layout and build

```
InventoryManager/
├── CMakeLists.txt
├── README.md              <- you write this in milestone 7
├── SPEC.md                <- this file
├── .gitignore             <- must ignore build/
├── include/               <- your .h files
├── src/                   <- your .cpp files, including main.cpp
├── tests/
│   └── test_inventory.cpp
├── data/
│   └── inventory.csv
└── docs/
    └── design-notes.md    <- container decision, complexity table, what you'd change
```

Build tooling isn't what this project is teaching, so here is the `CMakeLists.txt`. Adjust the source file names to match yours.

```cmake
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
```

```
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build
ctest --test-dir build --output-on-failure
```

Build in **Debug**: a Release build switches `assert` off, and your tests would pass without checking anything.

If CMake isn't installed yet, this works in the meantime:

```
g++ -Wall -Wextra -Wpedantic -std=c++17 -Iinclude src/*.cpp -o inventory
```

---

## 8. Tests

`tests/test_inventory.cpp` is a second program with its own `main`. It calls your logic directly and checks results with `assert` from `<cassert>`. No test framework needed for version 1.

Every row in section 9 marked **T** must have a test. Write the test for a behaviour *before* or *alongside* the code for it, not at the end.

---

## 9. Acceptance checklist

The project is finished when every row passes. **T** = must be covered by an automated test. **M** = check by hand.

| ID | Do this | Expect | |
|---|---|---|---|
| A1 | Add a valid item, then find it by SKU | Found, every field matches | T |
| A2 | Add an item whose SKU already exists | Rejected as duplicate; item count unchanged; original untouched | T |
| A3 | Add with SKU `kb-104`, find with `KB-104` | Found | T |
| A4 | SKU `KB 104`, `KB_104`, empty, 17 characters | Each rejected | T |
| A5 | Adjust stock by `+10` on quantity 4 | Quantity 14 | T |
| A6 | Adjust stock by `-9` on quantity 4 | Rejected as insufficient; quantity still 4 | T |
| A7 | Adjust stock by `-4` on quantity 4 | Quantity 0; item still exists | T |
| A8 | Any operation on a SKU that doesn't exist | Reported as not found; nothing changes | T |
| A9 | Every row of the price table in 4.3 | Matches exactly | T |
| A10 | Remove an item, then find it | Not found; count dropped by one; every other item still findable | T |
| A11 | Search `key` with `Mechanical Keyboard` and `Keycap Set` present | Both returned | T |
| A12 | Low-stock report | Exactly the items with quantity ≤ reorder level, in SKU order | T |
| A13 | Total value for a known set of three items | Equals the figure you calculated by hand | T |
| A14 | Every command on an empty inventory | Sensible message, total `0.00`, no crash | T |
| A15 | Save, load into a fresh inventory | Identical contents | T |
| A16 | Load a file with one bad line among good ones | Good lines loaded, bad one skipped, warning names the line number | T |
| A17 | Load when the file doesn't exist | Empty inventory, no crash | T |
| A18 | Type `abc` at a number prompt | Re-prompts once per bad entry; no crash; no infinite loop | M |
| A19 | Quit with unsaved changes | Asks before discarding | M |
| A20 | Build | Zero warnings | M |

---

## 10. Milestones

One commit at the end of each, minimum. For every milestone: **pseudocode and a hand trace in `docs/design-notes.md` first, code second.**

| | Milestone | Done when | Sessions |
|---|---|---|---|
| M0 | Setup | Folder layout exists, `git init`, an empty `main` builds and runs, first commit | 1 |
| M1 | Design | Container decision written up with its complexity table; item record defined | 1 |
| M2 | Core in memory | Add, find, list working, called from a throwaway `main`. A1–A3 pass. | 2 |
| M3 | Change and remove | Adjust stock, change price, remove. A5–A8, A10 pass. | 2 |
| M4 | Validation | SKU rules and price parsing. A4, A9 pass. | 2 |
| M5 | Persistence | Save and load, including every row of 5.2. A15–A17 pass. | 2 |
| M6 | Reports and menu | Search, low-stock, total value, the full menu loop. A11–A14, A18–A19 pass. | 2 |
| M7 | Finish | README written, zero warnings, complexity table complete, everything in section 9 ticked | 1 |

---

## 11. Complexity table — fill this in

Goes in `docs/design-notes.md`. Every cell, with one line of reasoning each. *n* = number of items.

| Operation | Time: best | Time: average | Time: worst | Extra space |
|---|---|---|---|---|
| Add item | | | | |
| Find by SKU | | | | |
| Search by name | | | | |
| Adjust stock | | | | |
| Remove item | | | | |
| List all (sorted) | | | | |
| Low-stock report | | | | |
| Total value | | | | |
| Load from file | | | | |
| Save to file | | | | |

Where best and worst differ, say what input causes each.

---

## 12. README — what a reviewer looks for

Written last, in your own words:

1. One paragraph: what it is.
2. How to build and run it — commands someone can paste.
3. How to run the tests.
4. **Design decisions:** the container choice and why; why cents; how a corrupt file is handled.
5. **Known limitations** — commas in names, single user, whole file rewritten on every save. Naming your own limits reads as competence, not weakness.
6. What you'd do next.

---

## 13. After version 1 — pick any

- **Undo** the last change. Which structure from the guide is this, exactly?
- **Sort the list** by name, quantity or value, chosen at run time.
- **Crash-safe save:** write to a temporary file, then rename it over the real one, so a crash mid-save can't destroy the data.
- **Allow commas in names** by supporting quoted CSV fields.
- **Python companion:** a script that generates a 100,000-item CSV, runs your program against it and times the load. This is where your Python and your C++ meet, and it gives you a real measured number.
- Replace the hand-rolled tests with Catch2 or GoogleTest.
- GitHub Actions: build and run the tests on every push.

---

## 14. Putting it on a résumé

Don't write the line until it's built. Then fill this in with what's actually true:

> Built a command-line inventory system in C++17 with `<your container choice>` giving O(1) average lookup; CSV persistence that tolerates malformed input; `<N>` automated tests; CMake build. `<One measured number, if you did the Python benchmark.>`

A number you measured is worth more than any adjective. A number you didn't measure doesn't go on the page.

---

## 15. How we work on this

- You write every line. I review, I point at the problem, you fix it.
- Bring the design write-up from M1 before you start M2. That review is the highest-value hour in the project.
- Stuck for 45 minutes on one thing: stop and bring it. That's the stuck protocol, not a failure.
