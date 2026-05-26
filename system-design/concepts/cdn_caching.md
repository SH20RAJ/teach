# System Design: CDN (Content Delivery Networks) & Caching Mechanics
> **Format:** Concept-First Whiteboard Explanation
> **Topic:** Global Asset Distribution & Latency Reduction

---

## 1. First Understand the Problem

Imagine you build a portfolio website. Your web server (Origin) is physically located in **Mumbai, India**.
- A user in **Mumbai** visits your site. The packets travel 10 kilometers.
  - **Latency:** 15 milliseconds. The page loads instantly!
- A user in **New York, USA** visits your site. The packets must travel through underwater optical fiber cables across oceans.
  - **Latency:** 350 milliseconds. The page feels sluggish, loading images piece by piece.

Now, imagine your site has giant media assets (10 MB videos or high-resolution banner images).
If 100,000 users from New York access your Mumbai server simultaneously:
1. Your Mumbai server's upload bandwidth is completely saturated.
2. The network transit lines get bottlenecked.
3. The page load times shoot up to 10 seconds for everyone.

### The Scaling Solution
We cannot change the speed of light. Packets will always take time to cross the globe.
So, instead of bringing the user's request all the way to Mumbai, why don't we **bring the files closer to the user**?

If we put copy-servers in New York, London, Tokyo, and Sydney, the New York user can fetch the files directly from the New York copy-server!
This global network of copy-servers is called a **Content Delivery Network (CDN)**.

---

## 2. Imagine the Warehouse Analogy

Imagine a massive manufacturing brand. Its factory is in India.
If a customer in New York orders a single bottle of shampoo, shipping it from the India factory takes 10 days and costs a lot of money.

What does the brand do?
It builds local distribution warehouses (CDNs) in New York, London, and Tokyo!
- When a customer in New York orders shampoo, they don't call the India factory.
- They order from the New York warehouse.
- The warehouse delivers it in 2 hours.

```text
[ India Factory ] ─── (Ship once in bulk) ───► [ NY Warehouse ] ─── (2 hours) ───► [ NY Customer ]
```

### Mapping to CDN System Design:
- **India Factory** = **Origin Server** (the source of truth database/assets server).
- **Shampoo Bottle** = **Static files** (HTML, CSS, JS, JPEG, MP4).
- **NY Warehouse** = **CDN Edge Server / Point of Presence (PoP)**.
- **NY Customer** = **Client Browser**.

---

## 3. CDN Mechanics: Cache Pull vs Cache Push

How do the files get inside the Edge Servers? There are two models.

### Model A: Cache Pull (On-Demand Loading)
```text
Client in NY requests logo.png from NY Edge Server.

"Do you have logo.png?"
NY Edge checks storage: Cache Miss!

Now, what does the Edge Server do?
It contacts the India Origin Server: "Send me logo.png".
India Origin sends logo.png to NY Edge.
NY Edge stores logo.png in its cache.
NY Edge delivers logo.png to the client.

Next client requests logo.png.
NY Edge checks storage: Cache Hit!
Delivered instantly. No trip to India.
```
- **Pro:** Saves storage space on the CDN. Only files that are active are cached.
- **Con:** The very first user to request a file faces high latency (the cache miss penalty).

---

### Model B: Cache Push (Pre-Loading)
- The developer uploads new files directly to the CDN Edge servers simultaneously during deployment.
- **Pro:** 100% Cache Hits for all users immediately.
- **Con:** Wastes storage space. Files that are never visited are still pushed globally, costing money.

---

## 4. Cache Invalidation: The Out-of-Date Problem

What happens if you update `logo.png` on your Origin server (e.g. you changed your brand logo)?
The Edge servers still have the old `logo.png` cached.
The NY customer still sees the old logo!

How do we fix this? **Cache Invalidation**.

1. **TTL (Time to Live):** You set an expiration time (e.g., `Cache-Control: max-age=86400` which means 1 day). After 1 day, the Edge server automatically marks the file as stale and pulls the new version on the next request.
2. **Purging:** You send an API call to the CDN provider to force-clear the cache of a specific path (e.g., `Purge /images/logo.png`). This forces all Edge servers to delete the file immediately.
3. **Cache- Busting (Best Practice):** You rename the file using a hash of its contents: `logo.v1.png` becomes `logo.v2.png` or `logo.8fa2c3d.png`. Because the URL changes, the CDN treats it as a brand new file, fetching it instantly on the next request without needing purging!

---

## 5. Common Confusion

### Confusion: Can we cache dynamic pages like `/api/user-profile`?
*   **Correction:** Generally, no. CDNs are optimized for **static content** (files that are identical for all users). User profiles contain highly specific database data that changes per second. Caching them on a public CDN would leak private data to other users. (However, modern Edge networks support *Dynamic Content Acceleration*, which optimizes routing paths to speed up database connection handshakes, even if the content itself isn't cached).

---

## 6. Interview-Safe Definitions

1. **Point of Presence (PoP):** A physical location or data center where CDN edge servers are placed to interface with local internet service providers (ISPs).
2. **Origin Server:** The source-of-truth web server where the primary version of a website's files and database reside.
3. **Cache Invalidation:** The process of removing stale cached files from CDN edge servers before their default expiration date (TTL) has passed.

---

## 7. Active Recall Questions

1. **Explain the difference between Cache-Busting and CDN Cache Purging.**
2. **Why does a CDN save considerable server cost and bandwidth on the Origin server?**
3. **What is a "stampede" or "thundering herd" problem on an Origin server when a popular file's CDN cache expires?**
