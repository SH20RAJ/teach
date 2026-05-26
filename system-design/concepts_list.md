# Syllabus: High-Scale System Design Concepts
> **Instructor Course Plan**

Welcome to your course system design curriculum! Below is the curated list of core system design topics. Each topic has a dedicated, concept-first whiteboard explanation markdown file.

## 📚 Active Lesson Plans

### 1. [HLS Video Streaming Concept](file:///Users/shaswatraj/Desktop/youtubr/teach/system-design/hls-video-streaming/teaching_doc.md)
*   **The Problem:** Why sending raw `movie.mp4` files over a network causes high startup buffer latency, data wastage, and network freezes.
*   **Analogies:** The 1000-page book torn into brochures, multi-lane speed highway.
*   **Key Terms:** Video chunking, H.264/AAC, Master/Media Playlists (.m3u8), Adaptive Bitrate (ABR).

### 2. [Consistent Hashing](file:///Users/shaswatraj/Desktop/youtubr/teach/system-design/concepts/consistent_hashing.md)
*   **The Problem:** How traditional modulo hashing (`hash % N`) triggers cache stamps and database crashes when a cluster scales or encounters nodes down.
*   **Analogies:** The Clock Hash Ring, clockwise walking.
*   **Key Terms:** Hash Ring, Virtual Nodes (Vnodes), Cache Stampede.

### 3. [Rate Limiting (Token vs Leaky Bucket)](file:///Users/shaswatraj/Desktop/youtubr/teach/system-design/concepts/rate_limiting.md)
*   **The Problem:** Defending APIs against brute-force script abuse, resource exhaustion, and high vendor cost-leaks.
*   **Analogies:** Arcade ticket machine coin bucket vs water funnel.
*   **Key Terms:** Token Bucket, Leaky Bucket, Queue smoothing, bursty traffic allowance, HTTP 429.

### 4. [Database Sharding](file:///Users/shaswatraj/Desktop/youtubr/teach/system-design/concepts/database_sharding.md)
*   **The Problem:** Overcoming the physical disk and CPU limits of vertical scaling when a table grows to 50M+ rows.
*   **Analogies:** Filing cabinet alphabet labels (A-I, J-R, S-Z).
*   **Key Terms:** Horizontal Partitioning, Sharding Key, Scatter-Gather queries, Database Replication vs Sharding.

### 5. [Message Queues (RabbitMQ vs Kafka)](file:///Users/shaswatraj/Desktop/youtubr/teach/system-design/concepts/message_queues.md)
*   **The Problem:** Resolving high latency and single point of failure in microservices synchronous HTTP calling.
*   **Analogies:** Post office inbox letter box vs physical ledger tape recorder logs.
*   **Key Terms:** Asynchronous queues, Message Brokers, Event Log Streams, Offsets, Consumer Groups, Dead Letter Queues (DLQ).

---

## 🚀 Recommended Future Additions

Here are next-level concepts to add to your course material:
- **Load Balancing (Layer 4 vs Layer 7 Routing):** Explaining routing packets vs headers using mail routing analogies.
- **CDN Caching & Purging Policies:** Edge server caching, GeoDNS routing, TTL and cache invalidation methods.
- **CAP Theorem (Consistency vs Availability vs Partition Tolerance):** Deciding between immediate correctness or fast reads when a network link snaps.
- **Heartbeat & Gossip Protocols:** How servers in a large cluster know who is alive and who crashed.
