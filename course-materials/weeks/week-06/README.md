# Week 6: Architecture & Design Patterns — Guiding AI Toward Good Patterns

> "The goal of good architecture is to defer decisions. With AI, it's not about deferring decisions — it's about making them explicit in instruction files and tests so the AI can follow them." — Martin Fowler, adapted

---

## Quick Pulse Check (Start)

Before we begin, rate your current reality (1-5):

| Question | 1 | 2 | 3 | 4 | 5 |
|----------|---|---|---|---|---|
| When AI generates code, how often does it put everything in one file? | Always | Often | Sometimes | Rarely | Never |
| How confident are you explaining why modular code beats spaghetti code? | Not at all | Slightly | Moderately | Very | Teach it |
| Have you ever refactored AI-generated code to be more modular? | Never | Once | A few times | Often | Always |
| Do you currently use a CLAUDE.md or similar to enforce architecture rules? | No | Planning to | Have one (unused) | Using it | Using + iterating |

Keep these answers — we'll revisit them at the end.

---

## Learning Objectives

By the end of this week, you will be able to:

1. **Identify the Spaghetti Problem** — Recognize when AI generates monolithic code and understand why separation of concerns matters
2. **Apply three fundamental patterns** — Repository, Service Layer, and Middleware — to modular, testable code
3. **Use instruction files as architecture guards** — Write CLAUDE.md and similar files that enforce patterns at generation time, not refactoring time
4. **Execute the Refactoring Workflow** — Take messy AI output, identify violations, and systematically refactor using patterns
5. **Understand the Dependency Rule** — Design layers that point inward and never create circular dependencies
6. **Build maintainable APIs with AI** — Generate clean REST endpoints that grow without collapsing into chaos

---

## The Spaghetti Problem: AI's Default Behavior

### What Happens When You Ask AI to Build an API

Ask Claude or Gemini to "build a REST API for event management," and you get something like this:

**The Messy Way (500+ lines in index.js):**

```javascript
// index.js — Everything. All of it. In one file.
const express = require('express');
const sqlite3 = require('sqlite3');
const jwt = require('jsonwebtoken');
const app = express();

app.use(express.json());

const db = new sqlite3.Database(':memory:');

// Database initialization
db.run(`
  CREATE TABLE events (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    date TEXT NOT NULL,
    category TEXT,
    created_by TEXT
  )
`);

db.run(`
  CREATE TABLE rsvps (
    id INTEGER PRIMARY KEY,
    event_id INTEGER,
    user_id TEXT,
    FOREIGN KEY(event_id) REFERENCES events(id)
  )
`);

// Inline authentication
const verifyToken = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });
  try {
    req.user = jwt.verify(token, 'secret-key');
    next();
  } catch (err) {
    res.status(403).json({ error: 'Invalid token' });
  }
};

// Inline validation
const validateEvent = (req, res, next) => {
  const { title, date, category } = req.body;
  if (!title || title.trim().length === 0) {
    return res.status(400).json({ error: 'Title required' });
  }
  if (!date) return res.status(400).json({ error: 'Date required' });
  if (category && !['tech', 'social', 'sports'].includes(category)) {
    return res.status(400).json({ error: 'Invalid category' });
  }
  next();
};

// Routes with business logic, DB queries, validation all mixed together
app.post('/events', verifyToken, validateEvent, (req, res) => {
  const { title, date, category } = req.body;

  db.run(
    'INSERT INTO events (title, date, category, created_by) VALUES (?, ?, ?, ?)',
    [title, date, category || null, req.user.id],
    function(err) {
      if (err) return res.status(500).json({ error: 'DB error' });
      res.json({ id: this.lastID, title, date, category });
    }
  );
});

app.get('/events', (req, res) => {
  const { category, startDate, endDate } = req.query;

  // Complex query building inline
  let query = 'SELECT * FROM events WHERE 1=1';
  const params = [];

  if (category) {
    query += ' AND category = ?';
    params.push(category);
  }
  if (startDate) {
    query += ' AND date >= ?';
    params.push(startDate);
  }
  if (endDate) {
    query += ' AND date <= ?';
    params.push(endDate);
  }

  db.all(query, params, (err, rows) => {
    if (err) return res.status(500).json({ error: 'DB error' });
    res.json(rows);
  });
});

app.post('/events/:id/rsvp', verifyToken, (req, res) => {
  const eventId = req.params.id;
  const userId = req.user.id;

  // Check if already RSVP'd
  db.get(
    'SELECT * FROM rsvps WHERE event_id = ? AND user_id = ?',
    [eventId, userId],
    (err, row) => {
      if (err) return res.status(500).json({ error: 'DB error' });
      if (row) return res.status(400).json({ error: 'Already RSVP\'d' });

      db.run(
        'INSERT INTO rsvps (event_id, user_id) VALUES (?, ?)',
        [eventId, userId],
        function(err) {
          if (err) return res.status(500).json({ error: 'DB error' });
          res.json({ rsvp_id: this.lastID });
        }
      );
    }
  );
});

// ... 100 more lines of this

app.listen(3000, () => console.log('Server running'));
```

