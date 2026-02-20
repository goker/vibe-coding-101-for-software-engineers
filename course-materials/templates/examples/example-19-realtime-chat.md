# PRD Example 19: Real-time Chat Application

> **Difficulty:** Advanced | **Project Type:** Full Stack Web App | **Time:** 2-3 weeks

---

## Overview

| | |
|---|---|
| **What** | A real-time chat application with rooms and direct messages |
| **Who** | Teams or communities who need simple, self-hosted chat |
| **Why** | Provides real-time communication without relying on third-party services |

---

## Core Features (MVP)

1. **User Registration:** Sign up with username and password
2. **Chat Rooms:** Create and join public chat rooms
3. **Real-time Messages:** Send/receive messages instantly via WebSocket
4. **Message History:** Load previous messages when joining room
5. **Online Status:** See who's currently online in a room

---

## Non-Goals

**Will NOT build:**
- Direct messages (rooms only)
- File/image sharing
- Message editing or deletion
- Message reactions or threads
- Voice or video chat
- Push notifications
- Admin moderation tools
- Message search
- End-to-end encryption

**Will NOT use:**
- Third-party auth providers (OAuth, etc.)
- External chat services
- Redis for pub/sub (use in-memory)
- Complex ORM (use simple queries)

---

## Technical Constraints

| | |
|---|---|
| **Frontend** | Next.js 14, TypeScript, Tailwind CSS |
| **Backend** | Next.js API Routes |
| **Real-time** | Socket.io |
| **Database** | SQLite with Prisma |
| **Auth** | NextAuth.js with credentials |
| **Testing** | Jest, Playwright for E2E |
| **Deploy** | Self-hosted (Docker) |

---

## Success Criteria

- [ ] Users can register and log in
- [ ] Users can create chat rooms
- [ ] Messages appear instantly for all room members
- [ ] Message history loads on room join
- [ ] Online users list updates in real-time
- [ ] Works with 50 concurrent users
- [ ] All tests pass

---

## Implementation Phases

### Phase 1: Project Setup & Auth
**Goal:** Set up project with authentication

**Tasks:**
1. Create Next.js project with TypeScript
2. Set up Prisma with SQLite
3. Create User model (id, username, passwordHash)
4. Implement NextAuth with credentials provider
5. Create login/register pages

**Verification:**
```bash
npm run dev
# App runs on http://localhost:3000

# Register new user → Success
# Login → Redirects to home
# Access protected route → Works when logged in
```

**Deliverables:** Project with working auth

---

### Phase 2: Room Management
**Goal:** Create and list chat rooms

**Tasks:**
1. Create Room model (id, name, createdBy)
2. Create RoomMember model (userId, roomId, joinedAt)
3. Build room list page
4. Build create room form
5. Build room detail page (placeholder for messages)

**Verification:**
```
1. Create room "General"
2. Room appears in list
3. Click room → Opens room page
4. Other users can see and join room
```

**Deliverables:** Room CRUD functionality

---

### Phase 3: Real-time Messaging
**Goal:** Implement WebSocket chat

**Tasks:**
1. Set up Socket.io server
2. Create Message model (id, content, userId, roomId, createdAt)
3. Implement send message (emit to room)
4. Implement receive message (update UI)
5. Load message history on room join

**Verification:**
```
1. User A sends message in Room 1
2. User B sees message instantly
3. User C joins Room 1 → sees message history
4. Messages persist after page refresh
```

**Deliverables:** Real-time chat working

---

### Phase 4: Online Status & Polish
**Goal:** Show online users and polish UX

**Tasks:**
1. Track connected users per room
2. Broadcast join/leave events
3. Display online users list
4. Add typing indicator
5. Add timestamps and user avatars

**Verification:**
```
1. Join room → Online count increases
2. Leave room → Online count decreases
3. See "UserX is typing..." indicator
4. Messages show timestamp and username
```

**Deliverables:** Online presence features

---

### Phase 5: Testing & Deployment
**Goal:** Test and deploy

**Tasks:**
1. Write Jest unit tests for API routes
2. Write Playwright E2E tests for chat flow
3. Create Dockerfile
4. Create docker-compose.yml
5. Document deployment steps

**Verification:**
```bash
npm test
# All tests pass

docker-compose up
# App runs in container

# Load test with 50 concurrent connections
# Messages still instant
```

**Deliverables:** Deployed, tested application

---

## Database Schema

```prisma
model User {
  id           String   @id @default(uuid())
  username     String   @unique
  passwordHash String
  createdAt    DateTime @default(now())
  messages     Message[]
  rooms        RoomMember[]
}

model Room {
  id        String   @id @default(uuid())
  name      String
  createdAt DateTime @default(now())
  members   RoomMember[]
  messages  Message[]
}

model RoomMember {
  userId   String
  roomId   String
  joinedAt DateTime @default(now())
  user     User     @relation(fields: [userId], references: [id])
  room     Room     @relation(fields: [roomId], references: [id])
  @@id([userId, roomId])
}

model Message {
  id        String   @id @default(uuid())
  content   String
  userId    String
  roomId    String
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id])
  room      Room     @relation(fields: [roomId], references: [id])
}
```

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Hash passwords before storing | Before changing database schema | Store plain text passwords |
| Validate message content length | Before adding new features | Send messages without auth |
| Escape HTML in messages (XSS) | Before changing Socket.io events | Expose user passwords |
| Handle WebSocket disconnects | | Delete message history |
