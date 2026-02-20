# PRD Example 10: Unit Converter CLI

> **Difficulty:** Beginner | **Project Type:** CLI Tool | **Time:** 2-3 hours

---

## Overview

| | |
|---|---|
| **What** | A command-line tool for converting between common units |
| **Who** | Anyone who needs quick unit conversions without leaving the terminal |
| **Why** | Provides instant unit conversion without searching online or using apps |

---

## Core Features (MVP)

1. **Convert Units:** `convert 100 km to miles` → Shows conversion result
2. **List Categories:** `convert list` → Shows supported unit categories
3. **List Units:** `convert list length` → Shows all units in a category
4. **Reverse Convert:** `convert 62.14 miles to km` → Works both directions

---

## Non-Goals

**Will NOT build:**
- Currency conversion (requires live rates)
- Scientific units (only common everyday units)
- Custom unit definitions
- Conversion history
- Batch conversions from file
- GUI or web interface
- Unit abbreviation guessing

**Will NOT use:**
- External APIs (all conversions are hardcoded)
- Database
- Configuration files
- Third-party unit libraries

---

## Technical Constraints

| | |
|---|---|
| **Language** | Python 3.11+ |
| **Framework** | None (stdlib only) |
| **Dependencies** | None — stdlib only |
| **Categories** | Length, Weight, Temperature, Volume, Time |
| **Testing** | pytest, minimum 5 tests |
| **Code Style** | Black formatter, type hints |

---

## Success Criteria

- [ ] `convert 100 km to miles` returns correct value (62.14)
- [ ] `convert 32 fahrenheit to celsius` returns 0
- [ ] `convert list` shows all 5 categories
- [ ] `convert list length` shows km, miles, meters, feet, inches, cm
- [ ] Invalid unit shows helpful error message
- [ ] All tests pass

---

## Implementation Phases

### Phase 1: Length & Weight Conversions
**Goal:** Implement basic conversions for two categories

**Tasks:**
1. Create `convert.py` with argument parsing
2. Define conversion factors for length (km, miles, m, ft, in, cm)
3. Define conversion factors for weight (kg, lb, oz, g)
4. Implement natural language parsing ("100 km to miles")

**Verification:**
```bash
python convert.py 100 km to miles
# Output: 100 km = 62.14 miles

python convert.py 5 kg to lb
# Output: 5 kg = 11.02 lb

python convert.py 1 mile to meters
# Output: 1 mile = 1609.34 meters
```

**Deliverables:** `convert.py` with length and weight

---

### Phase 2: Temperature, Volume, Time
**Goal:** Add remaining unit categories

**Tasks:**
1. Add temperature conversions (C, F, K) — formula-based, not ratio
2. Add volume conversions (L, gal, qt, pt, cup, ml)
3. Add time conversions (sec, min, hr, day, week)
4. Handle special case for temperature formulas

**Verification:**
```bash
python convert.py 100 celsius to fahrenheit
# Output: 100 celsius = 212 fahrenheit

python convert.py 1 gallon to liters
# Output: 1 gallon = 3.79 liters

python convert.py 24 hours to minutes
# Output: 24 hours = 1440 minutes
```

**Deliverables:** All 5 categories working

---

### Phase 3: List Commands & Testing
**Goal:** Add help commands and tests

**Tasks:**
1. Implement `list` command for categories
2. Implement `list <category>` for units in category
3. Add helpful error messages for invalid units
4. Write pytest tests for all conversions

**Verification:**
```bash
python convert.py list
# Output:
# Available categories:
# - length: km, miles, m, ft, in, cm
# - weight: kg, lb, oz, g
# - temperature: celsius, fahrenheit, kelvin
# - volume: L, gal, qt, pt, cup, ml
# - time: sec, min, hr, day, week

python convert.py 100 foo to bar
# Output: "Unknown unit 'foo'. Use 'convert list' to see available units."

pytest
# All tests pass
```

**Deliverables:** Complete CLI with tests

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Round output to 2 decimal places | Before adding new categories | Add currency conversion |
| Show units in output ("62.14 miles") | Before adding new units | Use external APIs |
| Handle case-insensitively | Before changing output format | Add scientific units |
| Validate both units exist | | Store conversion history |