**The Problems:**

1. **No separation of concerns** — Routes contain DB logic, validation, auth, and business logic
2. **Impossible to test** — Can't test business logic without running a server and hitting a database
3. **Hard to reuse** — Want to check if a user already RSVP'd? You have to extract it from middleware
4. **Changes cascade** — Need to refactor auth? You're touching every single route
5. **AI keeps doing this** — Because the pattern is in its training data and requires discipline to avoid

### The Clean Way (Using Patterns)

**File structure:**

```
src/
├── routes/
│   ├── events.js       (Route handlers only)
│   └── rsvps.js
├── services/
│   ├── eventService.js (Business logic)
│   └── rsvpService.js
├── repositories/
│   ├── eventRepository.js  (Data access)
│   └── rsvpRepository.js
├── middleware/
│   ├── auth.js         (Authentication)
│   ├── validation.js   (Input validation)
│   └── errorHandler.js
└── index.js            (Setup and bootstrap)
```

**src/repositories/eventRepository.js:**

```javascript
class EventRepository {
  constructor(db) {
    this.db = db;
  }

  create(eventData) {
    return new Promise((resolve, reject) => {
      this.db.run(
        'INSERT INTO events (title, date, category, created_by) VALUES (?, ?, ?, ?)',
        [eventData.title, eventData.date, eventData.category, eventData.createdBy],
        function(err) {
          if (err) reject(err);
          else resolve({ id: this.lastID, ...eventData });
        }
      );
    });
  }

  findById(id) {
    return new Promise((resolve, reject) => {
      this.db.get('SELECT * FROM events WHERE id = ?', [id], (err, row) => {
        if (err) reject(err);
        else resolve(row);
      });
    });
  }

  findByFilters(filters) {
    // Date, category filtering logic isolated here
    let query = 'SELECT * FROM events WHERE 1=1';
    const params = [];

    if (filters.category) {
      query += ' AND category = ?';
      params.push(filters.category);
    }
    if (filters.startDate) {
      query += ' AND date >= ?';
      params.push(filters.startDate);
    }
    if (filters.endDate) {
      query += ' AND date <= ?';
      params.push(filters.endDate);
    }

    return new Promise((resolve, reject) => {
      this.db.all(query, params, (err, rows) => {
        if (err) reject(err);
        else resolve(rows || []);
      });
    });
  }
}

module.exports = EventRepository;
```

**src/services/eventService.js:**

```javascript
class EventService {
  constructor(eventRepository) {
    this.eventRepository = eventRepository;
  }

  async createEvent(eventData) {
    // Business logic: validation, enrichment, etc.
    if (!eventData.title || eventData.title.trim().length === 0) {
      throw new Error('Title is required');
    }

    if (!eventData.date) {
      throw new Error('Date is required');
    }

    const validCategories = ['tech', 'social', 'sports'];
    if (eventData.category && !validCategories.includes(eventData.category)) {
      throw new Error(`Category must be one of: ${validCategories.join(', ')}`);
    }

    // Delegate to repository
    return await this.eventRepository.create(eventData);
  }

  async getEventsByFilters(filters) {
    // Business logic: filtering, sorting
    return await this.eventRepository.findByFilters(filters);
  }

  async getEventById(id) {
    const event = await this.eventRepository.findById(id);
    if (!event) {
      throw new Error('Event not found');
    }
    return event;
  }
}

module.exports = EventService;
```

