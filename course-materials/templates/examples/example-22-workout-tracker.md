# PRD Example 22: Mobile Workout Tracker

> **Difficulty:** Advanced | **Project Type:** Mobile App | **Time:** 2-3 weeks

---

## Overview

| | |
|---|---|
| **What** | A mobile app for tracking workouts, exercises, and progress |
| **Who** | Fitness enthusiasts who want to log and track their gym sessions |
| **Why** | Provides a simple way to record workouts and see progress over time |

---

## Core Features (MVP)

1. **Exercise Library:** Browse exercises by muscle group
2. **Create Workout:** Build workout from exercises with sets/reps/weight
3. **Log Session:** Record actual performance during workout
4. **History:** View past workouts
5. **Progress Charts:** See strength progression per exercise

---

## Non-Goals

**Will NOT build:**
- Social features or sharing
- Pre-built workout programs
- Video exercise demonstrations
- Rest timer or notifications
- Calorie or nutrition tracking
- Heart rate or wearable integration
- Workout templates or scheduling
- Personal records board

**Will NOT use:**
- External exercise databases
- Push notification services
- Analytics platforms
- Backend server (offline-first)

---

## Technical Constraints

| | |
|---|---|
| **Framework** | React Native + Expo |
| **Language** | TypeScript |
| **Storage** | AsyncStorage (local only) |
| **Charts** | react-native-chart-kit |
| **Navigation** | Expo Router |
| **Testing** | Jest, React Native Testing Library |
| **Deploy** | Expo EAS Build |

---

## Success Criteria

- [ ] Can browse exercises by muscle group
- [ ] Can create custom workout with exercises
- [ ] Can log sets during workout session
- [ ] Can view workout history by date
- [ ] Can see progress chart for each exercise
- [ ] Data persists offline
- [ ] Runs on iOS and Android
- [ ] All tests pass

---

## Implementation Phases

### Phase 1: Project Setup & Exercise Library
**Goal:** Set up Expo project and exercise data

**Tasks:**
1. Create Expo project with TypeScript
2. Define Exercise type (name, muscle group, equipment)
3. Create hardcoded exercise library (50+ exercises)
4. Build exercise browse screen with filtering
5. Set up Expo Router navigation

**Verification:**
```
1. npm start → App runs in Expo Go
2. Browse exercises → Filter by "Chest"
3. See exercises: Bench Press, Incline Press, etc.
4. Search "squat" → Shows matching exercises
```

**Deliverables:** Exercise library screen

---

### Phase 2: Workout Builder
**Goal:** Create and save workouts

**Tasks:**
1. Define Workout type (name, exercises[], created_at)
2. Build "Create Workout" screen
3. Add exercises to workout with set/rep targets
4. Save workout to AsyncStorage
5. List saved workouts on home screen

**Verification:**
```
1. Create "Push Day" workout
2. Add Bench Press (4x8), Shoulder Press (3x10)
3. Save → Appears in workout list
4. Restart app → Workout persists
```

**Deliverables:** Workout creation flow

---

### Phase 3: Workout Logging
**Goal:** Record live workout sessions

**Tasks:**
1. Define WorkoutLog type (workout_id, date, exercises[])
2. Build "Start Workout" screen
3. Show exercises with input for actual weight/reps
4. Allow adding/removing sets
5. Save completed log

**Verification:**
```
1. Start "Push Day" workout
2. Log Bench Press: Set 1: 135lb x 8
3. Add another set
4. Complete workout → Summary shown
5. Log saved with timestamp
```

**Deliverables:** Live workout logging

---

### Phase 4: History & Progress
**Goal:** View history and track progress

**Tasks:**
1. Build History screen with date grouping
2. Tap log to view details
3. Build Progress screen per exercise
4. Show line chart of max weight over time
5. Show total volume trend

**Verification:**
```
1. View History → See logs by week
2. Tap Feb 12 log → See exercise details
3. View Bench Press progress
4. Chart shows: Jan 100lb, Feb 115lb, Mar 125lb
5. Volume trend shows improvement
```

**Deliverables:** History and charts

---

### Phase 5: Polish & Build
**Goal:** Final polish and production build

**Tasks:**
1. Add empty states for no data
2. Add confirmation before deleting workouts
3. Write Jest tests for core logic
4. Build with EAS for iOS and Android
5. Test on physical devices

**Verification:**
```bash
eas build --platform all
# Builds created for iOS and Android

# Install on device → All features work
# No crashes on edge cases
```

**Deliverables:** Production app builds

---

## Data Schema

```typescript
interface Exercise {
  id: string;
  name: string;
  muscleGroup: 'chest' | 'back' | 'legs' | 'shoulders' | 'arms' | 'core';
  equipment: 'barbell' | 'dumbbell' | 'machine' | 'bodyweight' | 'cable';
}

interface WorkoutExercise {
  exerciseId: string;
  sets: number;
  reps: number;
  targetWeight?: number;
}

interface Workout {
  id: string;
  name: string;
  exercises: WorkoutExercise[];
  createdAt: string;
}

interface SetLog {
  weight: number;
  reps: number;
  completed: boolean;
}

interface ExerciseLog {
  exerciseId: string;
  sets: SetLog[];
}

interface WorkoutLog {
  id: string;
  workoutId: string;
  date: string;
  exercises: ExerciseLog[];
  duration: number; // minutes
}
```

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Store data locally with AsyncStorage | Before changing data schema | Use backend server |
| Validate weight/reps are positive numbers | Before adding new features | Require account creation |
| Confirm before deleting workouts | Before changing exercise library | Send data to external services |
| Handle missing data gracefully | | Add social features |
