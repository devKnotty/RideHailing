
RIDE-HAILING BACKEND
Project Sprint Plan
2 Developers  •  2–3 hrs/day  •  12 Weeks  •  6 Sprints


TOTAL SPRINTS
6	DURATION
12 Weeks	DAILY HOURS
2–3 hrs	TEAM SIZE
2 Devs

What You Will Build
A production-grade backend for a ride-hailing platform (like Uber/Ola) with 4 microservices, each tackling a distinct hard problem in Java backend engineering.

📍 LocationService
Real-time driver tracking via Redis geospatial. Handles 5000+ location events/sec reactively.	🎯 MatchingService
Race-condition-free driver assignment. Lock-free concurrency with Java 21 Virtual Threads.	🚗 TripService
Complete trip lifecycle via Kafka event pipeline. Saga pattern for distributed transactions.	🔔 NotificationSvc
Async event-driven notifications. Circuit breaker + retry patterns via Resilience4j.


Tech Stack
Layer	Technology	What You Learn
Language	Java 21 (Virtual Threads, Records, Sealed Classes)	Modern Java features in production context
Framework	Spring Boot 3 + WebFlux	Reactive non-blocking programming
Async / Events	Apache Kafka + Project Reactor	Event-driven architecture, backpressure
Cache / Geo	Redis + Redisson client	Geospatial queries, TTL, distributed cache
Primary DB	PostgreSQL + HikariCP + Flyway	Connection pooling, migrations, read replicas
History DB	Apache Cassandra	Wide-column store, partition key design
Resilience	Resilience4j	Circuit breaker, rate limiter, retry
Observability	OpenTelemetry + Jaeger + Grafana	Distributed tracing, metrics dashboards
Testing / Load	JUnit 5 + TestContainers + Gatling	Integration tests, load testing
Infra	Docker + Docker Compose	Local multi-service environment


Sprint Timeline at a Glance
Sprint	Theme	Primary Focus	Deliverable
S1 · Wk 1–2	Foundation	Project setup, domain modeling, REST APIs	4 services running, full CRUD, Swagger docs
S2 · Wk 3–4	Location	Redis geo, reactive WebFlux, backpressure	Live driver tracking at 5000 events/sec
S3 · Wk 5–6	Concurrency	Locks, Virtual Threads, optimistic locking	Zero duplicate matches under 1000 concurrent req
S4 · Wk 7–8	Events	Kafka pipeline, Saga, idempotency, DLQ	Full trip lifecycle event-driven
S5 · Wk 9–10	Persistence	CQRS, Cassandra, read replicas, HikariCP	Polyglot DB, CQRS working, query optimized
S6 · Wk 11–12	Production	Tracing, circuit breakers, load test, demo	Grafana dashboard, Jaeger traces, load test report

Detailed Sprint Plans
Each sprint runs for 2 weeks (10 working days). Daily bandwidth: 2–3 hours per developer. Day numbers below represent working days within each sprint.

Sprint 1 (Week 1–2): Project Setup & Core Domain Modeling
🎯 Sprint Goal: Monorepo scaffolded, all services skeletons running, core entities defined, Docker environment working
🛠 Tech Focus: Spring Boot 3, Java 21, Docker Compose, PostgreSQL, MapStruct, Lombok
📅 Sprint 1 (Week 1–2): Project Setup & Core Domain Modeling
Day	Dev 1 Tasks	Dev 2 Tasks	End-of-Day Goal
1	Init monorepo, multi-module Maven/Gradle setup	Setup Docker Compose (Postgres, Redis, Kafka, Zookeeper)	docker-compose up runs all infra locally
2	Scaffold 4 service modules: matching, location, trip, notification	Setup shared-lib module (common DTOs, exceptions, constants)	All 4 services start with /health endpoint
3	Design core entities: Driver, Rider, Trip (JPA + Flyway migrations)	Design DriverLocation, MatchRequest, TripEvent entities	ERD finalized, tables created via Flyway
4	Implement Repository layer + basic CRUD for Driver & Rider	Implement Repository layer for Trip & MatchRequest	CRUD APIs working for all entities
5	Driver & Rider REST APIs with validation (Jakarta Bean Validation)	Trip REST APIs + global exception handler + error DTOs	Postman collection ready, all APIs tested
6	Add Spring Security skeleton (JWT structure, no auth logic yet)	Add Swagger/OpenAPI docs to all 4 services	API docs accessible at /swagger-ui
7	Sprint review, integration smoke test, fix issues, push to Git	Write README per service, document setup steps	Code merged to main, demo-ready

