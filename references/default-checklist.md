# Default checklist (fallback only)

Use this **only** when the user asks for a general best-practices/engineering
audit and hasn't supplied their own list — e.g. "چک کن چی کم داریم", "audit us
against general best practices", "what should a production app like this have".
If the user pasted or showed their own list, use that instead and ignore this
file entirely — it exists to cover the no-list case, not to replace the user's
intent.

It's grouped by theme so related items can be investigated together (e.g. grep
the same middleware file for CORS + CSRF + rate limiting in one pass), not
because the grouping itself matters for the output — the output still uses the
skill's normal four-bucket format, flat, not grouped by theme.

## Reliability & resilience
Rate Limiting, Circuit Breakers, Timeouts, Retries, Exponential Backoff,
Idempotency, Backpressure, Health Checks, Liveness & Readiness Probes,
Chaos Engineering

## Scaling & infra
Load Balancing, Reverse Proxies, API Gateways, Autoscaling, Horizontal Scaling,
Vertical Scaling, CDN, Edge Caching, Multi-Region Deployments, Service Discovery

## Data & storage
Caching, Cache Invalidation, Database Indexing, Query Optimization, N+1 Queries,
Connection Pooling, Read Replicas, Sharding, Partitioning, Replication,
Optimistic Locking, Pessimistic Locking, Distributed Locks, Database Migrations,
Schema Versioning

## Distributed systems
Leader Election, CAP Theorem, Eventual Consistency, Distributed Transactions,
Saga Pattern, Network Partitions, Clock Skew, Race Conditions, Deadlocks

## Messaging & async
Message Queues, Pub/Sub, Event-Driven Architecture, Dead Letter Queues,
Cron Jobs, WebSockets, Long Polling, Server-Sent Events, Webhooks

## Security
Secrets Management, IAM, OAuth, JWT Rotation, TLS, Encryption at Rest,
Encryption in Transit, WAF, DDoS Protection, CORS, CSRF, SQL Injection, XSS,
SSRF

## Observability
Monitoring, Logging, Distributed Tracing, Metrics, Alerting, SLOs, SLIs,
Error Budgets, Observability, Latency, Throughput, P99 Latency, Tail Latency

## Deployment & delivery
CI/CD, Docker, Kubernetes, Feature Flags, Blue-Green Deployments,
Canary Releases, Rolling Deployments, Rollbacks, Infrastructure as Code,
Terraform, Helm Charts, Build Caching

## Reliability ops
Disaster Recovery, Backups, Failover, Production Incidents, On-call,
Postmortems

## Runtime / language
Memory Leaks, Garbage Collection, Thread Safety

## Networking
DNS, TCP vs UDP, HTTP/2 & HTTP/3, gRPC

## API design
API Versioning, Semantic Versioning

## Misc
Cost Optimization, Cold Starts, Serverless Limits, Dependency Hell
