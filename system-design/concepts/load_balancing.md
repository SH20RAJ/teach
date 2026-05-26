# System Design: Load Balancing (Layer 4 vs Layer 7)
> **Format:** Concept-First Whiteboard Explanation
> **Topic:** Scaling Web Traffic & Request Routing

---

## 1. First Understand the Problem

Imagine you launch an online ticket booking app.
You deploy your web application on a single server machine: **Server A**.
Your server can handle a maximum of **1,000 requests per second (RPS)**.

Suddenly, tickets for a major concert go on sale!
**10,000 users click "Book Now" at the exact same second.**

What happens to Server A?
1. CPU spikes to 100%.
2. Network buffer overflows.
3. System runs out of memory (OOM) and crashes.
4. All users get a "504 Gateway Timeout" page.

### The Scaling Solution
You buy 4 more servers: **Server B, C, D, and E**.
Now you have 5 server machines. But users only know your domain: `www.tickets.com`.

When a user types your URL, how do you distribute their traffic evenly among your 5 servers so that no single server is overloaded?

You need a device that sits in front of your servers, intercepts incoming requests, and routes them to the least-busy server.
That device is a **Load Balancer**.

---

## 2. Imagine the Analogy Scene

Let's look at the two main ways a load balancer operates using real-life analogies.

### Scene A: Layer 4 Load Balancing (The Mail sorter)
Imagine a corporate mailroom clerk.
- A package arrives. The clerk does **not** open the package to read the letter inside.
- The clerk only looks at the envelope's **From Address** and **To Address** (IP Address and Port).
- The clerk quickly forwards it to Room 102.

```text
Incoming Packet (TCP Handshake)
       │
       ▼
 [ Layer 4 Balancer ] ──► Looks only at IP:Port ──► Forwards packet to [ Server B ]
```
**Key Point:** The L4 balancer does not look at what HTTP page, cookie, or payload is inside the request. It just routes raw TCP connection packets.

---

### Scene B: Layer 7 Load Balancing (The Smart Hotel Receptionist)
Imagine a hotel receptionist.
- A guest walks up and says: "I want to check in to a VIP Suite."
- The receptionist opens their request, reads their VIP ID card, and checks their booking profile.
- The receptionist routes the VIP guest to the VIP Wing (Server B) and a regular guest to the Standard Wing (Server A).

```text
Incoming HTTP Request (GET /api/checkout, Cookie: VIP=true)
       │
       ▼
 [ Layer 7 Balancer ] ──► Terminates TLS, reads HTTP headers ──► Routes to [ Checkout Server ]
```
**Key Point:** The L7 balancer fully terminates the connection, parses the HTTP content, reads cookies, headers, and URL paths, and routes requests intelligently based on the application layer data.

---

## 3. Layer 4 vs Layer 7: The Deep Technical Comparison

Let's write the system trade-offs on the whiteboard:

```text
                      +-------------------+
                      |   Client Request  |
                      +-------------------+
                                │
                                ▼
                       +-----------------+
                       |  Load Balancer  |
                       +-----------------+
                         /             \
            [ Layer 4 Routing ]    [ Layer 7 Routing ]
           - Direct Packet Forward  - Terminates Connection
           - No SSL decryption      - Decrypts SSL/TLS
           - Faster (low CPU)       - Slower (high CPU)
           - Simple TCP routing     - Smart URL/Cookie paths
```

### 1. Packet Handling & Speed
- **Layer 4 (Transport Layer):** Works with TCP/UDP packets. It does not inspect the message payload. It requires very low CPU because it just shifts packets. It cannot modify HTTP headers.
- **Layer 7 (Application Layer):** Works with HTTP/HTTPS, WebSockets, gRPC. It terminates the connection, decrypts the SSL/TLS certificate, inspects the request headers, cookies, and JSON body, and opens a new connection to the backend server. It is slower and requires higher CPU.

### 2. Smart Routing Capabilities
- **Layer 4:** Can only route based on client IP addresses. If one user sends a giant upload stream and another sends a small ping, it cannot balance the load based on complexity.
- **Layer 7:** Can route requests to different microservices based on the URL path:
  - `www.tickets.com/api/payment` ==> Routes to **Payment Servers**
  - `www.tickets.com/api/images`  ==> Routes to **Static Assets Storage**
  - Can read user session cookies to ensure a user is always pinned to the same server (**Sticky Sessions**).

---

## 4. Common Algorithms on the Whiteboard

How does the load balancer choose *which* server to send a request to?

1. **Round Robin:** Sends request 1 to Server A, request 2 to Server B, request 3 to Server C in order. (Problem: Assumes all servers have equal capacity and all requests require equal processing work).
2. **Least Connections:** Sends the new request to the server that is currently handling the fewest active connections. (Great for long-running database queries or websocket connections).
3. **IP Hash:** Hashes the client's IP address and uses it to select the server. This ensures a client always talks to the same server (session persistence).

---

## 5. Common Confusion

### Confusion: Can a Layer 4 load balancer handle SSL certificate termination?
*   **Correction:** No! SSL/TLS lives at the presentation/application layers. A Layer 4 load balancer does not read or decrypt application payload bytes. If you need your load balancer to terminate SSL (so your backend servers don't have to spend CPU decrypting HTTPS traffic), you **must** use a Layer 7 load balancer.

---

## 6. Interview-Safe Definitions

1. **Layer 4 Load Balancing:** Routing network traffic at the transport layer (TCP/UDP) using only IP address and port numbers without inspecting the application content.
2. **Layer 7 Load Balancing:** Routing network traffic at the application layer (HTTP/HTTPS) by inspecting URL paths, request headers, cookies, and payloads.
3. **SSL Termination:** The process where a load balancer acts as the endpoint for SSL/TLS encryption, decrypting incoming requests and forwarding them as plain HTTP to internal backend servers to reduce CPU overhead on those servers.

---

## 7. Active Recall Questions

1. **Why does Layer 7 load balancing require significantly more CPU resources than Layer 4 load balancing?**
2. **Under what scenario would you choose Round Robin over Least Connections as your load balancing algorithm?**
3. **How does an API gateway utilize Layer 7 routing to support a microservices architecture?**
