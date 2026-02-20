# PRD Example 20: Personal Finance Dashboard

> **Difficulty:** Advanced | **Project Type:** Full Stack Web App | **Time:** 2-3 weeks

---

## Overview

| | |
|---|---|
| **What** | A web dashboard for tracking personal finances with visualizations |
| **Who** | Individuals who want to understand their spending patterns |
| **Why** | Provides insights into spending without sharing data with financial apps |

---

## Core Features (MVP)

1. **Add Transactions:** Log income and expenses with categories
2. **Dashboard View:** Summary cards (income, expenses, net, savings rate)
3. **Charts:** Spending by category (pie), monthly trend (line)
4. **Category Management:** Create and edit spending categories
5. **Monthly Report:** Export month summary as PDF

---

## Non-Goals

**Will NOT build:**
- Bank account integration or import
- Multi-currency support
- Budgeting or goals
- Bill reminders
- Investment tracking
- Multi-user or family sharing
- Mobile app
- Recurring transaction automation

**Will NOT use:**
- External financial APIs
- PDF generation libraries (use browser print)
- Chart libraries beyond Chart.js
- Complex state management (Context is sufficient)

---

## Technical Constraints

| | |
|---|---|
| **Frontend** | React 19, TypeScript, Tailwind CSS |
| **Backend** | FastAPI (Python) |
| **Database** | SQLite with SQLAlchemy |
| **Charts** | Chart.js with react-chartjs-2 |
| **Auth** | JWT tokens (simple implementation) |
| **Testing** | pytest (backend), Vitest (frontend) |
| **Deploy** | Frontend: Vercel, Backend: Railway |

---

## Success Criteria

- [ ] Users can register and log in
- [ ] Can add income/expense transactions
- [ ] Dashboard shows accurate totals
- [ ] Pie chart shows spending by category
- [ ] Line chart shows monthly trends
- [ ] Can export monthly report (browser print to PDF)
- [ ] All tests pass

---

## Implementation Phases

### Phase 1: Backend Setup & Auth
**Goal:** Set up FastAPI with authentication

**Tasks:**
1. Create FastAPI project structure
2. Set up SQLite with SQLAlchemy
3. Create User model and auth endpoints
4. Implement JWT token generation/validation
5. Create protected route middleware

**Verification:**
```bash
uvicorn main:app --reload
# API running on http://localhost:8000

curl -X POST /auth/register -d '{"email": "...", "password": "..."}'
# Returns user created

curl -X POST /auth/login -d '{"email": "...", "password": "..."}'
# Returns JWT token
```

**Deliverables:** FastAPI backend with auth

---

### Phase 2: Transaction CRUD
**Goal:** Implement transaction management

**Tasks:**
1. Create Transaction model (amount, type, category, date, note)
2. Create Category model (name, color, icon)
3. Implement CRUD endpoints for transactions
4. Implement CRUD endpoints for categories
5. Add default categories on user creation

**Verification:**
```bash
curl -X POST /transactions -H "Authorization: Bearer ..." \
  -d '{"amount": 50.00, "type": "expense", "category_id": 1, "note": "Lunch"}'
# Transaction created

curl /transactions?month=2025-02
# Returns transactions for February 2025
```

**Deliverables:** Transaction API

---

### Phase 3: Frontend Dashboard
**Goal:** Build React dashboard with charts

**Tasks:**
1. Create React project with Vite
2. Build auth pages (login, register)
3. Build dashboard layout with summary cards
4. Implement transaction list with add/edit forms
5. Add Chart.js charts (pie for categories, line for trends)

**Verification:**
```
1. Login → Dashboard loads
2. Add transaction → List updates
3. Pie chart shows category breakdown
4. Line chart shows monthly trend
5. Summary cards show correct totals
```

**Deliverables:** Working dashboard UI

---

### Phase 4: Category Management & Filters
**Goal:** Manage categories and filter data

**Tasks:**
1. Build category management page
2. Add date range filter to dashboard
3. Filter transactions by category
4. Add transaction search
5. Implement edit/delete transactions

**Verification:**
```
1. Create new category "Entertainment"
2. Assign transaction to new category
3. Filter by date range → Data updates
4. Filter by category → Shows only matching
5. Delete transaction → Removed from list
```

**Deliverables:** Category management and filters

---

### Phase 5: Reports & Deployment
**Goal:** Export and deploy

**Tasks:**
1. Create monthly report view (print-optimized)
2. Use browser print to PDF
3. Write backend tests (pytest)
4. Write frontend tests (Vitest)
5. Deploy to Vercel + Railway

**Verification:**
```bash
# Click "Export Report"
# Print dialog opens
# Save as PDF → Clean formatted report

pytest
npm test
# All tests pass

# Visit deployed URL → App works
```

**Deliverables:** Deployed application

---

## Database Schema

```python
class User(Base):
    id: int
    email: str
    password_hash: str
    created_at: datetime

class Category(Base):
    id: int
    user_id: int
    name: str
    color: str  # hex color
    type: str   # "income" or "expense"

class Transaction(Base):
    id: int
    user_id: int
    category_id: int
    amount: Decimal  # stored as cents
    type: str        # "income" or "expense"
    date: date
    note: str
    created_at: datetime
```

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Store amounts as integers (cents) | Before changing database schema | Use external financial APIs |
| Validate positive amounts | Before adding new chart types | Store sensitive financial data unencrypted |
| Hash passwords with bcrypt | Before adding new features | Delete transactions without confirmation |
| Use parameterized queries | | Share user data across accounts |
