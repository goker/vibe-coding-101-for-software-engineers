# PRD Example 21: Task Management API with Teams

> **Difficulty:** Advanced | **Project Type:** REST API | **Time:** 2-3 weeks

---

## Overview

| | |
|---|---|
| **What** | A REST API for task management with team collaboration features |
| **Who** | Development teams building project management or productivity apps |
| **Why** | Provides a robust backend for multi-user task tracking with team workspaces |

---

## Core Features (MVP)

1. **User Management:** Register, login, profile management
2. **Teams:** Create teams, invite members, manage roles (admin/member)
3. **Projects:** Create projects within teams
4. **Tasks:** CRUD for tasks with status, assignee, due date
5. **Task Comments:** Add comments to tasks

---

## Non-Goals

**Will NOT build:**
- File attachments
- Time tracking
- Kanban board logic (status only)
- Notifications or emails
- Activity feed or audit log
- Subtasks or dependencies
- Labels or tags
- Sprint or milestone tracking

**Will NOT use:**
- External email services
- Background job processors
- File storage services
- Complex permission libraries

---

## Technical Constraints

| | |
|---|---|
| **Language** | Python 3.11+ |
| **Framework** | FastAPI |
| **Database** | PostgreSQL with SQLAlchemy |
| **Auth** | JWT with refresh tokens |
| **Caching** | Redis for rate limiting |
| **Testing** | pytest, httpx |
| **Docs** | OpenAPI (auto-generated) |
| **Deploy** | Docker + Railway |

---

## Success Criteria

- [ ] Users can register, login, refresh tokens
- [ ] Users can create and manage teams
- [ ] Team admins can invite members
- [ ] Projects belong to teams
- [ ] Tasks belong to projects with full CRUD
- [ ] Comments can be added to tasks
- [ ] Proper authorization (team members only)
- [ ] Rate limiting prevents abuse
- [ ] All tests pass

---

## Implementation Phases

### Phase 1: Project Setup & Auth
**Goal:** Set up project with JWT authentication

**Tasks:**
1. Create FastAPI project with proper structure
2. Set up PostgreSQL with SQLAlchemy models
3. Implement User model with password hashing
4. Create auth endpoints (register, login, refresh, logout)
5. Add JWT middleware for protected routes

**Verification:**
```bash
curl -X POST /auth/register \
  -d '{"email": "user@example.com", "password": "secret123", "name": "John"}'
# Returns: {"id": "...", "email": "...", "name": "John"}

curl -X POST /auth/login \
  -d '{"email": "user@example.com", "password": "secret123"}'
# Returns: {"access_token": "...", "refresh_token": "..."}
```

**Deliverables:** Auth system with JWT

---

### Phase 2: Teams & Members
**Goal:** Implement team management

**Tasks:**
1. Create Team model (name, created_by)
2. Create TeamMember model (user_id, team_id, role)
3. Implement team CRUD endpoints
4. Implement member invitation (by email)
5. Implement role-based permissions (admin vs member)

**Verification:**
```bash
curl -X POST /teams -H "Authorization: Bearer ..." \
  -d '{"name": "Engineering"}'
# Returns: {"id": "...", "name": "Engineering", "role": "admin"}

curl -X POST /teams/{id}/members \
  -d '{"email": "colleague@example.com", "role": "member"}'
# Adds member to team

curl /teams/{id}/members
# Returns list of team members with roles
```

**Deliverables:** Team management API

---

### Phase 3: Projects & Tasks
**Goal:** Implement projects and tasks

**Tasks:**
1. Create Project model (name, team_id, description)
2. Create Task model (title, description, status, assignee_id, due_date)
3. Implement project CRUD (team-scoped)
4. Implement task CRUD (project-scoped)
5. Add task status transitions (todo → in_progress → done)

**Verification:**
```bash
curl -X POST /teams/{team_id}/projects \
  -d '{"name": "Website Redesign"}'
# Creates project in team

curl -X POST /projects/{project_id}/tasks \
  -d '{"title": "Design homepage", "assignee_id": "...", "due_date": "2025-03-01"}'
# Creates task in project

curl -X PATCH /tasks/{task_id} \
  -d '{"status": "in_progress"}'
# Updates task status
```

**Deliverables:** Project and task management

---

### Phase 4: Comments & Authorization
**Goal:** Add comments and secure endpoints

**Tasks:**
1. Create Comment model (task_id, user_id, content)
2. Implement comment CRUD
3. Add authorization middleware (team membership)
4. Verify task assignee is team member
5. Add rate limiting with Redis

**Verification:**
```bash
curl -X POST /tasks/{task_id}/comments \
  -d '{"content": "Looks good, ready for review"}'
# Adds comment

# Non-team member tries to access:
curl /teams/{team_id}/projects
# Returns: 403 Forbidden

# Rate limit test:
for i in {1..100}; do curl /tasks; done
# Returns: 429 Too Many Requests (after limit)
```

**Deliverables:** Comments and authorization

---

### Phase 5: Testing & Deployment
**Goal:** Complete testing and deploy

**Tasks:**
1. Write comprehensive pytest test suite
2. Test auth flows, team permissions, task CRUD
3. Create Dockerfile and docker-compose.yml
4. Set up CI/CD with GitHub Actions
5. Deploy to Railway

**Verification:**
```bash
pytest --cov=app
# Coverage > 80%, all tests pass

docker-compose up
# App runs with PostgreSQL and Redis

# Deploy to Railway → API accessible
curl https://api.example.com/health
# Returns: {"status": "healthy"}
```

**Deliverables:** Deployed, tested API

---

## Database Schema

```python
class User(Base):
    id: UUID
    email: str (unique)
    password_hash: str
    name: str

class Team(Base):
    id: UUID
    name: str
    created_by: UUID (FK User)

class TeamMember(Base):
    user_id: UUID (FK User)
    team_id: UUID (FK Team)
    role: Enum("admin", "member")
    # Composite PK

class Project(Base):
    id: UUID
    team_id: UUID (FK Team)
    name: str
    description: str

class Task(Base):
    id: UUID
    project_id: UUID (FK Project)
    title: str
    description: str
    status: Enum("todo", "in_progress", "done")
    assignee_id: UUID (FK User, nullable)
    due_date: date (nullable)

class Comment(Base):
    id: UUID
    task_id: UUID (FK Task)
    user_id: UUID (FK User)
    content: str
    created_at: datetime
```

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Check team membership before data access | Before changing database schema | Expose user passwords |
| Use UUIDs for public IDs | Before adding new endpoints | Allow cross-team data access |
| Validate assignee is team member | Before adding new features | Delete teams without confirmation |
| Apply rate limiting to all endpoints | | Store plain text passwords |