**src/routes/events.js:**

```javascript
const express = require('express');
const router = express.Router();

module.exports = (eventService) => {
  // Route handlers are now JUST HTTP plumbing

  router.post('/', (req, res, next) => {
    // validateEvent middleware already ran — req.body is clean
    eventService
      .createEvent({
        title: req.body.title,
        date: req.body.date,
        category: req.body.category,
        createdBy: req.user.id, // Set by auth middleware
      })
      .then((event) => res.status(201).json(event))
      .catch((err) => next(err)); // Pass to error handler
  });

  router.get('/', (req, res, next) => {
    eventService
      .getEventsByFilters({
        category: req.query.category,
        startDate: req.query.startDate,
        endDate: req.query.endDate,
      })
      .then((events) => res.json(events))
      .catch((err) => next(err));
  });

  return router;
};
```

**Why this is better:**

- **Test the service in isolation** — No database, no HTTP, just JavaScript
- **Reuse repositories** — Multiple services can use the same data access layer
- **Change auth once** — Update the middleware, not 30 route handlers
- **Routes are readable** — Each handler is 5 lines: validate input, call service, return result, handle error

---

## Three Patterns That Work With AI

### Pattern 1: Repository Pattern

**What it does:** Separates data access logic from business logic. All database queries live in one place.

**Why it matters with AI:** AI loves putting `db.run()` directly in route handlers. The Repository pattern makes it obvious where all data queries belong.

**Template:**

```javascript
// Repository — all data access
class TodoRepository {
  constructor(db) {
    this.db = db;
  }

  async create(task) {
    // All INSERT logic here
  }

  async findById(id) {
    // All SELECT logic here
  }

  async update(id, changes) {
    // All UPDATE logic here
  }

  async delete(id) {
    // All DELETE logic here
  }
}

// Service — business logic, uses repository
class TodoService {
  constructor(repository) {
    this.repository = repository;
  }

  async createTask(title, userId) {
    // Validate, enrich, transform
    const task = await this.repository.create({ title, userId, completed: false });
    return task;
  }
}

// Route — just HTTP
router.post('/tasks', async (req, res) => {
  const task = await todoService.createTask(req.body.title, req.user.id);
  res.json(task);
});
```

**How to guide AI:**

In your CLAUDE.md or prompt:

```markdown
## Architecture Rules

### Repository Pattern
- ALL database queries go in repositories/ directory
- Services never call db.run() or db.get() directly
- Routes never touch the database
- Repositories accept a db connection in the constructor
```

### Pattern 2: Service Layer

**What it does:** Business logic lives in services, not route handlers. Routes are just HTTP plumbing.

**Why it matters with AI:** AI generates fat route handlers. The Service Layer enforces a clear separation: services contain the logic, routes call the services.

**Template:**

```javascript
// Service — contains ALL business logic
class EventService {
  constructor(eventRepository, rsvpRepository) {
    this.eventRepository = eventRepository;
    this.rsvpRepository = rsvpRepository;
  }

  async createEvent(title, date, category, createdBy) {
    // Validation
    if (!title) throw new Error('Title required');
    if (!date) throw new Error('Date required');

    // Business rule: only allow future dates
    if (new Date(date) < new Date()) {
      throw new Error('Cannot create events in the past');
    }

    // Persist and return
    return await this.eventRepository.create({
      title,
      date,
      category,
      createdBy,
    });
  }

  async rsvpForEvent(eventId, userId) {
    // Business rule: can't RSVP to an event that doesn't exist
    const event = await this.eventRepository.findById(eventId);
    if (!event) throw new Error('Event not found');

    // Business rule: can't RSVP twice
    const existing = await this.rsvpRepository.findByEventAndUser(eventId, userId);
    if (existing) throw new Error('Already RSVP\'d');

    // Persist and return
    return await this.rsvpRepository.create({ eventId, userId });
  }

  async getEventRsvpCount(eventId) {
    // Business logic: aggregate
    return await this.rsvpRepository.countByEvent(eventId);
  }
}

// Route — just HTTP plumbing
router.post('/events/:id/rsvp', async (req, res, next) => {
  try {
    const rsvp = await eventService.rsvpForEvent(req.params.id, req.user.id);
    res.json(rsvp);
  } catch (err) {
    next(err); // Pass to error handler
  }
});
```

