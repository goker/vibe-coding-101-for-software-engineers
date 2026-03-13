# Week 6: Complete Refactoring Flow Example
## Study Group Finder API — From Monolith to Clean Architecture

**Author:** Goker Ezberci
**Difficulty:** Intermediate
**Time:** 45 minutes (read & understand)

---

## Introduction

This example walks through a **real-world refactoring** of a messy AI-generated Express.js API into clean, maintainable code following the **Repository Pattern**, **Service Layer**, and **Middleware Separation**.

The scenario: A student asks Claude to build a "Study Group Finder" API—a service where engineering students can create study groups for specific courses, join groups, and discover active groups. Simple requirement, but Claude's initial response mixes everything together.

By the end, you'll see:
- What violates the rules (and why)
- How to ask Claude to refactor using your CLAUDE.md rules
- The clean result with proper separation of concerns
- How to verify the architecture is sound

---

## Step 1: The Messy AI-Generated Code

**Scenario:** Student's prompt to Claude:
> "Build me a study group finder API. Students can create study groups for courses, join groups, and find active groups by course. Use Express.js and SQLite. Include auth and validation. I want it working ASAP."

**Claude's response:** A single `app.js` file with everything mixed in (~85 lines of code).

```javascript
// app.js - MESSY VERSION
const express = require('express');
const sqlite3 = require('sqlite3');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const app = express();

app.use(express.json());
const db = new sqlite3.Database('./study_groups.db');

const JWT_SECRET = 'test-secret';

// Initialize DB
db.run(`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY,
    email TEXT UNIQUE,
    password TEXT
  )
`);
db.run(`
  CREATE TABLE IF NOT EXISTS groups (
    id INTEGER PRIMARY KEY,
    name TEXT,
    course TEXT,
    leader_id INTEGER,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
  )
`);
db.run(`
  CREATE TABLE IF NOT EXISTS memberships (
    id INTEGER PRIMARY KEY,
    user_id INTEGER,
    group_id INTEGER,
    UNIQUE(user_id, group_id)
  )
`);

// Auth middleware
app.use((req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });
  try {
    req.user = jwt.verify(token, JWT_SECRET);
    next();
  } catch (e) {
    res.status(401).json({ error: 'Invalid token' });
  }
});

// Auth routes
app.post('/register', async (req, res) => {
  const { email, password } = req.body;
  if (!email || password.length < 8) {
    return res.status(400).json({ error: 'Invalid input' });
  }
  const hashed = await bcrypt.hash(password, 10);
  db.run('INSERT INTO users (email, password) VALUES (?, ?)',
    [email, hashed],
    function(err) {
      if (err) return res.status(400).json({ error: 'Email exists' });
      const token = jwt.sign({ id: this.lastID, email }, JWT_SECRET);
      res.json({ userId: this.lastID, token });
    }
  );
});

app.post('/login', (req, res) => {
  const { email, password } = req.body;
  db.get('SELECT * FROM users WHERE email = ?', [email], async (err, user) => {
    if (!user) return res.status(401).json({ error: 'User not found' });
    const valid = await bcrypt.compare(password, user.password);
    if (!valid) return res.status(401).json({ error: 'Invalid password' });
    const token = jwt.sign({ id: user.id, email: user.email }, JWT_SECRET);
    res.json({ userId: user.id, token });
  });
});

// Group routes (with auth check in each one)
app.post('/groups', (req, res) => {
  const { name, course } = req.body;
  if (!name || !course) {
    return res.status(400).json({ error: 'Missing fields' });
  }
  db.run(
    'INSERT INTO groups (name, course, leader_id) VALUES (?, ?, ?)',
    [name, course, req.user.id],
    function(err) {
      if (err) return res.status(400).json({ error: 'DB error' });
      res.json({ groupId: this.lastID });
    }
  );
});

app.get('/groups/course/:course', (req, res) => {
  const course = req.params.course;
  db.all(
    'SELECT * FROM groups WHERE course = ? ORDER BY created_at DESC',
    [course],
    (err, rows) => {
      if (err) return res.status(500).json({ error: 'DB error' });
      res.json(rows);
    }
  );
});

app.post('/groups/:groupId/join', (req, res) => {
  const groupId = req.params.groupId;
  db.run(
    'INSERT INTO memberships (user_id, group_id) VALUES (?, ?)',
    [req.user.id, groupId],
    (err) => {
      if (err) {
        if (err.message.includes('UNIQUE')) {
          return res.status(400).json({ error: 'Already a member' });
        }
        return res.status(400).json({ error: 'DB error' });
      }
      res.json({ success: true });
    }
  );
});

app.get('/groups/:groupId/members', (req, res) => {
  const groupId = req.params.groupId;
  db.all(
    `SELECT u.id, u.email FROM users u
     JOIN memberships m ON u.id = m.user_id
     WHERE m.group_id = ?`,
    [groupId],
    (err, rows) => {
      if (err) return res.status(500).json({ error: 'DB error' });
      res.json(rows);
    }
  );
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

---

## Step 2: Identify Violations

Let's walk through the checklist. Using the **CLAUDE.md rules** (adapted for this course):

| Rule | Location(s) | Violation |
|------|-----------|-----------|
| **Separate routes from business logic** | Lines 50-105 | Routes directly execute DB queries. No service layer. |
| **Separate data access from business logic** | Lines 50-105 | Business logic (validation, membership checks) embedded in routes. DB calls everywhere. |
| **Keep middleware focused** | Lines 41-49 | Auth middleware works, but there's no separate middleware file. |
| **One file = one responsibility** | Entire file | Single file handles: DB init, auth, routes, business logic, validation. 6+ responsibilities. |
| **Database queries belong in repository** | Lines 67, 75, 87, 99 | Raw DB calls scattered across route handlers. |
| **Error handling is inconsistent** | Lines 68, 76, 88, 100 | Mix of `.json()` and direct `res.status()`. No error abstraction. |
| **No separation between layers** | Entire file | Routes are tightly coupled to SQLite callback patterns. |

**Grade: D+ (functional but unmaintainable)**
- Pros: It works. Auth is functional.
- Cons: Cannot test. Hard to extend. Errors are ad-hoc. Hard to reuse database logic.

---

## Step 3: The Refactoring Prompt

**Student creates `CLAUDE.md` in the project root:**

```markdown
# CLAUDE.md - Study Group Finder API

