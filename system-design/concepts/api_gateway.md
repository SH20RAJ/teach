# System Design: API Gateway vs Reverse Proxy
> **Format:** Concept-First Whiteboard Explanation
> **Topic:** Microservices Orchestration & Traffic Control

---

## 1. First Understand the Problem

Imagine you build a distributed ride-sharing app. You have split your backend into multiple **microservices**:
1. **Auth Service:** Verifies user logins.
2. **Driver Service:** Tracks driver coordinates.
3. **Trip Service:** Manages ride requests.
4. **Billing Service:** Charges payment methods.

Now, you write the mobile app. The mobile client must contact all these services.
If you connect directly:

```text
               Mobile Client
              /      |      \
             ▼       ▼       ▼
         [ Auth ] [ Driver ] [ Trip ]
```

**Now, what are the architectural nightmares?**
1. **Multiple Endpoints:** The mobile developer must manage 10 different base URLs (`auth.taxi.com`, `driver.taxi.com`, `trip.taxi.com`).
2. **Security Leak:** Every microservice must implement its own JWT authentication check, rate limiting defense, and SSL certificate decryption. If you find a security bug in your auth logic, you have to patch and redeploy all 10 microservices!
3. **Protocol Mismatch:** Some internal microservices communicate using high-speed gRPC, but mobile devices can only easily make HTTP/REST requests.

We need a single, unified entry point that intercepts all client requests, authenticates them once, translates protocols, and routes them to the correct internal microservice.
That entry point is the **API Gateway**.

---

## 2. Imagine the Security Guard Analogy

Imagine a high-security corporate building containing multiple separate departments (Finance, HR, Engineering).
You do not let visitors wander around the hallways, knock on different department doors, and show their ID cards at every desk.

What does the building do?
It builds a **Front Reception Lobby (API Gateway)**!

```text
Visitor (HTTP Request) ──► [ Lobby Guard ] ─── (Verify badge & direct) ───► [ Finance Room ]
```

### The Lobby Rules:
1. **One Entrance:** All visitors enter through the lobby door (Single Base URL: `api.taxi.com`).
2. **Security Check:** The lobby guard checks your visitor badge (JWT token). If you don't have a valid badge, you are thrown out immediately (Authentication fail).
3. **Direction Routing:** You tell the guard: *"I want to submit an expense report."* The guard directs you to the Finance Department (Service Routing: `/billing` -> Billing Service).
4. **Crowd Control:** If too many people try to enter at once, the guard blocks the line (Rate Limiting).

---

## 3. API Gateway vs Reverse Proxy: The Difference

On the whiteboard, they look similar. Both sit between clients and servers. What is the difference?

### 1. Reverse Proxy (The Mail Router)
A reverse proxy (like Nginx, HAProxy) is a **generic infrastructure network tool**.
- Its primary job is load balancing, SSL termination, and routing static paths.
- It has no business logic. It does not inspect database states, decode JWT tokens using custom databases, or transform request JSONs.

### 2. API Gateway (The Smart Application Controller)
An API Gateway (like Kong, Apigee, AWS API Gateway) is an **application-aware orchestration layer**.
- It is programmable.
- It can make database calls, check custom Auth servers, strip headers, rewrite query params, run custom Lua scripts, track API monetization quotas, and handle protocol translation (e.g. converting incoming HTTP to internal gRPC).

---

## 4. API Gateway Architecture Diagram

```text
                   External Clients (HTTPS)
                              │
                              ▼
                    +───────────────────+
                    |    API Gateway    |  <-- SSL termination, JWT verify, Rate limit
                    +───────────────────+
                      /        |        \
            (gRPC)   /         | (HTTP)  \   (gRPC)
                    ▼          ▼          ▼
                [ Auth ]    [ Trip ]  [ Billing ]
```

---

## 5. Common Confusion

### Confusion: Does an API Gateway make microservices slower by adding an extra hop?
*   **Correction:** Technically, yes, it adds a few milliseconds of network latency because traffic goes through an extra server hops. However, it **improves overall performance** because:
    1. It handles CPU-heavy SSL decryption at the gateway, leaving internal services clean.
    2. It filters out spam/unauthorized requests early, protecting backend nodes from wasted computation.
    3. It aggregates responses (e.g. fetching user info and their trip list in a single HTTP request to the client, but fetching them internally across multiple microservices simultaneously).

---

## 6. Interview-Safe Definitions

1. **API Gateway:** A server that acts as an API front-end, routing requests, enforcing security policies, terminating SSL, and translating protocols across microservices.
2. **Reverse Proxy:** An application-agnostic server that intercepts requests from external clients and forwards them to one or more internal web servers.
3. **Protocol Translation:** The process where an API Gateway translates incoming network protocols (like REST/HTTP) into internal backend service protocols (like gRPC, AMQP, or TCP).

---

## 7. Active Recall Questions

1. **Explain why SSL termination at the API Gateway layer improves backend service performance.**
2. **Under what scenario would you choose a simple Nginx reverse proxy over a dedicated API Gateway like Kong?**
3. **What is "BFF" (Backend-For-Frontend) pattern, and how does it relate to API Gateway design?**
