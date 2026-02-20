# PRD Example 24: Presentation Poll Tool (Mentimeter Clone)

> **Difficulty:** Intermediate | **Project Type:** Web App | **Time:** 1-2 weeks

---

## Overview

| | |
|---|---|
| **What** | A web app for creating interactive polls and Q&A during presentations |
| **Who** | Teachers, speakers, and meeting hosts who want audience engagement |
| **Why** | Enables real-time audience participation without expensive enterprise tools |

---

## Core Features (MVP)

1. **Create Poll:** Multiple choice questions with 2-5 options
2. **Share Code:** Generate 6-digit room code for audience to join
3. **Live Voting:** Audience votes on phones, results update in real-time
4. **Display Results:** Full-screen bar chart of live results
5. **Q&A Mode:** Audience submits questions, presenter sees list

---

## Non-Goals

**Will NOT build:**
- User accounts or saved presentations
- Word clouds or open-ended response visualizations
- Quiz mode with correct answers
- Timer or countdown features
- Audience analytics or reports
- PDF/PowerPoint export
- Custom branding or themes
- Multiple slides/questions per session

**Will NOT use:**
- Database (session data in memory, lost on restart)
- User authentication
- External presentation integrations
- Payment processing

---

## Technical Constraints

| | |
|---|---|
| **Frontend** | Next.js 14, TypeScript, Tailwind CSS |
| **Real-time** | Socket.io |
| **Data Storage** | In-memory (Map object) |
| **Charts** | Chart.js |
| **Testing** | Playwright for E2E |
| **Deploy** | Vercel (with serverless limitations noted) |

---

## Success Criteria

- [ ] Presenter creates poll with question and options
- [ ] System generates unique 6-digit room code
- [ ] Audience joins via room code on mobile
- [ ] Votes appear in real-time on presenter screen
- [ ] Bar chart updates live as votes come in
- [ ] Q&A mode shows submitted questions
- [ ] Works with 50+ simultaneous voters
- [ ] All E2E tests pass

---

## Implementation Phases

### Phase 1: Project Setup & Poll Creation
**Goal:** Set up project and create poll flow

**Tasks:**
1. Create Next.js project with TypeScript
2. Build presenter home page with "Create Poll" button
3. Build poll creation form (question + 2-5 options)
4. Generate unique 6-digit room code
5. Store session in memory Map

**Verification:**
```
1. Visit home page → Click "Create Poll"
2. Enter: "What's your favorite language?"
3. Add options: Python, JavaScript, Go, Rust
4. Click Create → Room code shown: "847291"
5. Poll data stored in memory
```

**Deliverables:** Poll creation working

---

### Phase 2: Audience Voting Page
**Goal:** Build mobile voting experience

**Tasks:**
1. Create `/join` page with room code input
2. Create `/vote/[code]` page showing poll
3. Display question and options as large buttons
4. Submit vote via API
5. Show "Vote submitted" confirmation

**Verification:**
```
1. On phone, visit /join
2. Enter code: 847291
3. See poll: "What's your favorite language?"
4. Tap "Python" → Vote submitted
5. Can't vote again (stored in sessionStorage)
```

**Deliverables:** Mobile voting flow

---

### Phase 3: Real-time Results
**Goal:** Live updating results display

**Tasks:**
1. Set up Socket.io server
2. Emit vote event when vote submitted
3. Presenter page listens for votes
4. Update Chart.js bar chart in real-time
5. Show vote count per option

**Verification:**
```
1. Presenter sees results page with bar chart
2. Audience member votes for Python
3. Chart instantly updates: Python 1 vote
4. Another vote for JavaScript
5. Chart shows: Python 1, JavaScript 1
```

**Deliverables:** Real-time chart updates

---

### Phase 4: Q&A Mode
**Goal:** Audience question submission

**Tasks:**
1. Add "Q&A" mode option when creating session
2. Audience sees text input instead of options
3. Questions appear on presenter screen
4. Presenter can mark questions as "answered"
5. Questions sorted by submission time

**Verification:**
```
1. Create Q&A session → Code generated
2. Audience joins and types question
3. "How does this work?" appears on presenter screen
4. More questions stack below
5. Presenter clicks checkmark → Question grayed out
```

**Deliverables:** Q&A functionality

---

### Phase 5: Polish & Deploy
**Goal:** Full-screen display and deployment

**Tasks:**
1. Add full-screen presenter mode
2. Add "End Session" button
3. Style for projection (large fonts, high contrast)
4. Write Playwright E2E tests
5. Deploy to Vercel (note: in-memory won't persist)

**Verification:**
```bash
# Full screen mode fills display
# Results visible from back of room

npx playwright test
# All tests pass

vercel --prod
# App deployed

# Note in docs: Sessions lost on serverless cold start
```

**Deliverables:** Deployed presentation tool

---

## Data Schema

```typescript
interface PollOption {
  id: string;
  text: string;
  votes: number;
}

interface Poll {
  code: string;
  question: string;
  options: PollOption[];
  createdAt: Date;
  mode: 'poll' | 'qa';
}

interface Question {
  id: string;
  text: string;
  answered: boolean;
  submittedAt: Date;
}

interface QASession {
  code: string;
  title: string;
  questions: Question[];
  createdAt: Date;
  mode: 'qa';
}

// In-memory storage
const sessions = new Map<string, Poll | QASession>();
```

---

## URL Structure

```
/ → Home page (Create Poll / Create Q&A)
/create → Poll/Q&A creation form
/present/[code] → Presenter view with results
/join → Audience join page
/vote/[code] → Audience voting page
/qa/[code] → Audience Q&A submission page
```

---

## UI Sketches

**Presenter View (Poll Mode):**
```
┌────────────────────────────────────────────────────┐
│  What's your favorite programming language?        │
│                                              847291│
├────────────────────────────────────────────────────┤
│                                                    │
│  Python      ████████████████████████  45 (45%)   │
│  JavaScript  ████████████████  32 (32%)           │
│  Go          ██████  12 (12%)                     │
│  Rust        █████  11 (11%)                      │
│                                                    │
│                              Total votes: 100      │
└────────────────────────────────────────────────────┘
```

**Audience View (Mobile):**
```
┌──────────────────┐
│ Join: 847291     │
├──────────────────┤
│                  │
│ What's your      │
│ favorite         │
│ programming      │
│ language?        │
│                  │
│ ┌──────────────┐ │
│ │   Python     │ │
│ └──────────────┘ │
│ ┌──────────────┐ │
│ │  JavaScript  │ │
│ └──────────────┘ │
│ ┌──────────────┐ │
│ │     Go       │ │
│ └──────────────┘ │
│ ┌──────────────┐ │
│ │    Rust      │ │
│ └──────────────┘ │
└──────────────────┘
```

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Generate unique 6-digit codes | Before adding new question types | Use database (in-memory only) |
| Prevent duplicate votes (sessionStorage) | Before changing result display | Store personal information |
| Show vote counts and percentages | Before adding timer features | Add user authentication |
| Make results visible from distance | | Allow editing votes after submission |
