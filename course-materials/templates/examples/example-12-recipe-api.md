# PRD Example 12: Recipe API with Search

> **Difficulty:** Intermediate | **Project Type:** REST API | **Time:** 6-8 hours

---

## Overview

| | |
|---|---|
| **What** | A REST API for storing and searching recipes with ingredients |
| **Who** | Developers building recipe apps or meal planning tools |
| **Why** | Provides a simple backend for recipe management with search capabilities |

---

## Core Features (MVP)

1. **Create Recipe:** `POST /recipes` → Create recipe with name, ingredients, instructions
2. **Get Recipe:** `GET /recipes/{id}` → Retrieve single recipe
3. **Search Recipes:** `GET /recipes?ingredient=chicken` → Find recipes by ingredient
4. **List All:** `GET /recipes` → List all recipes with pagination

---

## Non-Goals

**Will NOT build:**
- User authentication or accounts
- Recipe ratings or reviews
- Image upload or storage
- Nutritional information calculation
- Meal planning features
- Shopping list generation
- Recipe sharing or social features
- Complex search (full-text, fuzzy matching)

**Will NOT use:**
- Database (use JSON file for storage)
- External APIs
- Authentication libraries
- Full-text search engines (Elasticsearch, etc.)

---

## Technical Constraints

| | |
|---|---|
| **Language** | Python 3.11+ |
| **Framework** | FastAPI |
| **Dependencies** | fastapi, uvicorn, pydantic |
| **Data Storage** | JSON file at `./data/recipes.json` |
| **Testing** | pytest, httpx for API tests |
| **Code Style** | Black formatter, type hints, Pydantic models |

---

## Success Criteria

- [ ] `POST /recipes` creates recipe with validation
- [ ] `GET /recipes/{id}` returns recipe or 404
- [ ] `GET /recipes?ingredient=X` filters by ingredient (case-insensitive)
- [ ] `GET /recipes` supports pagination (?page=1&limit=10)
- [ ] All endpoints return proper JSON responses
- [ ] API docs available at `/docs` (Swagger UI)
- [ ] All tests pass

---

## Implementation Phases

### Phase 1: Project Setup & Data Model
**Goal:** Set up FastAPI project and define models

**Tasks:**
1. Create project structure with `main.py`, `models.py`, `data/`
2. Define Pydantic models for Recipe (name, ingredients, instructions, id)
3. Implement JSON file read/write utilities
4. Set up FastAPI app with CORS

**Verification:**
```bash
uvicorn main:app --reload
# Server starts on http://localhost:8000

curl http://localhost:8000/docs
# Swagger UI loads
```

**Deliverables:** Project structure, models, JSON utilities

---

### Phase 2: CRUD Endpoints
**Goal:** Implement create, read, list endpoints

**Tasks:**
1. Implement `POST /recipes` with validation
2. Implement `GET /recipes/{id}` with 404 handling
3. Implement `GET /recipes` with pagination
4. Auto-generate UUIDs for new recipes

**Verification:**
```bash
# Create recipe
curl -X POST http://localhost:8000/recipes \
  -H "Content-Type: application/json" \
  -d '{"name": "Pasta", "ingredients": ["pasta", "tomato", "garlic"], "instructions": "Boil pasta..."}'
# Returns: {"id": "abc123", "name": "Pasta", ...}

# Get recipe
curl http://localhost:8000/recipes/abc123
# Returns recipe JSON

# List with pagination
curl "http://localhost:8000/recipes?page=1&limit=5"
# Returns: {"items": [...], "total": 10, "page": 1}
```

**Deliverables:** Working CRUD endpoints

---

### Phase 3: Search & Testing
**Goal:** Add ingredient search and tests

**Tasks:**
1. Implement `GET /recipes?ingredient=X` filter
2. Support multiple ingredients (?ingredient=chicken&ingredient=rice)
3. Write pytest tests for all endpoints
4. Test edge cases (empty results, invalid IDs)

**Verification:**
```bash
# Search by ingredient
curl "http://localhost:8000/recipes?ingredient=chicken"
# Returns recipes containing chicken

# Multiple ingredients
curl "http://localhost:8000/recipes?ingredient=chicken&ingredient=rice"
# Returns recipes with both ingredients

pytest
# All tests pass
```

**Deliverables:** Complete API with tests

---

## API Response Format

```json
// Single recipe
{
  "id": "abc123",
  "name": "Chicken Stir Fry",
  "ingredients": ["chicken", "vegetables", "soy sauce"],
  "instructions": "1. Cut chicken... 2. Heat pan...",
  "created_at": "2025-02-12T10:00:00Z"
}

// List response
{
  "items": [...],
  "total": 50,
  "page": 1,
  "limit": 10,
  "pages": 5
}
```

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Validate request body with Pydantic | Before adding new endpoints | Use database (JSON only) |
| Return proper HTTP status codes | Before changing response format | Add authentication |
| Include CORS headers | Before adding new query params | Store sensitive data |
| Document endpoints in OpenAPI | | Delete recipes (read-only after create) |
