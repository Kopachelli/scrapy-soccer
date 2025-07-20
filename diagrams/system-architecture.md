# System Architecture Diagrams

## 1. High-Level System Architecture

```mermaid
graph TB
    subgraph "User Layer"
        A[Developer/User]
        B[API Client]
        C[Web Dashboard]
    end
    
    subgraph "Application Layer"
        D[Scrapy Engine]
        E[MatchSpider]
        F[Data Pipelines]
        G[API Server]
    end
    
    subgraph "Data Layer"
        H[MySQL Database]
        I[Redis Cache]
        J[HTTP Cache]
    end
    
    subgraph "External Services"
        K[stat-football.com]
        L[ScrapingHub]
        M[Monitoring Tools]
    end
    
    A --> D
    B --> G
    C --> G
    D --> E
    E --> K
    E --> F
    F --> H
    F --> I
    D --> J
    G --> H
    G --> I
    E --> L
    D --> M
```

## 2. Data Flow Architecture

```mermaid
flowchart TD
    A[Start Scraping] --> B[Initialize Spider]
    B --> C[Fetch stat-football.com]
    C --> D{Check HTTP Cache}
    D -->|Cache Hit| E[Use Cached Data]
    D -->|Cache Miss| F[Download HTML]
    F --> G[Parse HTML]
    E --> G
    G --> H[Extract Match Data]
    H --> I[Create MatchItem]
    I --> J[Cleaning Pipeline]
    J --> K{Data Valid?}
    K -->|No| L[Drop Item]
    K -->|Yes| M[Database Pipeline]
    M --> N[Store in MySQL]
    N --> O[Update Cache]
    O --> P[Log Results]
    P --> Q{More Pages?}
    Q -->|Yes| C
    Q -->|No| R[Scraping Complete]
```

## 3. Component Interaction

```mermaid
sequenceDiagram
    participant U as User
    participant R as run.py
    participant S as Scrapy Engine
    participant SP as MatchSpider
    participant CP as CleaningPipeline
    participant DP as DatabasePipeline
    participant DB as MySQL Database
    participant C as HTTP Cache
    
    U->>R: Execute spider
    R->>S: Initialize crawler
    S->>SP: Load spider rules
    
    loop For each page
        SP->>C: Check cache
        alt Cache miss
            SP->>stat-football.com: Request page
            stat-football.com->>SP: HTML response
            SP->>C: Store in cache
        else Cache hit
            C->>SP: Return cached data
        end
        
        SP->>SP: Extract match data
        SP->>CP: Send MatchItem
        CP->>CP: Clean and validate
        CP->>DP: Valid item
        DP->>DB: Insert match
        DB->>DP: Confirm insertion
    end
    
    S->>U: Scraping complete
```

## 4. Database Schema

```mermaid
erDiagram
    MATCHES {
        int id PK
        varchar country
        varchar league
        date match_date
        varchar home_team
        varchar away_team
        int home_goals1
        int home_goals2
        int away_goals1
        int away_goals2
        datetime created_at
        datetime updated_at
    }
    
    LEAGUES {
        int id PK
        varchar name
        varchar country
        int match_count
        datetime last_updated
    }
    
    TEAMS {
        int id PK
        varchar name
        varchar country
        varchar league
        int match_count
    }
    
    STATISTICS {
        int id PK
        int match_id FK
        int total_home_goals
        int total_away_goals
        varchar result
        datetime calculated_at
    }
    
    MATCHES ||--o{ STATISTICS : "has"
    LEAGUES ||--o{ MATCHES : "contains"
    TEAMS ||--o{ MATCHES : "plays_in"
```

## 5. Pipeline Architecture

```mermaid
graph LR
    subgraph "Input"
        A[Raw HTML]
    end
    
    subgraph "Extraction"
        B[CSS Selectors]
        C[Data Parsing]
    end
    
    subgraph "Processing"
        D[Cleaning Pipeline]
        E[Validation]
        F[Transformation]
    end
    
    subgraph "Storage"
        G[Database Pipeline]
        H[MySQL Storage]
    end
    
    subgraph "Output"
        I[JSON Export]
        J[CSV Export]
        K[API Response]
    end
    
    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H
    H --> I
    H --> J
    H --> K
```

