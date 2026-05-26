# Syllabus: High-Scale System Design Concepts
> **Instructor Course Plan**

Welcome to your course system design curriculum! Below is the complete catalog of core system design topics. Each topic has a dedicated, long, detailed, and concept-first whiteboard explanation markdown file.

---

## 📚 Core Lesson Plans

### 1. [HLS Video Streaming Concept](hls-video-streaming/teaching_doc.md)
*   **Concepts:** Video segment chunking, H.264/AAC, Master/Media Playlists (.m3u8), Adaptive Bitrate (ABR).
*   **Analogies:** The 1000-page book torn into brochures, multi-lane speed highway.

### 2. [Consistent Hashing](concepts/consistent_hashing.md)
*   **Concepts:** Modulo routing failures, hash rings, virtual nodes (Vnodes), scale outage cascades.
*   **Analogies:** The Clock Hash Ring, clockwise walking.

### 3. [Rate Limiting (Token vs Leaky Bucket)](concepts/rate_limiting.md)
*   **Concepts:** API abuse defense, Token Bucket vs Leaky Bucket, traffic queue smoothing, burst allowances.
*   **Analogies:** Arcade ticket machine coin bucket vs water funnel.

### 4. [Database Sharding](concepts/database_sharding.md)
*   **Concepts:** Horizontal partitioning, sharding keys, scatter-gather queries, horizontal vs vertical scaling.
*   **Analogies:** Filing cabinet alphabet labels (A-I, J-R, S-Z).

### 5. [Message Queues (RabbitMQ vs Kafka)](concepts/message_queues.md)
*   **Concepts:** Asynchronous queues, brokers, sequential log tracks, offsets, consumer groups, AMQP vs Log streaming.
*   **Analogies:** Post office mailbox inbox vs physical ledger tape recorder.

### 6. [Load Balancing (Layer 4 vs Layer 7)](concepts/load_balancing.md)
*   **Concepts:** Round Robin, Least Connections, IP Hash, TCP packet routing (L4) vs Application HTTP header/cookie routing (L7), SSL termination.
*   **Analogies:** Corporate mailroom clerk vs smart hotel receptionist suite router.

### 7. [CDN Caching Mechanics](concepts/cdn_caching.md)
*   **Concepts:** Edge Servers, Point of Presence (PoP), Cache Pull vs Cache Push, TTL, Cache Invalidation, Cache-Busting.
*   **Analogies:** Regional distribution warehouses vs central factory shipping.

### 8. [Database Replication](concepts/database_replication.md)
*   **Concepts:** Single-Leader, Multi-Leader, Leaderless replication, write quorums ($W$) and read quorums ($R$), replication lag, split-brain failure.
*   **Analogies:** Manager and assistants ledger coping, committee voting.

### 9. [Distributed Transactions](concepts/distributed_transactions.md)
*   **Concepts:** Two-Phase Commit (2PC) Prepare/Commit phases, blocking database locks, Saga Pattern (Orchestration vs Choreography), compensating transactions.
*   **Analogies:** Priest wedding ceremony votes vs custom dress refund sequence.

### 10. [SQL vs NoSQL](concepts/sql_vs_nosql.md)
*   **Concepts:** Strict schema table Joins vs Denormalized self-contained document folders, ACID transactions vs BASE eventual consistency, scaling ceilings.
*   **Analogies:** Strict library index card cabinets vs self-contained manila folder boxes.

### 11. [Distributed Locking](concepts/distributed_locking.md)
*   **Concepts:** Concurrency race conditions, Redis SET NX PX locks, lock lease expiration safety, garbage collection (GC) freeze failures, fencing tokens.
*   **Analogies:** Shared meeting room whiteboard markers, sequential incrementing authorization stamps.

### 12. [Heartbeats & Gossip Protocols](concepts/heartbeats_gossip.md)
*   **Concepts:** Cluster membership list tracking, network load scaling bottlenecks, peer-to-peer gossip rumor loops, Merkle Trees, Anti-Entropy sync.
*   **Analogies:** Office whispers rumor spreading.

### 13. [DNS Resolution & Caching](concepts/dns_resolution.md)
*   **Concepts:** Recursive Resolvers, Root Nameservers, Top-Level Domain (TLD) servers, Authoritative Name Servers, DNS caching layers, A/CNAME records.
*   **Analogies:** Detective traversing hierarchical building directories.

### 14. [API Gateway vs Reverse Proxy](concepts/api_gateway.md)
*   **Concepts:** Authentication middleware, reverse proxy network forwarding vs API gateway programmable controllers, protocol translation (HTTP to gRPC).
*   **Analogies:** Office lobby reception security guard verification desk.

### 15. [Distributed ID Generation (Snowflake ID)](concepts/distributed_id_generation.md)
*   **Concepts:** Unique IDs, database auto-increment limits, random UUID drawbacks, Twitter Snowflake bit layout (Timestamp, Worker ID, Sequence), clock drift.
*   **Analogies:** Nationwide cargo truck shipping parcel labeling.

### 16. [Vector Databases](concepts/vector_databases.md)
*   **Concepts:** Semantic search, embeddings, high-dimensional vector grids, Hierarchical Navigable Small World (HNSW) graph skip-lists, Cosine similarity.
*   **Analogies:** Coordinate mapping on a 2D sheet of paper scaled to 1536 dimensions.
