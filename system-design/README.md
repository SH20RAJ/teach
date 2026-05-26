# System Design Whiteboard Course Material
> **High-Scale Engineering Concepts Taught Whiteboard-Style**

This repository contains the complete syllabus, lecture plans, and interactive visual aids for your system design course. It is designed to teach high-scale infrastructure concepts using a **concept-first, visual-first whiteboard style**.

---

## 📂 Repository Structure

- **[teaching_methodology.md](teaching_methodology.md)**: The pedagogical guide on how to deliver these lectures visually and build long-term student memory.
- **[concepts_list.md](concepts_list.md)**: Main syllabus index mapping active lesson plans.
- **`concepts/`**: Detailed markdown-only whiteboard lesson plans for core system design topics.

---

## 🚀 How to Launch the Interactive Board

1. Open your terminal and start the server:
   ```bash
   cd concepts/hls-video-streaming
   node server.js
   ```
2. Open your web browser:
   👉 **[http://localhost:3000](http://localhost:3000)**

You can now use the slides, edit markdown in real-time, draw using the whiteboard layer, and click **"Save to File"** to make your annotations permanent.

---

## 📚 Core Lesson Plans (Markdown)

1. **[HLS Video Streaming](concepts/hls-video-streaming/teaching_doc.md)**: Chunking, playlists, and adaptive bitrates.
2. **[Consistent Hashing](concepts/consistent_hashing.md)**: Hash rings, scale outages, and virtual nodes.
3. **[Rate Limiting](concepts/rate_limiting.md)**: Token Bucket vs Leaky Bucket algorithms.
4. **[Database Sharding](concepts/database_sharding.md)**: Horizontal database partition keys and query routing.
5. **[Message Queues](concepts/message_queues.md)**: AMQP messaging vs Log-based event streaming (RabbitMQ vs Kafka).
6. **[Load Balancing (L4 vs L7)](concepts/load_balancing.md)**: Round robin, least connections, packet vs header routing, and SSL termination.
7. **[CDN Caching Mechanics](concepts/cdn_caching.md)**: Edge servers, PoP nodes, Cache Pull vs Push, cache invalidations and TTLs.
8. **[Database Replication](concepts/database_replication.md)**: Leader-follower, multi-leader, leaderless quorum, write/read coordination, split-brain scenario.
9. **[Distributed Transactions](concepts/distributed_transactions.md)**: Two-Phase Commit (2PC), Sagas (Orchestrator vs Choreography), compensating write actions.
10. **[SQL vs NoSQL](concepts/sql_vs_nosql.md)**: Relational schema tables vs Denormalized documents, ACID vs BASE transaction bounds.
11. **[Distributed Locking](concepts/distributed_locking.md)**: Mutex lock parameters, Redis SET NX PX commands, Garbage Collection freeze errors, Fencing tokens.
12. **[Heartbeats & Gossip Protocols](concepts/heartbeats_gossip.md)**: Cluster state coordination, UDP rumor loops, failure detection, Merkle Trees.
13. **[DNS Resolution & Caching](concepts/dns_resolution.md)**: Recursive resolvers, Root nameservers, TLD servers, Authoritative Name Servers, TTL propagation.
14. **[API Gateway vs Reverse Proxy](concepts/api_gateway.md)**: Nginx network routing vs programmable gateway middleware, HTTP to gRPC translation.
15. **[Distributed ID Generation (Snowflake ID)](concepts/distributed_id_generation.md)**: Auto-increment bottlenecks, UUID index page split failures, Twitter Snowflake bit segments, clock drift.
16. **[Vector Databases](concepts/vector_databases.md)**: Semantic AI search, embeddings, high-dimensional coordinate distance, HNSW graphs.