**How to guide AI:**

```markdown
## Service Layer Rule

- Business logic MUST go in services/
- If the logic answers "what should happen?", it's business logic
- Examples: validation, checking business rules, coordination
- NEVER put business logic in route handlers
- Routes are ONLY: validate HTTP format, call service, return HTTP response
```

### Pattern 3: Middleware Pattern

**What it does:** Cross-cutting concerns (auth, validation, logging) become reusable middleware.

**Why it matters with AI:** AI generates auth checks inside every route. Middleware extracts these to one place.

**Template:**

```javascript
// middleware/auth.js — reusable auth
const auth = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'Unauthorized' });

  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (err) {
    res.status(403).json({ error: 'Invalid token' });
  }
};

// middleware/validation.js — reusable validation
const validateEvent = (req, res, next) => {
  const { title, date } = req.body;
  if (!title) return res.status(400).json({ error: 'Title required' });
  if (!date) return res.status(400).json({ error: 'Date required' });
  next();
};

// middleware/errorHandler.js — centralized errors
const errorHandler = (err, req, res, next) => {
  console.error(err);
  const status = err.status || 500;
  const message = err.message || 'Internal server error';
  res.status(status).json({ error: message });
};

// Usage in routes
app.post('/events', auth, validateEvent, (req, res, next) => {
  // At this point:
  // - req.user is set (by auth middleware)
  // - req.body is validated (by validateEvent middleware)
  // - Any errors will be caught by errorHandler
  eventService
    .createEvent({ ...req.body, createdBy: req.user.id })
    .then((event) => res.json(event))
    .catch((err) => next(err));
});
```

**How to guide AI:**

```markdown
## Middleware Pattern

- Authentication logic goes in middleware/auth.js
- Input validation goes in middleware/validation.js
- Error handling goes in middleware/errorHandler.js
- Never write auth or validation logic in route handlers
- Use app.use(middleware) for global middleware
- Use router.post('/path', middleware, handler) for route-specific middleware
```

---

## Instruction Files as Architecture Guides

Your CLAUDE.md (or .cursorrules / GEMINI.md) is a contract between you and the AI. Use it to enforce architecture at generation time, not refactoring time.

### Example CLAUDE.md for Clean Architecture

Create this file at your repo root:

```markdown
# Claude Code Instructions for Campus Event Board API

## Architecture Philosophy

This project follows clean architecture with clear layer separation:
- Routes handle HTTP only
- Services handle business logic
- Repositories handle data access
- Middleware handles cross-cutting concerns

## Mandatory Rules

### Rule 1: No Database Queries in Route Handlers
- NEVER write db.run(), db.get(), or db.all() in routes/
- All database queries MUST go in repositories/
- Routes delegate to services, services delegate to repositories

### Rule 2: No Business Logic in Routes
- Routes are ONLY: parse HTTP input → call service → return HTTP response
- If the logic answers "should we allow this?", it's business logic → goes in a service
- Examples of business logic:
  - Checking if a user already RSVP'd (belongs in EventService, not route)
  - Preventing events in the past (belongs in EventService)
  - Generating unique IDs or timestamps (belongs in service or repository)

### Rule 3: Reusable Middleware for Auth & Validation
- Authentication MUST use middleware/auth.js (apply with app.use() or route-specific)
- Input validation MUST use middleware/validation.js
- Validation errors MUST be caught by middleware/errorHandler.js
- Never replicate auth logic across multiple routes

### Rule 4: Dependency Injection
- Services receive repositories in the constructor
- Routes receive services in the constructor (or as dependencies)
- Benefits: testable, no global state, easy to mock

### Rule 5: Error Handling
- All errors flow through middleware/errorHandler.js
- Services throw descriptive errors with status codes
- Routes use try/catch or .catch() and pass to next(err)

### Rule 6: File Structure
```
src/
├── index.js (setup, middleware, listen)
├── routes/
│   ├── events.js
│   └── rsvps.js
├── services/
│   ├── eventService.js
│   └── rsvpService.js
├── repositories/
│   ├── eventRepository.js
│   └── rsvpRepository.js
└── middleware/
    ├── auth.js
    ├── validation.js
    └── errorHandler.js