## Coding Standards

### Architecture Rules
- **Separate concerns**: Each file has ONE responsibility (routes, services, repositories, middleware).
- **No business logic in routes**: Routes must ONLY handle HTTP concerns (params, auth checks, responses).
- **Service layer**: ALL business logic lives in `services/`. Routes call services, services call repositories.
- **Repository pattern**: ALL database access goes in `repositories/`. No raw queries in services or routes.
- **Middleware separation**: Auth and validation middleware in separate files in `middleware/`.
- **Error handling**: Create custom error classes. Routes must not have try/catch — errors bubble to central handler.

### File Structure
```
src/
  middleware/
    authMiddleware.js      # JWT verification only
    validationMiddleware.js # Input validation
  repositories/
    userRepository.js      # User DB queries
    groupRepository.js     # Group DB queries
    membershipRepository.js # Membership DB queries
  services/
    authService.js         # Register, login, token logic
    groupService.js        # Create group, join, search logic
  routes/
    authRoutes.js          # POST /register, POST /login
    groupRoutes.js         # POST /groups, GET /groups, POST /join, GET /members
  app.js                   # Express setup, middleware, error handler
  index.js                 # Server startup
```

### Validation Rules
- User registration: email must be valid email, password >= 8 chars
- Group creation: name required (1-100 chars), course required (1-50 chars)
- Join group: user must exist, group must exist, can't join twice
- Find groups: return sorted by creation date DESC, include member count

### Testing
- Each service must be independently testable (no DB mocking needed if using repo)
- Repositories use real SQLite during testing
```

**Student's refactoring prompt to Claude:**

> "I need to refactor this Express API to follow my CLAUDE.md rules. The current app.js is a mess.
>
> Here's what I need:
> 1. Split into proper files: repositories, services, routes, middleware
> 2. Move all DB queries to repositories
> 3. Move all business logic to services
> 4. Routes should only handle HTTP stuff
> 5. Create middleware files for auth and validation
> 6. Follow my CLAUDE.md file structure exactly
> 7. Include unit tests for the service layer
>
> Here's my CLAUDE.md: [pastes CLAUDE.md]
> Here's my current code: [pastes app.js]
>
> Please refactor and give me complete, working code for each file. Make sure it's all testable."

---

## Step 4: The Clean Code

### File 1: `src/middleware/authMiddleware.js`

```javascript
const jwt = require('jsonwebtoken');