Sprint 2 (Week 3–4): Location Service & Redis Geospatial
🎯 Sprint Goal: Real-time driver location tracking working end-to-end with Redis geospatial queries
🛠 Tech Focus: Redis GEOADD/GEORADIUS, Redisson, WebFlux, Spring Data Redis
📅 Sprint 2 (Week 3–4): Location Service & Redis Geospatial
Day	Dev 1 Tasks	Dev 2 Tasks	End-of-Day Goal
1	Integrate Redisson client in LocationService, configure Redis connection pool	Design DriverLocationDTO, write Redis key schema (driver:location:{id})	Redis connectivity confirmed, key design documented
2	Implement GEOADD: driver location update endpoint (POST /location)	Implement GEORADIUS query: findNearbyDrivers(lat, lng, radiusKm)	Location stored and queried from Redis
3	Add location expiry (TTL): driver removed if no ping in 30s	Add driver status filter in geo query (only AVAILABLE drivers)	Stale drivers auto-expire from geo index
4	WebFlux: convert LocationService to reactive (Mono/Flux)	Add backpressure handling on location update stream	Non-blocking endpoints, reactive pipeline
5	Load test: simulate 500 drivers pinging location every 2s (Gatling)	Observe Redis memory under load, tune connection pool settings	Benchmark report: ops/sec, latency p99
6	Implement location history (last 10 positions) stored in Postgres async	Write scheduled cleanup job for stale location data	Location history persisted without blocking main flow
7	Sprint review, refactor hot paths from profiling, document Redis schema	Write integration tests for geo queries with TestContainers	All location tests green, TestContainers running

Sprint 3 (Week 5–6): Matching Service & Concurrency Deep Dive
🎯 Sprint Goal: Race-condition-free driver matching under high concurrency — the hardest sprint
🛠 Tech Focus: ReentrantLock, ConcurrentHashMap, Optimistic Locking, Java 21 Virtual Threads, JFR
📅 Sprint 3 (Week 5–6): Matching Service & Concurrency Deep Dive
Day	Dev 1 Tasks	Dev 2 Tasks	End-of-Day Goal
1	Implement naive matching (no locking) — intentionally broken baseline	Write concurrent test: 100 threads, same driver, first-come-first-served	Reproduce the race condition, document it
2	Fix with pessimistic locking (synchronized) — measure throughput hit	Fix with ReentrantLock — compare throughput vs synchronized	Benchmark: synchronized vs ReentrantLock
3	Implement optimistic locking with @Version in JPA on Driver entity	Handle OptimisticLockException: retry logic with exponential backoff	Optimistic locking working, retries handled
4	Implement lock-free matching using ConcurrentHashMap + compareAndSet	Profile all 4 approaches with JFR (Java Flight Recorder)	Performance comparison report across 4 strategies
5	Switch matching service thread pool to Java 21 Virtual Threads	Benchmark Virtual Threads vs platform threads under 1000 concurrent matches	Virtual thread benchmark complete
6	Implement match timeout: if driver doesn't accept in 30s, re-match	Add match scoring algorithm (distance + driver rating weight)	Smart matching with timeout and scoring
7	Sprint review, write detailed concurrency doc explaining all approaches	Stress test final implementation: 1000 concurrent match requests	Zero duplicate matches under max load

Sprint 4 (Week 7–8): Kafka Event Pipeline & Trip Lifecycle
🎯 Sprint Goal: Complete trip lifecycle (request → match → start → complete → pay) fully event-driven via Kafka
🛠 Tech Focus: Apache Kafka, Consumer Groups, Saga Pattern, Idempotency Keys, Dead Letter Queue
📅 Sprint 4 (Week 7–8): Kafka Event Pipeline & Trip Lifecycle
Day	Dev 1 Tasks	Dev 2 Tasks	End-of-Day Goal
1	Setup Kafka topics: ride.requested, driver.matched, trip.started, trip.completed	Configure consumer groups per service, set partition count = 3	Topics created, consumers connected
2	TripService: produce ride.requested event when rider books	MatchingService: consume ride.requested, produce driver.matched	Event flows: booking triggers matching
3	TripService: consume driver.matched, update trip state, produce trip.started	NotificationService: consume all events, send push notification stubs	Full event chain: request → match → start
4	Implement idempotency keys on all Kafka consumers (dedup on event ID)	Implement Dead Letter Queue (DLQ) for failed message processing	Exactly-once processing, failed msgs in DLQ
5	Implement Saga: trip cancellation flow (compensating transactions)	Implement Saga: payment failure rollback (refund ride, re-queue driver)	Saga compensation working for 2 failure scenarios
6	Add Kafka consumer lag monitoring with Micrometer metrics	Simulate consumer crash mid-trip — verify recovery and state consistency	Crash recovery test passing, metrics visible
7	Sprint review, draw complete event flow diagram, update docs	End-to-end integration test: full trip lifecycle via events	Full trip e2e test passing

