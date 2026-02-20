# PRD Example 3: Weather Dashboard API

> **Difficulty:** Advanced | **Project Type:** Shipped Product | **Time:** 2-3 weeks

---

# Weather Dashboard API

## Overview

| | |
|---|---|
| **What** | A REST API that aggregates weather data and serves it to a simple dashboard |
| **Who** | Developers building weather-aware applications |
| **Why** | Provide a clean, cached interface to weather data with usage analytics |

## Features

1. **Current Weather:** `GET /weather/{city}` → Returns temperature, conditions, humidity
2. **Caching:** Responses cached for 10 minutes to reduce API calls
3. **Usage Stats:** `GET /stats` → Returns request counts per endpoint (last 24h)
4. **Health Check:** `GET /health` → Returns API status and upstream provider status
5. **Simple Dashboard:** Static HTML page showing weather for 3 preset cities

## Non-Goals

**Will NOT build:**
- User accounts or authentication
- Weather forecasts (current conditions only)
- Historical weather data
- Mobile app or native clients
- Custom city lists per user
- Push notifications or alerts
- Rate limiting (MVP only)

**Will NOT use:**
- Frontend frameworks (React, Vue, etc.)
- ORMs (use raw SQL or simple key-value)
- Kubernetes or complex orchestration
- Multiple weather API providers

## Technical Constraints

| | |
|---|---|
| **Language** | Python 3.11+ |
| **Framework** | FastAPI |
| **Dependencies** | httpx, redis, pytest, uvicorn |
| **Database** | Redis (caching + stats) |
| **External API** | OpenWeatherMap (free tier) |
| **Testing** | pytest, 80% coverage, mocked external calls |
| **Deployment** | Railway or Render (free tier) |

**API Response Format:**
```json
{
  "city": "London",
  "temperature": 15,
  "unit": "celsius",
  "conditions": "cloudy",
  "humidity": 72,
  "cached": true,
  "timestamp": "2025-01-15T10:30:00Z"
}
```

**Error Response:**
```json
{
  "error": "City not found",
  "code": "CITY_NOT_FOUND"
}
```

## Phases

### Phase 1: Core API
**Tasks:** Set up FastAPI project, implement `/weather/{city}` endpoint with OpenWeatherMap.
**Verify:**
```bash
uvicorn main:app --reload
curl http://localhost:8000/weather/london
# Returns JSON with temperature and conditions
```

### Phase 2: Caching
**Tasks:** Add Redis caching. Cache responses for 10 minutes. Add `cached: true/false` to response.
**Verify:**
```bash
curl http://localhost:8000/weather/london  # cached: false
curl http://localhost:8000/weather/london  # cached: true (instant)
```

### Phase 3: Stats & Health
**Tasks:** Track request counts in Redis. Implement `/stats` and `/health` endpoints.
**Verify:**
```bash
curl http://localhost:8000/stats
# Returns: {"weather_requests": 42, "period": "24h"}
curl http://localhost:8000/health
# Returns: {"status": "ok", "redis": "connected", "upstream": "ok"}
```

### Phase 4: Dashboard & Deploy
**Tasks:** Create `static/index.html` with weather for London, NYC, Tokyo. Deploy to Railway.
**Verify:** Visit deployed URL, see weather cards for 3 cities.

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Handle API errors gracefully | Before adding new endpoints | Expose API keys in code |
| Return consistent JSON structure | Before changing response schema | Skip Redis connection handling |
| Log errors with context | Before adding new dependencies | Deploy without testing locally |
| Mock external APIs in tests | Before changing cache duration | Store sensitive data in Redis |

## Environment Variables

```bash
OPENWEATHER_API_KEY=xxx  # Required
REDIS_URL=redis://localhost:6379  # Default for local
PORT=8000  # Default
```
