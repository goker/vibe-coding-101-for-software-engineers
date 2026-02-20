# PRD Example 14: Habit Tracker with Streaks

> **Difficulty:** Intermediate | **Project Type:** Web App | **Time:** 6-8 hours

---

## Overview

| | |
|---|---|
| **What** | A web app for tracking daily habits and maintaining streaks |
| **Who** | Anyone building new habits who wants visual streak motivation |
| **Why** | Simple habit tracking focused on streak visualization to maintain motivation |

---

## Core Features (MVP)

1. **Add Habit:** Create habit with name and target frequency (daily)
2. **Mark Complete:** Check off habits for today
3. **View Streaks:** See current streak count for each habit
4. **Calendar View:** Monthly view showing completed days
5. **Delete Habit:** Remove habits with streak warning

---

## Non-Goals

**Will NOT build:**
- User accounts or authentication
- Habit reminders or notifications
- Weekly/monthly habits (daily only)
- Habit categories or grouping
- Statistics or analytics beyond streaks
- Data export
- Mobile app
- Social features or sharing

**Will NOT use:**
- Backend server (client-side only)
- External APIs
- Database (use localStorage)
- Date libraries (use native Date)

---

## Technical Constraints

| | |
|---|---|
| **Language** | TypeScript |
| **Framework** | Next.js 14 (App Router, static export) |
| **Styling** | Tailwind CSS |
| **Data Storage** | localStorage |
| **Testing** | Jest, React Testing Library |
| **Deploy** | Static export to Vercel |

---

## Success Criteria

- [ ] Can create habits with name
- [ ] Can mark habits complete for today
- [ ] Streak count increases on consecutive days
- [ ] Streak resets if day is missed
- [ ] Calendar shows completed days (green) vs missed (red)
- [ ] Delete warns if streak > 7 days
- [ ] All tests pass
- [ ] Deploys to Vercel as static site

---

## Implementation Phases

### Phase 1: Project Setup & Data Model
**Goal:** Set up Next.js project and habit storage

**Tasks:**
1. Create Next.js project with App Router
2. Configure for static export (`output: 'export'`)
3. Define Habit type: `{id, name, completedDates[], createdAt}`
4. Create localStorage hooks

**Verification:**
```bash
npm run dev
# Opens http://localhost:3000

npm run build
# Creates static export in /out
```

**Deliverables:** Project structure, types, storage

---

### Phase 2: Habit CRUD & Daily Tracking
**Goal:** Create habits and track daily completion

**Tasks:**
1. Build AddHabit form
2. Build HabitCard with today's checkbox
3. Implement toggle completion for today
4. Calculate and display current streak

**Verification:**
```
1. Add habit "Exercise"
2. Check today's checkbox → streak shows "1 day"
3. Refresh page → completion persists
4. (Simulate) Check yesterday → streak shows "2 days"
```

**Deliverables:** Habit cards with streak display

---

### Phase 3: Calendar View
**Goal:** Monthly calendar showing habit history

**Tasks:**
1. Build Calendar component
2. Show current month with day cells
3. Color completed days green, missed days red
4. Navigate between months (prev/next)

**Verification:**
```
1. Click habit to expand calendar
2. See green cells for completed days
3. See red cells for missed days after creation
4. Navigate to previous month → shows history
```

**Deliverables:** Calendar component

---

### Phase 4: Delete & Deploy
**Goal:** Complete app and deploy

**Tasks:**
1. Implement delete with streak warning
2. Write tests for streak calculation
3. Deploy to Vercel
4. Test on deployed URL

**Verification:**
```bash
# Delete habit with 10-day streak:
"Are you sure? You'll lose your 10-day streak!"

npm test
# All tests pass

vercel --prod
# Deploys successfully
```

**Deliverables:** Complete app deployed to Vercel

---

## Streak Calculation Logic

```typescript
function calculateStreak(completedDates: string[]): number {
  // Sort dates descending
  // Count consecutive days from today backwards
  // If today not completed, start from yesterday
  // Return count
}
```

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Store dates as ISO strings (YYYY-MM-DD) | Before changing streak logic | Use backend or database |
| Warn before deleting habits with streaks | Before adding new features | Send notifications |
| Persist immediately on completion toggle | Before changing calendar design | Add authentication |
| Handle timezone correctly for "today" | | Use date libraries |
