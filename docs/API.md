# API Documentation

## Overview

This document outlines the planned API structure for the Soccer Data Scraper project. The API will provide programmatic access to scraped soccer match data.

## Base URL

```
https://api.soccer-scraper.com/v1
```

## Authentication

### API Key Authentication
```http
Authorization: Bearer YOUR_API_KEY
```

### Rate Limiting
- **Free Tier**: 100 requests/hour
- **Pro Tier**: 1000 requests/hour
- **Enterprise**: Custom limits

## Endpoints

### 1. Matches

#### Get All Matches
```http
GET /matches
```

**Query Parameters:**
- `league` (string): Filter by league name
- `country` (string): Filter by country
- `date_from` (date): Start date (YYYY-MM-DD)
- `date_to` (date): End date (YYYY-MM-DD)
- `team` (string): Filter by team name
- `limit` (integer): Number of results (max 1000)
- `offset` (integer): Pagination offset

**Response:**
```json
{
  "data": [
    {
      "id": "12345",
      "country": "England",
      "league": "Premier League",
      "date": "2023-12-25",
      "home_team": "Manchester United",
      "away_team": "Liverpool",
      "home_goals1": 1,
      "home_goals2": 1,
      "away_goals1": 0,
      "away_goals2": 1,
      "total_home_goals": 2,
      "total_away_goals": 1,
      "created_at": "2023-12-25T10:30:00Z",
      "updated_at": "2023-12-25T10:30:00Z"
    }
  ],
  "pagination": {
    "total": 15000,
    "limit": 100,
    "offset": 0,
    "has_next": true,
    "has_prev": false
  }
}
```

#### Get Match by ID
```http
GET /matches/{id}
```

**Response:**
```json
{
  "data": {
    "id": "12345",
    "country": "England",
    "league": "Premier League",
    "date": "2023-12-25",
    "home_team": "Manchester United",
    "away_team": "Liverpool",
    "home_goals1": 1,
    "home_goals2": 1,
    "away_goals1": 0,
    "away_goals2": 1,
    "total_home_goals": 2,
    "total_away_goals": 1,
    "created_at": "2023-12-25T10:30:00Z",
    "updated_at": "2023-12-25T10:30:00Z"
  }
}
```

### 2. Leagues

#### Get All Leagues
```http
GET /leagues
```

**Response:**
```json
{
  "data": [
    {
      "id": "1",
      "name": "Premier League",
      "country": "England",
      "match_count": 380,
      "last_updated": "2023-12-25T10:30:00Z"
    }
  ]
}
```

#### Get League by ID
```http
GET /leagues/{id}
```

**Response:**
```json
{
  "data": {
    "id": "1",
    "name": "Premier League",
    "country": "England",
    "match_count": 380,
    "last_updated": "2023-12-25T10:30:00Z",
    "seasons": [
      {
        "year": "2023-24",
        "match_count": 380,
        "start_date": "2023-08-11",
        "end_date": "2024-05-19"
      }
    ]
  }
}
```

### 3. Teams

#### Get All Teams
```http
GET /teams
```

**Query Parameters:**
- `league` (string): Filter by league
- `country` (string): Filter by country
- `limit` (integer): Number of results
- `offset` (integer): Pagination offset

**Response:**
```json
{
  "data": [
    {
      "id": "1",
      "name": "Manchester United",
      "country": "England",
      "league": "Premier League",
      "match_count": 38,
      "home_matches": 19,
      "away_matches": 19
    }
  ],
  "pagination": {
    "total": 500,
    "limit": 100,
    "offset": 0,
    "has_next": true,
    "has_prev": false
  }
}
```

#### Get Team by ID
```http
GET /teams/{id}
```

**Response:**
```json
{
  "data": {
    "id": "1",
    "name": "Manchester United",
    "country": "England",
    "league": "Premier League",
    "match_count": 38,
    "home_matches": 19,
    "away_matches": 19,
    "recent_matches": [
      {
        "id": "12345",
        "date": "2023-12-25",
        "opponent": "Liverpool",
        "home_goals": 2,
        "away_goals": 1,
        "result": "W"
      }
    ]
  }
}
```

### 4. Statistics

#### Get League Statistics
```http
GET /statistics/leagues/{league_id}
```