Sprint 5 (Week 9–10): DB Scaling, CQRS & Distributed Transactions
🎯 Sprint Goal: Polyglot persistence working — right DB for each access pattern, CQRS implemented
🛠 Tech Focus: CQRS, PostgreSQL Read Replicas, Cassandra, HikariCP tuning, 2PC vs Saga
📅 Sprint 5 (Week 9–10): DB Scaling, CQRS & Distributed Transactions
Day	Dev 1 Tasks	Dev 2 Tasks	End-of-Day Goal
1	Setup Cassandra for ride history, design partition key (rider_id + month)	Implement TripHistoryRepository (write to Cassandra post-trip-complete)	Ride history writes to Cassandra
2	Implement CQRS in TripService: separate Command (write) and Query (read) models	Build TripQueryService: reads from denormalized read model in Postgres	CQRS working — commands and queries separated
3	Configure Postgres read replica in Docker, route read queries to replica	Implement HikariCP separate pools: write pool → primary, read pool → replica	Read/write split with separate connection pools
4	Implement distributed transaction problem: atomically update Redis + Postgres on trip start	Compare approaches: 2PC (why it fails) vs Saga (why it works here)	Document tradeoffs, implement Saga solution
5	Add database-level indexes for hot query paths (driver lookup, trip history)	Run EXPLAIN ANALYZE on top 5 queries, fix N+1 issues with JOIN FETCH	Query performance report, slow queries fixed
6	Implement database connection pool stress test (exhaust pool, observe behavior)	Add HikariCP metrics to Micrometer + Grafana dashboard	DB metrics visible in Grafana
7	Sprint review, finalize data access layer, document DB choices rationale	Integration tests for CQRS, Cassandra writes, read replica routing	All DB tests green

Sprint 6 (Week 11–12): Observability, Resilience & Polish
🎯 Sprint Goal: Production-grade observability, circuit breakers, distributed tracing, resume-worthy demo
🛠 Tech Focus: OpenTelemetry, Jaeger, Resilience4j, Micrometer, Grafana, Gatling final load test
📅 Sprint 6 (Week 11–12): Observability, Resilience & Polish
Day	Dev 1 Tasks	Dev 2 Tasks	End-of-Day Goal
1	Add OpenTelemetry Java agent to all 4 services, export traces to Jaeger	Instrument Kafka producer/consumer with trace context propagation	One trace spans all 4 services in Jaeger UI
2	Add Resilience4j Circuit Breaker on MatchingService → LocationService calls	Configure fallback: if LocationService is down, use last known location	Circuit opens on failure, fallback activates
3	Add Resilience4j Rate Limiter on ride booking endpoint (50 req/s per user)	Add Retry with exponential backoff on Kafka publish failures	Rate limiting and retry working
4	Add Micrometer custom metrics: match_latency_ms, concurrent_trips, queue_depth	Build Grafana dashboard: 4 golden signals (latency, traffic, errors, saturation)	Live Grafana dashboard showing all metrics
5	Final Gatling load test: 10,000 concurrent virtual users, 30 min sustained	Monitor system under load: find and fix bottlenecks	Load test report: p50/p95/p99 latencies
6	Write architecture decision records (ADRs) for top 5 design decisions	Record a 5-min demo video: full trip flow + Jaeger trace + Grafana	Demo video + ADRs complete
7	Final review: clean up code, update README with architecture diagram	Update GitHub README with tech stack badge, system diagram, how to run	GitHub repo interview-ready


Recommended Daily Routine (2–3 hrs)
Time Block	Duration	Activity
Start	15 min	Sync: brief standup — what did I do yesterday, what am I doing today, any blockers?
Core Work	90–120 min	Deep focus coding — no interruptions. Each dev works on their assigned tasks.
Review	20 min	Code review each other's work, pair on any blockers, update task status.
Wrap-up	10 min	Push code to Git, update notes, jot down questions for next session.


Dev Split Strategy
The two developers are split by service ownership — not by frontend/backend. This ensures both devs deeply learn all 4 hard areas.

Dev 1 — Core Systems Owner

•	Primary: MatchingService (concurrency focus)
•	Secondary: LocationService (reactive/Redis)
•	Cross-cutting: Docker infra, CI setup
•	Load testing with Gatling
•	Architecture decision records (ADRs)	Dev 2 — Data & Events Owner

•	Primary: TripService (Kafka + Saga)
•	Secondary: NotificationService (resilience)
•	Cross-cutting: Kafka setup, schema registry
•	Observability: Grafana + Jaeger dashboards
•	Integration tests with TestContainers


Definition of Done — Per Sprint
•	All assigned tasks committed to Git with meaningful commit messages
•	At least one integration test per new feature using TestContainers
•	No broken builds — CI passes on the main branch
•	README updated with any new setup instructions
•	Both devs have reviewed and understood each other's code
•	Performance benchmark captured (latency, throughput) where relevant
•	Any architecture decisions documented in an ADR file


What You Can Say in Interviews After This Project

Topic	What You Can Confidently Discuss
Concurrency	Compared 4 locking strategies under load with JFR profiling — chose lock-free CAS for hot path
Virtual Threads	Benchmarked Java 21 Virtual Threads vs platform threads at 1000 concurrent requests
Reactive	Built non-blocking pipeline handling 5000 events/sec with Project Reactor backpressure
Kafka / Saga	Implemented Saga pattern for distributed transactions with DLQ and idempotency keys
DB Scaling	Designed CQRS with separate read replicas and polyglot persistence (Postgres + Cassandra + Redis)
Resilience	Configured circuit breakers, rate limiters, and retries with Resilience4j in production-like setup
Observability	Built end-to-end distributed traces across 4 services with OpenTelemetry and Grafana dashboards

Good luck — build something interview-worthy. 🚀
