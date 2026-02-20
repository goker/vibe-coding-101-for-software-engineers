# PRD Example 17: RSS Feed Aggregator API

> **Difficulty:** Intermediate | **Project Type:** REST API | **Time:** 6-8 hours

---

## Overview

| | |
|---|---|
| **What** | A REST API for aggregating and managing RSS feeds |
| **Who** | Developers building news readers or content aggregators |
| **Why** | Provides a unified API to fetch and cache RSS content from multiple sources |

---

## Core Features (MVP)

1. **Add Feed:** `POST /feeds` → Subscribe to RSS feed by URL
2. **List Feeds:** `GET /feeds` → Show all subscribed feeds
3. **Get Articles:** `GET /feeds/{id}/articles` → Get articles from a feed
4. **Refresh Feed:** `POST /feeds/{id}/refresh` → Fetch latest articles
5. **Delete Feed:** `DELETE /feeds/{id}` → Unsubscribe from feed

---

## Non-Goals

**Will NOT build:**
- User accounts or authentication
- Read/unread article tracking
- Article favoriting or bookmarking
- Full article content fetching (only RSS summary)
- OPML import/export
- Push notifications
- Automatic background refresh
- Search across articles

**Will NOT use:**
- Database (use JSON file)
- Background job processors
- Caching services (Redis, etc.)
- Authentication libraries

---

## Technical Constraints

| | |
|---|---|
| **Language** | Node.js 20+ |
| **Framework** | Express.js |
| **Dependencies** | express, rss-parser, uuid |
| **Data Storage** | JSON file at `./data/feeds.json` |
| **Testing** | Jest, supertest |
| **Code Style** | ESLint, Prettier |

---

## Success Criteria

- [ ] `POST /feeds` validates RSS URL and stores feed info
- [ ] `GET /feeds` returns all feeds with article count
- [ ] `GET /feeds/{id}/articles` returns parsed articles
- [ ] `POST /feeds/{id}/refresh` fetches latest articles
- [ ] `DELETE /feeds/{id}` removes feed
- [ ] Handles invalid RSS URLs gracefully
- [ ] All tests pass

---

## Implementation Phases

### Phase 1: Project Setup & Feed Management
**Goal:** Set up Express and feed CRUD

**Tasks:**
1. Create Express project with TypeScript
2. Define Feed type: `{id, url, title, lastRefreshed}`
3. Implement `POST /feeds` with URL validation
4. Implement `GET /feeds` and `DELETE /feeds/{id}`

**Verification:**
```bash
npm run dev
# Server starts on http://localhost:3000

curl -X POST http://localhost:3000/feeds \
  -H "Content-Type: application/json" \
  -d '{"url": "https://hnrss.org/frontpage"}'
# Returns: {"id": "abc123", "title": "Hacker News", ...}

curl http://localhost:3000/feeds
# Returns array of all feeds
```

**Deliverables:** Express server with feed endpoints

---

### Phase 2: RSS Parsing & Articles
**Goal:** Fetch and parse RSS content

**Tasks:**
1. Use rss-parser to fetch feed content
2. Store articles in feed data
3. Implement `GET /feeds/{id}/articles`
4. Parse article: `{title, link, pubDate, summary}`

**Verification:**
```bash
curl http://localhost:3000/feeds/abc123/articles
# Returns:
# {
#   "feed": {"id": "abc123", "title": "Hacker News"},
#   "articles": [
#     {"title": "Show HN: ...", "link": "...", "pubDate": "..."},
#     ...
#   ]
# }
```

**Deliverables:** Article parsing and retrieval

---

### Phase 3: Refresh & Testing
**Goal:** Manual refresh and tests

**Tasks:**
1. Implement `POST /feeds/{id}/refresh`
2. Update lastRefreshed timestamp
3. Handle RSS parsing errors gracefully
4. Write Jest tests with mocked RSS responses

**Verification:**
```bash
curl -X POST http://localhost:3000/feeds/abc123/refresh
# Returns: {"message": "Refreshed", "newArticles": 5}

# Test invalid URL
curl -X POST http://localhost:3000/feeds \
  -d '{"url": "not-a-valid-rss"}'
# Returns: {"error": "Invalid RSS feed URL"}

npm test
# All tests pass
```

**Deliverables:** Complete API with tests

---

## Data Schema

```typescript
interface Feed {
  id: string;
  url: string;
  title: string;
  description?: string;
  lastRefreshed: string; // ISO date
  articles: Article[];
}

interface Article {
  title: string;
  link: string;
  pubDate: string;
  summary?: string;
  guid: string;
}
```

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Validate URL is valid RSS before storing | Before adding new endpoints | Use database |
| Handle RSS parsing errors gracefully | Before changing article schema | Store full article content |
| Return proper HTTP status codes | Before adding authentication | Auto-refresh in background |
| Limit articles per feed (max 50) | | Scrape website content |