```

## Code Examples

### CORRECT: Route Handler
```javascript
router.post('/events', auth, validateEvent, async (req, res, next) => {
  try {
    const event = await eventService.createEvent({
      title: req.body.title,
      date: req.body.date,
      category: req.body.category,
      createdBy: req.user.id,
    });
    res.status(201).json(event);
  } catch (err) {
    next(err);
  }
});
```

### CORRECT: Service
```javascript
class EventService {
  constructor(eventRepository) {
    this.eventRepository = eventRepository;
  }

  async createEvent(eventData) {
    // Validation here
    if (!eventData.title) throw new Error('Title required');
    // Business logic here
    if (new Date(eventData.date) < new Date()) {
      throw new Error('Cannot create past events');
    }
    // Delegate to repository
    return await this.eventRepository.create(eventData);
  }
}
```

### INCORRECT: Route with DB queries
```javascript
// DON'T DO THIS
router.post('/events', (req, res) => {
  db.run('INSERT INTO events...', (err) => {
    // DB logic in route handler!
  });
});
```

## When to Refactor

If you see any of these, ask Claude to refactor:
1. `db.run()` or `db.get()` in a route handler
2. Business logic (validation, rules, coordination) in a route
3. Auth checks duplicated across multiple routes
4. Unhandled errors or missing try/catch blocks

## Testing Guidance

Services should be testable in isolation without a database:
- Mock the repository: `new EventService(mockRepository)`
- Test service logic without HTTP
- Test routes with mocked services

Routes should be testable with a test client (e.g., supertest):
- Mock the service
- Test that routes call services correctly
- Test HTTP response codes and formats
```

### How to Use This File

1. **When generating code:** "Build the event creation feature following the CLAUDE.md rules in this repo"
2. **When refactoring:** "This code violates Rule 1 (database queries in routes). Refactor it using the file structure and examples in CLAUDE.md"
3. **When extending:** "Add a new endpoint for deleting events. Follow the same patterns in CLAUDE.md"

---

## The Refactoring Workflow

This is the practical workflow you'll use every week when AI generates code:

### Step 1: Generate Messy Code (Intentionally)

Ask AI to generate fast, without worrying about architecture. Get a working prototype:

```
Build a REST API for the Campus Event Board. Don't worry about
architecture yet — just get it working. I'll refactor it next.
```

AI delivers a 500-line index.js with everything mixed together. Good.

### Step 2: Identify Violations

Scan the code (or use a checklist) and identify where architecture is violated:

**Checklist:**

- [ ] Do any route handlers call `db.run()` or `db.get()`?
- [ ] Is there business logic (validation, rules) in route handlers?
- [ ] Is auth code duplicated across multiple routes?
- [ ] Are there unhandled errors or missing error middleware?
- [ ] Could I test this service without a database?

### Step 3: Ask AI to Refactor

Tell Claude exactly what to refactor:

```
I have a Campus Event Board API that violates clean architecture.

Current issues:
1. Database queries are in route handlers (routes/events.js:45)
2. Validation logic is duplicated across POST and PUT handlers
3. No service layer for business logic

Please refactor using these rules:
1. Create repositories/ with EventRepository (all DB queries)
2. Create services/ with EventService (all business logic)
3. Create middleware/ with validation.js and errorHandler.js
4. Routes should ONLY: validate HTTP format, call service, return response

Use the file structure from CLAUDE.md.
```

### Step 4: Verify Structure

After refactoring, verify the code follows the pattern:

**Sample test:**