**Response:**
```json
{
  "data": {
    "league_id": "1",
    "league_name": "Premier League",
    "total_matches": 380,
    "total_goals": 1084,
    "average_goals_per_match": 2.85,
    "home_wins": 45.2,
    "away_wins": 28.1,
    "draws": 26.7,
    "top_scoring_team": "Manchester City",
    "most_clean_sheets": "Arsenal"
  }
}
```

#### Get Team Statistics
```http
GET /statistics/teams/{team_id}
```

**Response:**
```json
{
  "data": {
    "team_id": "1",
    "team_name": "Manchester United",
    "matches_played": 38,
    "wins": 23,
    "draws": 8,
    "losses": 7,
    "goals_for": 58,
    "goals_against": 43,
    "goal_difference": 15,
    "points": 77,
    "home_record": {
      "matches": 19,
      "wins": 12,
      "draws": 4,
      "losses": 3
    },
    "away_record": {
      "matches": 19,
      "wins": 11,
      "draws": 4,
      "losses": 4
    }
  }
}
```

### 5. Search

#### Search Matches
```http
GET /search/matches
```

**Query Parameters:**
- `q` (string): Search query
- `type` (string): Search type (team, league, date)

**Response:**
```json
{
  "data": [
    {
      "id": "12345",
      "country": "England",
      "league": "Premier League",
      "date": "2023-12-25",
      "home_team": "Manchester United",
      "away_team": "Liverpool",
      "score": "2-1",
      "relevance_score": 0.95
    }
  ],
  "total": 150,
  "query": "Manchester United Liverpool"
}
```

## Error Responses

### Standard Error Format
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid date format",
    "details": {
      "field": "date_from",
      "value": "2023-13-01"
    }
  }
}
```

### Common Error Codes
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `429` - Rate Limit Exceeded
- `500` - Internal Server Error

## Data Models

### Match
```typescript
interface Match {
  id: string;
  country: string;
  league: string;
  date: string;
  home_team: string;
  away_team: string;
  home_goals1: number;
  home_goals2: number;
  away_goals1: number;
  away_goals2: number;
  total_home_goals: number;
  total_away_goals: number;
  created_at: string;
  updated_at: string;
}
```

### League
```typescript
interface League {
  id: string;
  name: string;
  country: string;
  match_count: number;
  last_updated: string;
  seasons?: Season[];
}
```

### Team
```typescript
interface Team {
  id: string;
  name: string;
  country: string;
  league: string;
  match_count: number;
  home_matches: number;
  away_matches: number;
}
```

## SDKs and Libraries

### Python SDK
```python
from soccer_scraper import SoccerAPI

api = SoccerAPI(api_key="your_api_key")

# Get matches
matches = api.matches.get(
    league="Premier League",
    date_from="2023-12-01",
    date_to="2023-12-31"
)

# Get team statistics
stats = api.statistics.team(team_id="1")
```

### JavaScript SDK
```javascript
import { SoccerAPI } from 'soccer-scraper-sdk';

const api = new SoccerAPI({ apiKey: 'your_api_key' });

// Get matches
const matches = await api.matches.get({
  league: 'Premier League',
  dateFrom: '2023-12-01',
  dateTo: '2023-12-31'
});

// Get team statistics
const stats = await api.statistics.team('1');
```

## Webhooks

### Match Updates
```http
POST /webhooks/matches
```

**Payload:**
```json
{
  "event": "match.created",
  "data": {
    "match_id": "12345",
    "home_team": "Manchester United",
    "away_team": "Liverpool",
    "date": "2023-12-25"
  },
  "timestamp": "2023-12-25T10:30:00Z"
}
```

### Webhook Events
- `match.created` - New match added
- `match.updated` - Match data updated
- `league.updated` - League data updated
- `scraping.completed` - Scraping job completed
- `scraping.failed` - Scraping job failed

## Rate Limiting

### Headers
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
```

### Rate Limit Exceeded Response
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded",
    "retry_after": 3600
  }
}
```

## Versioning

API versioning is handled through the URL path:
- `v1` - Current stable version
- `v2` - Future version (when available)

## Deprecation Policy

- Deprecated endpoints will be announced 6 months in advance
- Deprecated features will continue to work for 12 months
- Migration guides will be provided for all breaking changes