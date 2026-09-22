# Backend / distributed-systems checklist (111 items)

The "things vibe coders never hear about" list that circulates as a meme. Kept
here verbatim, only re-ordered into the groups from step 3 of the workflow so
one pass of reads can answer a whole group at once.

This file is a source of **items, not verdicts**. Nothing here is a
recommendation for any project — most of these are actively wrong for a small
single-server app. Every item still goes through step 4 investigation before it
lands in a bucket.

At 111 items this is over the ~40 threshold, so the intended run is step 3's
parallel-subagent branch: one `Explore` agent per group below, each handed the
project-context summary and its own item numbers.

## Traffic & edge (1–11)

1. Rate Limiting
2. Caching
3. Load Balancing
4. Reverse Proxies
5. API Gateways
6. CDN
7. Edge Caching
8. Cache Invalidation
9. WAF
10. DDoS Protection
11. DNS

## Delivery, deploy & versioning (12–28)

12. CI/CD
13. Docker
14. Kubernetes
15. Helm Charts
16. Infrastructure as Code
17. Terraform
18. Feature Flags
19. Blue-Green Deployments
20. Canary Releases
21. Rolling Deployments
22. Rollbacks
23. Build Caching
24. Dependency Hell
25. Database Migrations
26. Schema Versioning
27. API Versioning
28. Semantic Versioning

## Resilience & distributed behavior (29–46)

29. Service Discovery
30. Circuit Breakers
31. Timeouts
32. Retries
33. Exponential Backoff
34. Idempotency
35. Leader Election
36. CAP Theorem
37. Eventual Consistency
38. Distributed Locks
39. Network Partitions
40. Clock Skew
41. Backpressure
42. Failover
43. Multi-Region Deployments
44. Chaos Engineering
45. Disaster Recovery
46. Backups

## Async & messaging (47–54)

47. Message Queues
48. Pub/Sub
49. Event-Driven Architecture
50. Distributed Transactions
51. Saga Pattern
52. Dead Letter Queues
53. Cron Jobs
54. Webhooks

## Realtime & protocols (55–60)

55. WebSockets
56. Long Polling
57. Server-Sent Events
58. TCP vs UDP
59. HTTP/2 & HTTP/3
60. gRPC

## Data layer (61–70)

61. Database Indexing
62. Query Optimization
63. N+1 Queries
64. Connection Pooling
65. Read Replicas
66. Sharding
67. Partitioning
68. Replication
69. Optimistic Locking
70. Pessimistic Locking

## Concurrency & runtime (71–77)

71. Race Conditions
72. Deadlocks
73. Memory Leaks
74. Garbage Collection
75. Thread Safety
76. Cold Starts
77. Serverless Limits

## Scaling & performance (78–85)

78. Autoscaling
79. Horizontal Scaling
80. Vertical Scaling
81. Latency
82. Throughput
83. P99 Latency
84. Tail Latency
85. Cost Optimization

## Observability (86–96)

86. Health Checks
87. Liveness & Readiness Probes
88. Monitoring
89. Logging
90. Distributed Tracing
91. Metrics
92. Alerting
93. SLOs
94. SLIs
95. Error Budgets
96. Observability

## Security (97–108)

97. Secrets Management
98. IAM
99. OAuth
100. JWT Rotation
101. TLS
102. Encryption at Rest
103. Encryption in Transit
104. CORS
105. CSRF
106. SQL Injection
107. XSS
108. SSRF

## Ops & human process (109–111)

109. Production Incidents
110. On-call
111. Postmortems

## Notes for auditing this particular list

Many items here are *concepts*, not features — "CAP Theorem", "Eventual
Consistency", "Latency", "Throughput", "Dependency Hell". These have no
enforcement point to grep for. Convert each into a concrete question about the
project before investigating, or the audit degenerates into a vocabulary quiz:

- "CAP Theorem" → is there more than one node holding state at all? If not, the
  item is not applicable for a nameable architectural reason.
- "Eventual Consistency" → is any read served from a replica or cache that can
  lag the write path, and does the code assume it can't?
- "Latency" / "Throughput" / "P99 Latency" → is anything measured, or is the
  answer simply "no numbers exist"? "No measurement" is a real finding.
- "Dependency Hell" → are versions pinned, is there a lockfile, does CI ever
  resolve dependencies fresh?
- "Race Conditions" / "Thread Safety" → find the actual shared mutable state
  (a counter, a cache, a job claim) and check it, rather than answering about
  the language's concurrency model in the abstract.

The single-server/small-project reality is that a large fraction of this list is
genuinely not applicable, and saying so with the architectural reason is the
correct output — not a padded "already have" bucket. Resist the pull to make a
111-item list produce 111 action items.