```bash
# Should pass: no database calls in routes
grep -r "db.run\|db.get\|db.all" src/routes/ && echo "FAIL: DB in routes" || echo "PASS"

# Should pass: services folder exists
[ -d src/services ] && echo "PASS: services/ exists" || echo "FAIL"

# Should pass: middleware folder exists
[ -d src/middleware ] && echo "PASS: middleware/ exists" || echo "FAIL"
```

### Step 5: Write Tests

Now write tests that verify architecture (bonus: these tests guide AI in the future):

```javascript
// test/architecture.test.js
import fs from 'fs';
import path from 'path';

describe('Architecture Rules', () => {
  it('should not have database calls in route handlers', () => {
    const routesDir = 'src/routes';
    const files = fs.readdirSync(routesDir);

    files.forEach((file) => {
      const content = fs.readFileSync(path.join(routesDir, file), 'utf-8');
      expect(content).not.toMatch(/db\.(run|get|all)\(/);
    });
  });

  it('should have a repositories folder', () => {
    expect(fs.existsSync('src/repositories')).toBe(true);
  });

  it('should have a services folder', () => {
    expect(fs.existsSync('src/services')).toBe(true);
  });

  it('should have a middleware folder', () => {
    expect(fs.existsSync('src/middleware')).toBe(true);
  });
});
```

---

## The Dependency Rule (Robert C. Martin)

Clean Architecture has one cardinal rule: **source code dependencies point inward.**

```
      Routes
       ↓ ↓
   Middleware
       ↓ ↓
    Services
       ↓ ↓
  Repositories
       ↓ ↓
    Models/DB
```

**What this means:**

- Routes depend on Services (✓ correct)
- Services depend on Repositories (✓ correct)
- Repositories depend on Models (✓ correct)
- **Repositories should NEVER depend on Services** (✗ wrong)
- **Services should NEVER depend on Routes** (✗ wrong)

**Why it matters:**

If Repositories depend on Services, you've created a circular dependency. Now you can't test the repository without the service, and you can't test the service without the repository.

**Example of violating the rule:**

```javascript
// WRONG: Repository depends on Service
class EventRepository {
  constructor(db, eventService) {
    this.eventService = eventService; // Circular!
  }

  create(event) {
    // ...
    this.eventService.logEventCreated(event); // Business logic in repository
    // ...
  }
}
```

**Example of following the rule:**

```javascript
// CORRECT: Repository is independent, Service uses Repository
class EventRepository {
  constructor(db) {
    this.db = db;
  }

  create(event) {
    // Just persist
    return this.db.run('INSERT INTO events...');
  }
}

class EventService {
  constructor(repository) {
    this.repository = repository;
  }

  createEvent(event) {
    const result = this.repository.create(event);
    this.logEventCreated(event); // Service controls the flow
    return result;
  }
}
```

**How to guide AI:**

```markdown
## Dependency Direction

Services can import from repositories.
Routes can import from services and middleware.
Repositories NEVER import from services.
Services NEVER import from routes.

This keeps the flow one-way: Routes → Services → Repositories
```

---

## Lab Exercise

### Part A: Refactor Your Week 3 Project

**Requirements:**

1. **Choose one route from your Week 3 deployed project** (the easiest one to start with)

2. **Ask AI to refactor it using clean architecture:**

```
I have a REST API deployed in Week 3. The [feature name] route
currently mixes database queries, validation, and business logic.

Please refactor it using clean architecture:
1. Create a Repository for all database access
2. Create a Service for business logic and validation
3. Update the Route to use the Service
4. Create Middleware for auth/validation if needed

Follow the file structure: src/routes, src/services, src/repositories, src/middleware

Start with just this one feature — don't refactor the whole API yet.
```

3. **Test the refactored code:**
   - Tests pass (from Week 5)
   - No database calls in route handlers
   - Service can be tested with a mock repository

4. **Push to GitHub with a commit message:**

```
refactor: apply clean architecture to [feature] endpoint

- Created [feature]Repository for data access
- Created [feature]Service for business logic
- Updated route to use service layer
- Added architecture validation tests

Follows CLAUDE.md patterns:
✓ No DB queries in routes
✓ Business logic in services
✓ Middleware for cross-cutting concerns
```