## 6. Error Handling Flow

```mermaid
flowchart TD
    A[Request Page] --> B{Network Error?}
    B -->|Yes| C[Retry with Backoff]
    C --> D{Max Retries?}
    D -->|No| A
    D -->|Yes| E[Log Error]
    
    B -->|No| F[Parse HTML]
    F --> G{Parse Error?}
    G -->|Yes| H[Skip Page]
    G -->|No| I[Extract Data]
    
    I --> J{Data Valid?}
    J -->|No| K[Drop Item]
    J -->|Yes| L[Process Item]
    
    L --> M{Database Error?}
    M -->|Yes| N[Retry Insert]
    N --> O{Max Retries?}
    O -->|No| M
    O -->|Yes| P[Log Error]
    M -->|No| Q[Success]
```

## 7. Caching Strategy

```mermaid
graph TB
    subgraph "Cache Layers"
        A[HTTP Cache]
        B[Redis Cache]
        C[Database Cache]
    end
    
    subgraph "Cache Policies"
        D[24h HTTP Cache]
        E[1h Redis Cache]
        F[Persistent DB Cache]
    end
    
    subgraph "Cache Operations"
        G[Cache Check]
        H[Cache Store]
        I[Cache Invalidate]
    end
    
    A --> D
    B --> E
    C --> F
    
    G --> A
    G --> B
    G --> C
    
    H --> A
    H --> B
    H --> C
    
    I --> A
    I --> B
    I --> C
```

## 8. Deployment Architecture

```mermaid
graph TB
    subgraph "Development"
        A[Local Python]
        B[Local Database]
        C[Local Cache]
    end
    
    subgraph "Staging"
        D[Staging Server]
        E[Staging DB]
        F[Staging Cache]
    end
    
    subgraph "Production"
        G[Load Balancer]
        H[App Server 1]
        I[App Server 2]
        J[Production DB]
        K[Redis Cluster]
        L[Monitoring]
    end
    
    A --> B
    A --> C
    
    D --> E
    D --> F
    
    G --> H
    G --> I
    H --> J
    I --> J
    H --> K
    I --> K
    H --> L
    I --> L
```

## 9. API Architecture

```mermaid
graph TB
    subgraph "API Gateway"
        A[Rate Limiting]
        B[Authentication]
        C[Request Routing]
    end
    
    subgraph "API Services"
        D[Matches API]
        E[Leagues API]
        F[Teams API]
        G[Statistics API]
    end
    
    subgraph "Data Access"
        H[Database Layer]
        I[Cache Layer]
        J[Search Engine]
    end
    
    A --> B
    B --> C
    C --> D
    C --> E
    C --> F
    C --> G
    
    D --> H
    E --> H
    F --> H
    G --> H
    
    D --> I
    E --> I
    F --> I
    G --> I
    
    D --> J
    E --> J
    F --> J
    G --> J
```

## 10. Monitoring and Observability

```mermaid
graph TB
    subgraph "Application Metrics"
        A[Request Rate]
        B[Response Time]
        C[Error Rate]
        D[Success Rate]
    end
    
    subgraph "Business Metrics"
        E[Matches Scraped]
        F[Leagues Covered]
        G[Data Quality]
        H[Update Frequency]
    end
    
    subgraph "Infrastructure Metrics"
        I[CPU Usage]
        J[Memory Usage]
        K[Disk Usage]
        L[Network I/O]
    end
    
    subgraph "Monitoring Tools"
        M[Prometheus]
        N[Grafana]
        O[Alert Manager]
        P[Log Aggregator]
    end
    
    A --> M
    B --> M
    C --> M
    D --> M
    
    E --> M
    F --> M
    G --> M
    H --> M
    
    I --> M
    J --> M
    K --> M
    L --> M
    
    M --> N
    M --> O
    M --> P
```