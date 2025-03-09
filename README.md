# System-Design-Analysis

## Executive Summary
The URL Shortener Service is a robust solution delivering:
- **Short URL Generation**: Creates unique, concise aliases for long URLs with minimal latency.
- **Fast Redirects**: Achieves sub-millisecond response times using Redis caching, with MySQL as a durable fallback.
- **Detailed Analytics**: Tracks usage patterns (total redirects, daily breakdowns, geo-distribution, user agents) for insights.

### **Value Proposition**
-  High-speed redirects optimized for user experience.
-  Scalable design ready for growth.
-  Analytics to empower data-driven decisions.

This architecture balances performance and scalability, making it suitable for applications like marketing campaigns, link tracking, or any URL management needs.

---

## **Design Considerations**
- **Normalization**: Separates URL mappings and redirect logs to avoid redundancy and simplify analytics.
- **Indexing**: Optimizes query performance for redirects and aggregations.
- **Scalability**: MySQL suits medium-scale deployments; for larger scale, consider sharding or NoSQL

---

## **System Architecture**
### **Components**
- **API Layer**: Laravel-based RESTful endpoints (`/api/shorten`, `/{alias}`, `/api/analytics/{alias}`).
- **Application**: PHP/Laravel app in Docker, handling business logic.
- **Cache**: Redis for low-latency storage of URLs and analytics.
- **Database**: MySQL for persistent data.
- **External APIs**: IP resolution (ip-api.com`) for geo data.
---

## **Caching Strategy**
### **URL Cache**
- **Key**: `short_url:{alias}`
- **TTL**: 24 hours  
- **Purpose**: Speeds up redirects.

### **Analytics Cache**
- **Key**: `analytics:{alias}`
- **TTL**: 1 hour or invalidated on new redirects  
- **Purpose**: Reduces DB load for frequent analytics requests.
---

## **DataBase Schema**
[View the DataBase Schema](https://drawsql.app/teams/haweil-74/diagrams/task/embed)
---

## **Rationale**
- **Redis**: Chosen for its in-memory speed, ideal for read-heavy redirect operations.
- **MySQL**: Provides durability and relational integrity for persistent storage.
- **JSON for Geo**: Flexible format for extensible geo data without schema changes.
---

## **Production Considerations & Improvements**
### **Scalability**
- **Horizontal Scaling**: Deploy multiple Laravel instances behind a load balancer (e.g., Nginx), with Redis clustering and MySQL read replicas.
- **High Traffic**: Use Redis Sentinel for cache failover and sharding for database partitioning.

### **Rate Limiting**
- **Current**: Laravel’s throttle middleware (1000 req/min).
- **Production**: Implement Redis-based rate limiting with sliding window per IP.

### **Monitoring**
- **Integrate Prometheus and Grafana to track**:
  - Redirect latency.
  - Cache hit/miss rates.
  - API request volumes.


### **Geo Data Optimization**
- **Cache IP-to-geo mappings in Redis** to minimize external API calls, improving latency and reducing costs.

---

## **Assumptions**
- **Read-Heavy Workload**: Redirects outnumber URL creations, favoring caching.
- **Network Reliability**: External IP APIs are available (fallback to local IP if unavailable).
- **Single Instance**: Initial design assumes one server; scaling requires additional setup.