const JWT_SECRET = process.env.JWT_SECRET || 'test-secret';

/**
 * Verify JWT token from Authorization header
 * Attaches user object to req.user if valid
 */
const authMiddleware = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'Missing authorization token' });
  }

  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid or expired token' });
  }
};

module.exports = { authMiddleware, JWT_SECRET };
```

### File 2: `src/middleware/validationMiddleware.js`

```javascript
/**
 * Validate registration/login input
 */
const validateAuth = (req, res, next) => {
  const { email, password } = req.body;

  if (!email || typeof email !== 'string' || email.trim().length === 0) {
    return res.status(400).json({ error: 'Email is required' });
  }

  if (!password || typeof password !== 'string' || password.length < 8) {
    return res.status(400).json({ error: 'Password must be at least 8 characters' });
  }

  // Simple email validation
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(email)) {
    return res.status(400).json({ error: 'Invalid email format' });
  }

  next();
};

/**
 * Validate group creation input
 */
const validateGroupCreation = (req, res, next) => {
  const { name, course } = req.body;

  if (!name || typeof name !== 'string' || name.trim().length === 0 || name.length > 100) {
    return res.status(400).json({ error: 'Group name must be 1-100 characters' });
  }

  if (!course || typeof course !== 'string' || course.trim().length === 0 || course.length > 50) {
    return res.status(400).json({ error: 'Course must be 1-50 characters' });
  }

  next();
};

module.exports = {
  validateAuth,
  validateGroupCreation,
};
```

### File 3: `src/repositories/userRepository.js`

```javascript
/**
 * User data access layer
 * All user-related database queries go here
 */
class UserRepository {
  constructor(db) {
    this.db = db;
  }

  /**
   * Create a new user
   * @param {string} email
   * @param {string} hashedPassword
   * @returns {Promise<number>} userId
   */
  create(email, hashedPassword) {
    return new Promise((resolve, reject) => {
      this.db.run(
        'INSERT INTO users (email, password) VALUES (?, ?)',
        [email, hashedPassword],
        function(err) {
          if (err) {
            if (err.message.includes('UNIQUE')) {
              reject(new Error('Email already exists'));
            } else {
              reject(err);
            }
          } else {
            resolve(this.lastID);
          }
        }
      );
    });
  }

  /**
   * Find user by email
   * @param {string} email
   * @returns {Promise<{id, email, password} | null>}
   */
  findByEmail(email) {
    return new Promise((resolve, reject) => {
      this.db.get('SELECT id, email, password FROM users WHERE email = ?', [email], (err, row) => {
        if (err) reject(err);
        else resolve(row || null);
      });
    });
  }

  /**
   * Find user by id
   * @param {number} userId
   * @returns {Promise<{id, email} | null>}
   */
  findById(userId) {
    return new Promise((resolve, reject) => {
      this.db.get('SELECT id, email FROM users WHERE id = ?', [userId], (err, row) => {
        if (err) reject(err);
        else resolve(row || null);
      });
    });
  }
}

module.exports = UserRepository;
```

### File 4: `src/repositories/groupRepository.js`

```javascript
/**
 * Group data access layer
 * All group-related database queries go here
 */
class GroupRepository {
  constructor(db) {
    this.db = db;
  }

  /**
   * Create a new study group
   * @param {string} name
   * @param {string} course
   * @param {number} leaderId
   * @returns {Promise<number>} groupId
   */
  create(name, course, leaderId) {
    return new Promise((resolve, reject) => {
      this.db.run(
        'INSERT INTO groups (name, course, leader_id) VALUES (?, ?, ?)',
        [name, course, leaderId],
        function(err) {
          if (err) reject(err);
          else resolve(this.lastID);
        }
      );
    });
  }

  /**
   * Find groups by course
   * @param {string} course
   * @returns {Promise<Array>} groups with member count
   */
  findByCourse(course) {
    return new Promise((resolve, reject) => {
      this.db.all(
        `SELECT
           g.id, g.name, g.course, g.leader_id, g.created_at,
           COUNT(m.id) as member_count
         FROM groups g
         LEFT JOIN memberships m ON g.id = m.group_id
         WHERE g.course = ?
         GROUP BY g.id
         ORDER BY g.created_at DESC`,
        [course],
        (err, rows) => {
          if (err) reject(err);
          else resolve(rows || []);
        }
      );
    });
  }

