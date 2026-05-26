# System Design: DNS Resolution & Caching Layers
> **Format:** Concept-First Whiteboard Explanation
> **Topic:** Internet Name Resolution & Routing Foundations

---

## 1. First Understand the Problem

Computers on the internet do not speak human languages.
Every server machine has a numerical coordinate called an **IP Address** (e.g., `142.250.190.46`).

When a user wants to visit your website, they type:
`www.tickets.com`

Your computer has no idea where `www.tickets.com` is.
It cannot send packets to a word. It must send packets to an IP.

How does the browser translate the word `www.tickets.com` into the IP `142.250.190.46`?

### Method 1: The Single Global Phonebook Server
You host one giant central server that holds all domain-to-IP mappings.
Every browser in the world queries this server before opening any web page.

**Why does this fail?**
1. **Unscalable Traffic:** Billions of devices would hit this single IP address every millisecond. The server would melt.
2. **Infinite Latency:** If you are in Tokyo, and the phonebook server is in Washington D.C., you face 300ms of lag just resolving the name of a site before you even start loading it!

We need a highly distributed, hierarchical, cached resolution system.
That system is the **Domain Name System (DNS)**.

---

## 2. Imagine the Detective Analogy

Imagine you are looking for a doctor named **Dr. Robert Smith**.
You don't call a global phone directory. Instead, you hire a detective (Recursive Resolver) to find his room number.

The detective goes step-by-step up a hierarchy of directories:

```text
       [ Recursive Resolver ] (Detective)
                │
                ├── Step 1: "Where is '.com' cabinet?" ──► [ Root Server ]
                │
                ├── Step 2: "Where is 'tickets.com'?"  ──► [ TLD Server (.com) ]
                │
                └── Step 3: "What is the IP address?"  ──► [ Authoritative Server ]
```

### The Search Process:
1. **The Root Directory (Root Server):** The detective asks: *"Where is Dr. Smith?"* The Root Directory says: *"I don't know Dr. Smith, but I know the registrar for the 'Smith' family (TLD Server). Go to Room 5."*
2. **The family Directory (TLD Server):** The detective goes to Room 5 and asks: *"Where is Robert Smith?"* The TLD Server says: *"I don't know his address, but I know his personal assistant's office (Authoritative Name Server) in Chicago. Go there."*
3. **The Personal Assistant (Authoritative Nameserver):** The detective calls Chicago. The assistant reads their ledger book and says: *"Dr. Robert Smith is in Room 1405 (The final IP address)!"*
4. **The Caching:** The detective returns to you, gives you the address, and writes it down in his pocket notebook (caching) so that if you ask for Dr. Smith again tomorrow, he does not have to make the trips to the directories!

---

## 3. How DNS Resolves a Name: Step-by-Step

Let's look at the actual network hop flow when you query `www.tickets.com`.

```text
Browser ──► [ Recursive Resolver (ISP / 8.8.8.8) ]
                    │
                    ├── Step 1: "Where is .com?" ──► [ Root Nameserver ] (e.g. 198.41.0.4)
                    │   ◄── Returns IP of TLD ──
                    │
                    ├── Step 2: "Where is tickets.com?" ──► [ TLD Nameserver ] (e.g. .com registry)
                    │   ◄── Returns IP of tickets.com NS ──
                    │
                    └── Step 3: "What is www.tickets.com?" ──► [ Authoritative NS ] (e.g. Route53)
                        ◄── Returns 142.250.190.46 (IP) ──
```

### The Roles of Each Component:
1. **Recursive Resolver (The ISP / Public DNS like Cloudflare 1.1.1.1):** The middleman that does the hard work of traversing the internet to resolve the name for the browser.
2. **Root Nameserver:** Represents the root zone (`.`). There are 13 logical root server addresses globally. They direct resolvers to TLD servers based on the domain suffix (`.com`, `.org`, `.net`).
3. **Top-Level Domain (TLD) Nameserver:** Manages domain suffixes. The `.com` TLD server points to the Authoritative Name Server of `tickets.com`.
4. **Authoritative Nameserver:** The actual nameserver configured by the domain owner (e.g., AWS Route53, GoDaddy). It has the definitive mapping database of domain records (A, AAAA, CNAME, MX).

---

## 4. DNS Caching & TTL (Time-To-Live)

Because doing 3 internet roundtrips for every single asset load would slow down the web, DNS relies heavily on **caching at every layer**:
- **Browser Cache:** Browser keeps records for a few minutes.
- **OS Cache:** Operating System checks its local hosts file and DNS cache.
- **Resolver Cache:** The ISP resolver caches queries based on the **TTL** (Time To Live) defined in the DNS record.

### TTL Example:
If you set the TTL of your A record to `300 seconds`:
- Once resolved, the resolver caches the IP for 5 minutes.
- If you change your server IP, it will take up to 5 minutes for the change to propagate globally.
- If you set TTL to `86400` (1 day), it takes up to 24 hours to update globally, but you save on DNS query charges!

---

## 5. Common Confusion

### Confusion: Does DNS handle the actual HTTP website traffic?
*   **Correction:** No! DNS is purely a **directory service** operating over UDP Port 53. Once the browser obtains the IP address (e.g. `142.250.190.46`), the DNS's job is complete. The browser then opens a separate TCP connection to that IP on Port 80/443 to load the website content.

---

## 6. Interview-Safe Definitions

1. **A Record (Address Record):** A DNS record type that maps a domain name directly to an IPv4 address.
2. **CNAME Record (Canonical Name):** A DNS record type that maps an alias domain name to another domain name (redirects resolution to another target).
3. **Recursive Resolution:** The query flow where a resolver recursively queries multiple nameservers down the tree to locate the IP address on behalf of the client.

---

## 7. Active Recall Questions

1. **Explain the difference between an Iterative DNS query and a Recursive DNS query.**
2. **Why does DNS query resolution primarily use UDP instead of TCP? Under what scenario does it switch to TCP?**
3. **What is "DNS Hijacking" and how does DNSSEC (DNS Security Extensions) prevent it?**
