# PRD Example 13: Bookmark Manager with Tags

> **Difficulty:** Intermediate | **Project Type:** Web App | **Time:** 6-8 hours

---

## Overview

| | |
|---|---|
| **What** | A web app for saving and organizing bookmarks with tags |
| **Who** | Anyone who wants to organize web links better than browser bookmarks |
| **Why** | Provides tag-based organization and search that browser bookmarks lack |

---

## Core Features (MVP)

1. **Add Bookmark:** Form to add URL with title and tags
2. **View All:** Grid/list view of all bookmarks
3. **Filter by Tag:** Click a tag to filter bookmarks
4. **Search:** Search bookmarks by title or URL
5. **Delete Bookmark:** Remove bookmarks with confirmation

---

## Non-Goals

**Will NOT build:**
- User accounts or authentication
- Bookmark folders/hierarchies (tags only)
- Browser extension for quick saving
- Favicon fetching
- Link preview/screenshot
- Import/export from browsers
- Sharing or public links
- Mobile-responsive design

**Will NOT use:**
- Backend server (client-side only)
- External APIs
- Database (use localStorage)
- CSS frameworks (custom CSS only)

---

## Technical Constraints

| | |
|---|---|
| **Language** | TypeScript |
| **Framework** | React 19 |
| **Styling** | CSS Modules (no Tailwind, no CSS frameworks) |
| **Data Storage** | localStorage |
| **Build Tool** | Vite |
| **Testing** | Vitest, React Testing Library |
| **Code Style** | ESLint, Prettier |

---

## Success Criteria

- [ ] Can add bookmark with URL, title, and multiple tags
- [ ] Bookmarks display in grid with title, URL preview, tags
- [ ] Clicking tag filters to show only matching bookmarks
- [ ] Search filters bookmarks by title or URL
- [ ] Delete removes bookmark with confirmation
- [ ] Data persists in localStorage across sessions
- [ ] All tests pass

---

## Implementation Phases

### Phase 1: Project Setup & Data Model
**Goal:** Set up React project and bookmark storage

**Tasks:**
1. Create Vite + React + TypeScript project
2. Define Bookmark type: `{id, url, title, tags, createdAt}`
3. Create localStorage hooks for CRUD operations
4. Build basic layout (header, main area)

**Verification:**
```bash
npm run dev
# Opens http://localhost:5173

# In browser console:
localStorage.getItem('bookmarks')
# Shows stored bookmarks array
```

**Deliverables:** Project structure, types, storage hooks

---

### Phase 2: Add & Display Bookmarks
**Goal:** Create and view bookmarks

**Tasks:**
1. Build AddBookmark form component
2. Build BookmarkCard component
3. Build BookmarkGrid component
4. Implement tag input (comma-separated)
5. Auto-extract title from URL if not provided

**Verification:**
```
1. Fill form: URL="https://example.com", Tags="dev, tools"
2. Click "Add Bookmark"
3. Bookmark appears in grid with tags displayed
4. Refresh page - bookmark persists
```

**Deliverables:** Add form and display components

---

### Phase 3: Filter, Search & Delete
**Goal:** Add filtering and deletion

**Tasks:**
1. Build TagFilter component (shows all unique tags)
2. Implement tag click to filter bookmarks
3. Build SearchBar component
4. Implement delete with confirmation modal
5. Write Vitest tests for components

**Verification:**
```
1. Click "dev" tag → only bookmarks with "dev" tag show
2. Type in search → filters by title/URL
3. Click delete → confirmation appears
4. Confirm → bookmark removed

npm test
# All tests pass
```

**Deliverables:** Complete app with tests

---

## Component Structure

```
src/
├── components/
│   ├── AddBookmark.tsx
│   ├── BookmarkCard.tsx
│   ├── BookmarkGrid.tsx
│   ├── TagFilter.tsx
│   ├── SearchBar.tsx
│   └── ConfirmModal.tsx
├── hooks/
│   └── useBookmarks.ts
├── types/
│   └── bookmark.ts
├── App.tsx
└── main.tsx
```

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Validate URL format before saving | Before changing data schema | Use external APIs |
| Show confirmation before delete | Before adding new features | Store data on server |
| Persist to localStorage on every change | Before changing UI layout | Use CSS frameworks |
| Generate unique IDs for bookmarks | | Add authentication |