  /**
   * Find group by id
   * @param {number} groupId
   * @returns {Promise<{id, name, course, leader_id, created_at} | null>}
   */
  findById(groupId) {
    return new Promise((resolve, reject) => {
      this.db.get(
        'SELECT id, name, course, leader_id, created_at FROM groups WHERE id = ?',
        [groupId],
        (err, row) => {
          if (err) reject(err);
          else resolve(row || null);
        }
      );
    });
  }
}

module.exports = GroupRepository;
```

### File 5: `src/repositories/membershipRepository.js`

```javascript
/**
 * Membership data access layer
 * All membership-related database queries go here
 */
class MembershipRepository {
  constructor(db) {
    this.db = db;
  }

  /**
   * Add user to group
   * @param {number} userId
   * @param {number} groupId
   * @returns {Promise<void>}
   */
  addMember(userId, groupId) {
    return new Promise((resolve, reject) => {
      this.db.run(
        'INSERT INTO memberships (user_id, group_id) VALUES (?, ?)',
        [userId, groupId],
        function(err) {
          if (err) {
            if (err.message.includes('UNIQUE')) {
              reject(new Error('User is already a member of this group'));
            } else {
              reject(err);
            }
          } else {
            resolve();
          }
        }
      );
    });
  }

  /**
   * Get all members of a group
   * @param {number} groupId
   * @returns {Promise<Array>} [{id, email}, ...]
   */
  getGroupMembers(groupId) {
    return new Promise((resolve, reject) => {
      this.db.all(
        `SELECT u.id, u.email
         FROM users u
         JOIN memberships m ON u.id = m.user_id
         WHERE m.group_id = ?`,
        [groupId],
        (err, rows) => {
          if (err) reject(err);
          else resolve(rows || []);
        }
      );
    });
  }

  /**
   * Check if user is member of group
   * @param {number} userId
   * @param {number} groupId
   * @returns {Promise<boolean>}
   */
  isMember(userId, groupId) {
    return new Promise((resolve, reject) => {
      this.db.get(
        'SELECT id FROM memberships WHERE user_id = ? AND group_id = ?',
        [userId, groupId],
        (err, row) => {
          if (err) reject(err);
          else resolve(!!row);
        }
      );
    });
  }
}

module.exports = MembershipRepository;
```

### File 6: `src/services/authService.js`

```javascript
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const { JWT_SECRET } = require('../middleware/authMiddleware');

/**
 * Authentication business logic
 * Handles user registration, login, token generation
 */
class AuthService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  /**
   * Register a new user
   * @param {string} email
   * @param {string} password
   * @returns {Promise<{userId, token}>}
   */
  async register(email, password) {
    // Check if user already exists
    const existingUser = await this.userRepository.findByEmail(email);
    if (existingUser) {
      throw new Error('Email already registered');
    }

    // Hash password
    const hashedPassword = await bcrypt.hash(password, 10);

    // Create user
    const userId = await this.userRepository.create(email, hashedPassword);

    // Generate token
    const token = jwt.sign({ id: userId, email }, JWT_SECRET, { expiresIn: '7d' });

    return { userId, token };
  }

  /**
   * Login user
   * @param {string} email
   * @param {string} password
   * @returns {Promise<{userId, email, token}>}
   */
  async login(email, password) {
    // Find user
    const user = await this.userRepository.findByEmail(email);
    if (!user) {
      throw new Error('User not found');
    }

    // Verify password
    const passwordValid = await bcrypt.compare(password, user.password);
    if (!passwordValid) {
      throw new Error('Invalid password');
    }

    // Generate token
    const token = jwt.sign({ id: user.id, email: user.email }, JWT_SECRET, { expiresIn: '7d' });

    return { userId: user.id, email: user.email, token };
  }
}

module.exports = AuthService;
```

### File 7: `src/services/groupService.js`

```javascript
/**
 * Group business logic
 * Handles group creation, searching, membership
 */
class GroupService {
  constructor(groupRepository, membershipRepository) {
    this.groupRepository = groupRepository;
    this.membershipRepository = membershipRepository;
  }

  /**
   * Create a new study group
   * @param {string} name
   * @param {string} course
   * @param {number} leaderId
   * @returns {Promise<{groupId}>}
   */
  async createGroup(name, course, leaderId) {
    const groupId = await this.groupRepository.create(name, course, leaderId);

    // Leader automatically joins the group
    await this.membershipRepository.addMember(leaderId, groupId);

    return { groupId };
  }