### Part B: Build Your CLAUDE.md

**Requirements:**

1. **Create CLAUDE.md at your repo root** with rules for your specific project

2. **Include sections:**
   - Architecture Philosophy (2-3 sentences)
   - Mandatory Rules (at least 5, specific to your API)
   - Code Examples (show CORRECT vs INCORRECT)
   - File Structure (your exact folder layout)
   - When to Refactor (your specific patterns)

3. **Test your instruction file by asking AI to generate something new:**

```
Add a new endpoint for [new feature]. Follow the rules in CLAUDE.md.
```

Check that the generated code follows your rules without needing refactoring.

### Part C: Architecture Validation Tests

**Requirements:**

1. **Create test/architecture.test.js** that validates your structure:

```javascript
// test/architecture.test.js
describe('Architecture Rules', () => {
  it('should not have database calls in routes', () => {
    // Check: grep src/routes/* for db calls
  });

  it('should have all required folders', () => {
    // Check: repositories/, services/, middleware/ exist
  });

  it('should follow file naming convention', () => {
    // Check: eventRepository.js, EventRepository class, etc.
  });

  it('should not have circular dependencies', () => {
    // Check: services don't import routes, etc.
  });
});
```

2. **Commit to GitHub:**

```
test: add architecture validation tests

Ensures clean architecture rules are maintained:
✓ No database calls in routes
✓ Required folders exist
✓ No circular dependencies
```

### Submission Checklist

Push to GitHub with:
- [ ] Refactored feature in Part A (one route, clean architecture)
- [ ] CLAUDE.md file with architecture rules
- [ ] Architecture validation tests passing
- [ ] Commit messages describing the refactoring
- [ ] Brief README section: "Architecture & Patterns" explaining your choices

---

## Quick Pulse Check (End)

Reflect on today's session:

| Question | 1 | 2 | 3 | 4 | 5 |
|----------|---|---|---|---|---|
| Can you explain the three patterns (Repository, Service, Middleware)? | | | | | |
| Would you be able to refactor messy AI code into clean architecture? | | | | | |
| Do you understand why the Dependency Rule matters? | | | | | |
| Could you write a CLAUDE.md that actually guides AI behavior? | | | | | |

Compare with your start-of-week scores. What's different now?

---

## Resources

### Architecture & Design Patterns

- [Martin Fowler: Patterns of Enterprise Application Architecture](https://martinfowler.com/books/eaa.html)
- [Martin Fowler: Repository Pattern](https://martinfowler.com/eaaCatalog/repository.html)
- [Robert C. Martin: Clean Architecture](https://www.oreilly.com/library/view/clean-architecture-a/9780134494272/) (The Dependency Rule, Chapter 15)
- [Martin Fowler: Test Pyramid](https://martinfowler.com/bliki/TestPyramid.html)

### AI-Specific Patterns

- [Anthropic: Prompt Engineering Best Practices](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering)
- [Anthropic: Extending Claude with Tools](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
- [CodeScene: Agentic AI Coding Patterns](https://www.codescene.com/engineering-blog)

### Instruction Files & Context Management

- [Cursor: .cursorrules Documentation](https://docs.cursor.sh/)
- [Anthropic: Claude.md and Project Context](https://docs.anthropic.com/en/docs/build-with-claude/system-prompts)
- [Gemini: System Instructions Guide](https://ai.google.dev/docs/system_instructions)

### Clean Code Principles

- [Robert C. Martin: Clean Code: A Handbook of Agile Software Craftsmanship](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
- [Eric Evans: Domain-Driven Design](https://www.domainlanguage.com/ddd/)

### Practical Examples

- [Node.js Express Best Practices](https://expressjs.com/en/advanced/best-practice-security.html)
- [Python Flask Patterns](https://flask.palletsprojects.com/en/latest/patterns/)

---

*Instructor: Goker Ezberci | gokerez@gmail.com*

*Vibe Coding 101: From Idea to Shipped Product — Week 6*
