# Design Notes

Written before the code, milestone by milestone. This file is part of the project, not scratch paper.

## 1. Container decision (Milestone 1)

Which container or containers hold the items, and why.

| Operation | Required | What my choice gives | Why |
|---|---|---|---|
| Find by SKU | O(1) average | | |
| Add item | O(1) average | | |
| List all in SKU order | O(n log n) or better | | |
| Remove item | my choice | | |

What I am trading away:

Alternatives I rejected, and why:

## 2. Integer types

Which type holds quantity, price in cents, and the total inventory value, and the largest value each can reach:

## 3. Pseudocode and hand traces

### Milestone 2: add, find, list

### Milestone 3: adjust stock, change price, remove

### Milestone 4: SKU validation, price parsing

### Milestone 5: save, load

### Milestone 6: search, reports, menu

## 4. Complexity table (Milestone 7)

n = number of items. One line of reasoning per cell.

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

## 5. What I would change next time