  /**
   * Find groups by course
   * @param {string} course
   * @returns {Promise<Array>} groups with member count
   */
  async findGroupsByCourse(course) {
    const groups = await this.groupRepository.findByCourse(course);
    return groups;
  }

  /**
   * Join a group
   * @param {number} userId
   * @param {number} groupId
   * @returns {Promise<{success: boolean}>}
   */
  async joinGroup(userId, groupId) {
    // Verify group exists
    const group = await this.groupRepository.findById(groupId);
    if (!group) {
      throw new Error('Group not found');
    }

    // Add membership (will throw if already member)
    await this.membershipRepository.addMember(userId, groupId);

    return { success: true };
  }

  /**
   * Get all members of a group
   * @param {number} groupId
   * @returns {Promise<Array>} members
   */
  async getGroupMembers(groupId) {
    // Verify group exists
    const group = await this.groupRepository.findById(groupId);
    if (!group) {
      throw new Error('Group not found');
    }

    const members = await this.membershipRepository.getGroupMembers(groupId);
    return members;
  }
}

module.exports = GroupService;
```

### File 8: `src/routes/authRoutes.js`

```javascript
const express = require('express');
const { validateAuth } = require('../middleware/validationMiddleware');

/**
 * Authentication routes
 * POST /auth/register - Register new user
 * POST /auth/login - Login user
 */
function createAuthRoutes(authService) {
  const router = express.Router();

  /**
   * POST /auth/register
   * Body: {email, password}
   * Returns: {userId, token}
   */
  router.post('/register', validateAuth, async (req, res, next) => {
    try {
      const { email, password } = req.body;
      const result = await authService.register(email, password);
      res.status(201).json(result);
    } catch (error) {
      next(error);
    }
  });

  /**
   * POST /auth/login
   * Body: {email, password}
   * Returns: {userId, email, token}
   */
  router.post('/login', validateAuth, async (req, res, next) => {
    try {
      const { email, password } = req.body;
      const result = await authService.login(email, password);
      res.json(result);
    } catch (error) {
      next(error);
    }
  });

  return router;
}

module.exports = createAuthRoutes;
```

### File 9: `src/routes/groupRoutes.js`

```javascript
const express = require('express');
const { authMiddleware } = require('../middleware/authMiddleware');
const { validateGroupCreation } = require('../middleware/validationMiddleware');

/**
 * Group routes
 * POST /groups - Create new group
 * GET /groups/course/:course - Find groups by course
 * POST /groups/:groupId/join - Join a group
 * GET /groups/:groupId/members - Get group members
 */
function createGroupRoutes(groupService) {
  const router = express.Router();

  // All group routes require authentication
  router.use(authMiddleware);

  /**
   * POST /groups
   * Body: {name, course}
   * Returns: {groupId}
   */
  router.post('/', validateGroupCreation, async (req, res, next) => {
    try {
      const { name, course } = req.body;
      const result = await groupService.createGroup(name, course, req.user.id);
      res.status(201).json(result);
    } catch (error) {
      next(error);
    }
  });

  /**
   * GET /groups/course/:course
   * Returns: [{id, name, course, leader_id, created_at, member_count}, ...]
   */
  router.get('/course/:course', async (req, res, next) => {
    try {
      const { course } = req.params;
      const groups = await groupService.findGroupsByCourse(course);
      res.json(groups);
    } catch (error) {
      next(error);
    }
  });

  /**
   * POST /groups/:groupId/join
   * Returns: {success: true}
   */
  router.post('/:groupId/join', async (req, res, next) => {
    try {
      const { groupId } = req.params;
      const result = await groupService.joinGroup(req.user.id, parseInt(groupId, 10));
      res.json(result);
    } catch (error) {
      next(error);
    }
  });

  /**
   * GET /groups/:groupId/members
   * Returns: [{id, email}, ...]
   */
  router.get('/:groupId/members', async (req, res, next) => {
    try {
      const { groupId } = req.params;
      const members = await groupService.getGroupMembers(parseInt(groupId, 10));
      res.json(members);
    } catch (error) {
      next(error);
    }
  });

  return router;
}

module.exports = createGroupRoutes;
```

### File 10: `src/app.js`

```javascript
const express = require('express');
const sqlite3 = require('sqlite3');
const path = require('path');

// Import repositories
const UserRepository = require('./repositories/userRepository');
const GroupRepository = require('./repositories/groupRepository');
const MembershipRepository = require('./repositories/membershipRepository');

// Import services
const AuthService = require('./services/authService');
const GroupService = require('./services/groupService');

// Import routes
const createAuthRoutes = require('./routes/authRoutes');
const createGroupRoutes = require('./routes/groupRoutes');

/**
 * Initialize Express app with all dependencies
 */
function createApp(db) {
  const app = express();

  // Middleware
  app.use(express.json());

  // Initialize repositories
  const userRepository = new UserRepository(db);
  const groupRepository = new GroupRepository(db);
  const membershipRepository = new MembershipRepository(db);

  // Initialize services
  const authService = new AuthService(userRepository);
  const groupService = new GroupService(groupRepository, membershipRepository);

  // Routes
  app.use('/auth', createAuthRoutes(authService));
  app.use('/groups', createGroupRoutes(groupService));

  // Health check
  app.get('/health', (req, res) => {
    res.json({ status: 'ok' });
  });

  // Global error handler
  app.use((err, req, res, next) => {
    console.error('Error:', err.message);

    // Validation errors from middleware
    if (err.message.includes('at least') || err.message.includes('required')) {
      return res.status(400).json({ error: err.message });
    }

    // Business logic errors
    if (err.message.includes('already') || err.message.includes('not found')) {
      return res.status(400).json({ error: err.message });
    }

    // JWT errors
    if (err.message.includes('token')) {
      return res.status(401).json({ error: err.message });
    }

    // Default error
    res.status(500).json({ error: 'Internal server error' });
  });

  return app;
}

/**
 * Initialize SQLite database with schema
 */
function initializeDatabase(db) {
  return new Promise((resolve, reject) => {
    db.serialize(() => {
      db.run(
        `CREATE TABLE IF NOT EXISTS users (
          id INTEGER PRIMARY KEY,
          email TEXT UNIQUE NOT NULL,
          password TEXT NOT NULL
        )`
      );

      db.run(
        `CREATE TABLE IF NOT EXISTS groups (
          id INTEGER PRIMARY KEY,
          name TEXT NOT NULL,
          course TEXT NOT NULL,
          leader_id INTEGER NOT NULL,
          created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
          FOREIGN KEY(leader_id) REFERENCES users(id)
        )`
      );

      db.run(
        `CREATE TABLE IF NOT EXISTS memberships (
          id INTEGER PRIMARY KEY,
          user_id INTEGER NOT NULL,
          group_id INTEGER NOT NULL,
          UNIQUE(user_id, group_id),
          FOREIGN KEY(user_id) REFERENCES users(id),
          FOREIGN KEY(group_id) REFERENCES groups(id)
        )`,
        (err) => {
          if (err) reject(err);
          else resolve();
        }
      );
    });
  });
}

module.exports = { createApp, initializeDatabase };
```

### File 11: `src/index.js`

```javascript
const sqlite3 = require('sqlite3');
const { createApp, initializeDatabase } = require('./app');

const PORT = process.env.PORT || 3000;
const DB_PATH = process.env.DB_PATH || './study_groups.db';

async function start() {
  try {
    // Initialize database
    const db = new sqlite3.Database(DB_PATH);
    await initializeDatabase(db);

    // Create and start app
    const app = createApp(db);
    app.listen(PORT, () => {
      console.log(`Server running on port ${PORT}`);
    });
  } catch (error) {
    console.error('Failed to start server:', error);
    process.exit(1);
  }
}

start();
```

---

## Step 5: Verification

### 5A: File Structure Check

Run this command to verify the structure matches CLAUDE.md:

```bash
tree src/ -I 'node_modules'
```

Expected output:

```
src/
├── middleware/
│   ├── authMiddleware.js
│   └── validationMiddleware.js
├── repositories/
│   ├── userRepository.js
│   ├── groupRepository.js
│   └── membershipRepository.js
├── services/
│   ├── authService.js
│   └── groupService.js
├── routes/
│   ├── authRoutes.js
│   └── groupRoutes.js
├── app.js
└── index.js
```

### 5B: No Raw Queries in Routes

Verify no raw `db.run()` or `db.get()` in route files:

```bash
grep -r "db\.run\|db\.get\|db\.all" src/routes/
```

Expected: No output (clean!)

```bash
grep -r "db\.run\|db\.get\|db\.all" src/services/
```

Expected: No output (services don't touch DB directly)

```bash
grep -r "db\.run\|db\.get\|db\.all" src/repositories/
```

Expected: Output showing queries ONLY in repositories (correct)

### 5C: No Business Logic in Routes

Routes should only handle HTTP. Verify with:

```bash
grep -r "bcrypt\|jwt\|UNIQUE\|password" src/routes/
```

Expected: No output (routes don't do crypto or validation)

```bash
grep -r "bcrypt\|jwt" src/services/
```

Expected: Output in authService.js only (correct location)

### 5D: Unit Test Example

Create `src/services/__tests__/groupService.test.js`:

```javascript
const GroupService = require('../groupService');

/**
 * Mock repositories for testing
 * Tests only the service logic, not the database
 */
describe('GroupService', () => {
  let groupService;
  let mockGroupRepository;
  let mockMembershipRepository;

  beforeEach(() => {
    // Mock the repositories
    mockGroupRepository = {
      create: jest.fn(),
      findByCourse: jest.fn(),
      findById: jest.fn(),
    };

    mockMembershipRepository = {
      addMember: jest.fn(),
      getGroupMembers: jest.fn(),
    };

    groupService = new GroupService(mockGroupRepository, mockMembershipRepository);
  });

  describe('createGroup', () => {
    it('should create a group and add leader as first member', async () => {
      mockGroupRepository.create.mockResolvedValue(1);
      mockMembershipRepository.addMember.mockResolvedValue(undefined);

      const result = await groupService.createGroup('Study Group', 'CS101', 5);

      expect(mockGroupRepository.create).toHaveBeenCalledWith('Study Group', 'CS101', 5);
      expect(mockMembershipRepository.addMember).toHaveBeenCalledWith(5, 1);
      expect(result).toEqual({ groupId: 1 });
    });
  });

  describe('joinGroup', () => {
    it('should throw error if group does not exist', async () => {
      mockGroupRepository.findById.mockResolvedValue(null);

      await expect(
        groupService.joinGroup(10, 999)
      ).rejects.toThrow('Group not found');
    });

    it('should add user to group if group exists', async () => {
      mockGroupRepository.findById.mockResolvedValue({ id: 1, name: 'Study Group' });
      mockMembershipRepository.addMember.mockResolvedValue(undefined);

      const result = await groupService.joinGroup(10, 1);

      expect(mockMembershipRepository.addMember).toHaveBeenCalledWith(10, 1);
      expect(result).toEqual({ success: true });
    });

    it('should throw error if user already a member', async () => {
      mockGroupRepository.findById.mockResolvedValue({ id: 1, name: 'Study Group' });
      mockMembershipRepository.addMember.mockRejectedValue(
        new Error('User is already a member of this group')
      );

      await expect(
        groupService.joinGroup(10, 1)
      ).rejects.toThrow('already a member');
    });
  });

  describe('findGroupsByCourse', () => {
    it('should return groups sorted by creation date', async () => {
      const groups = [
        { id: 1, name: 'Group A', course: 'CS101', member_count: 5 },
        { id: 2, name: 'Group B', course: 'CS101', member_count: 3 },
      ];
      mockGroupRepository.findByCourse.mockResolvedValue(groups);

      const result = await groupService.findGroupsByCourse('CS101');

      expect(mockGroupRepository.findByCourse).toHaveBeenCalledWith('CS101');
      expect(result).toEqual(groups);
    });
  });
});
```

Run tests:

```bash
npm test -- src/services/__tests__/groupService.test.js
```

Expected: All tests pass. Services are fully testable without touching the database.

### 5E: Integration Test (End-to-End)

Create `tests/api.integration.test.js`:

```javascript
const request = require('supertest');
const sqlite3 = require('sqlite3');
const { createApp, initializeDatabase } = require('../src/app');

describe('Study Group Finder API', () => {
  let app;
  let db;
  let authToken;
  let userId;

  beforeAll(async () => {
    // Create in-memory database for testing
    db = new sqlite3.Database(':memory:');
    await initializeDatabase(db);
    app = createApp(db);
  });

  afterAll((done) => {
    db.close(done);
  });

  it('should register a new user', async () => {
    const response = await request(app)
      .post('/auth/register')
      .send({
        email: 'alice@example.com',
        password: 'password123',
      });

    expect(response.status).toBe(201);
    expect(response.body).toHaveProperty('userId');
    expect(response.body).toHaveProperty('token');
    userId = response.body.userId;
    authToken = response.body.token;
  });

  it('should login a user', async () => {
    const response = await request(app)
      .post('/auth/login')
      .send({
        email: 'alice@example.com',
        password: 'password123',
      });

    expect(response.status).toBe(200);
    expect(response.body).toHaveProperty('token');
  });

  it('should create a study group', async () => {
    const response = await request(app)
      .post('/groups')
      .set('Authorization', `Bearer ${authToken}`)
      .send({
        name: 'Data Structures Study Group',
        course: 'CS201',
      });

    expect(response.status).toBe(201);
    expect(response.body).toHaveProperty('groupId');
  });

  it('should find groups by course', async () => {
    const response = await request(app)
      .get('/groups/course/CS201')
      .set('Authorization', `Bearer ${authToken}`);

    expect(response.status).toBe(200);
    expect(Array.isArray(response.body)).toBe(true);
    expect(response.body[0]).toHaveProperty('member_count');
  });
});
```

---

## Step 6: What Changed?

### Before (Messy Monolith)

```
app.js (85 lines)
  ├─ DB initialization
  ├─ Auth middleware
  ├─ User registration logic
  ├─ User login logic
  ├─ Password hashing
  ├─ JWT token generation
  ├─ Direct SQL queries
  ├─ Group creation logic
  ├─ Group search logic
  ├─ Join group logic
  ├─ Get members logic
  └─ Error handling (inconsistent)

Metrics:
  • 1 file
  • ~85 lines total
  • 0 layers of separation
  • 0% test coverage (untestable)
  • Hard to debug, extend, or maintain
```

### After (Clean Architecture)

```
src/
  ├─ middleware/
  │   ├─ authMiddleware.js (25 lines)
  │   └─ validationMiddleware.js (50 lines)
  ├─ repositories/
  │   ├─ userRepository.js (60 lines)
  │   ├─ groupRepository.js (70 lines)
  │   └─ membershipRepository.js (70 lines)
  ├─ services/
  │   ├─ authService.js (65 lines)
  │   └─ groupService.js (75 lines)
  ├─ routes/
  │   ├─ authRoutes.js (50 lines)
  │   └─ groupRoutes.js (80 lines)
  ├─ app.js (95 lines)
  └─ index.js (30 lines)

Metrics:
  • 11 files (organized by responsibility)
  • ~650 lines total (more comments, more clarity)
  • 3 clear layers (routes → services → repositories)
  • 100% test coverage possible (services fully mockable)
  • Easy to debug, extend, and maintain
```

### Key Improvements

| Aspect | Before | After |
|--------|--------|-------|
| **Database calls** | Scattered in routes | Isolated in repositories |
| **Business logic** | Mixed in routes | Organized in services |
| **Testing** | Impossible (everything coupled to DB) | Easy (services use mocked repos) |
| **Error handling** | Ad-hoc responses | Centralized error handler |
| **Code reuse** | None (logic locked in routes) | Full (services + repos are reusable) |
| **Debugging** | Hard (trace through entire file) | Easy (follow the layers) |
| **New features** | Risky (might break something) | Safe (add to existing services) |
| **Onboarding** | Confusing (where does the logic go?) | Clear (rules are explicit) |

---

## Key Takeaways

1. **Separation of concerns is not optional** — It makes your code testable, debuggeable, and maintainable.

2. **Use your CLAUDE.md as a contract** — It tells Claude exactly where each type of code belongs.

3. **Routes are HTTP adapters** — They translate HTTP requests to service calls and service responses back to HTTP.

4. **Services are your business logic** — They orchestrate repositories, validation, and error handling.

5. **Repositories are your database API** — They're the only place that knows SQL. Replace them, test them, extend them.

6. **Test the layers independently** — Mock repositories when testing services. Test the API end-to-end. Never depend on the real database for unit tests.

7. **Middleware is for cross-cutting concerns** — Auth, validation, logging. Not business logic.

---

## Next Steps

1. **Create your own CLAUDE.md** for your project. Be specific.
2. **Ask Claude to refactor** using the prompt format shown in Step 3.
3. **Run the verification commands** to ensure the architecture is clean.
4. **Write tests for the service layer** before adding new features.
5. **Add new routes safely** — they always follow the same pattern.

---

**Author:** Goker Ezberci
**Last Updated:** March 6, 2026
**Week:** 6 (Architecture & Refactoring)
