# Web Infrastructure Design

This project covers the design of web infrastructures, from a simple single-server LAMP stack to secured, monitored, and scaled distributed systems.

## Tasks

### 0. Simple web stack
A single-server LAMP stack hosting `www.foobar.com`.

**Components:**
- 1 server (IP: 8.8.8.8)
- Nginx (web server)
- Application server
- Application codebase
- MySQL database
- Domain: foobar.com with www A record → 8.8.8.8

**How it works:**
1. User types www.foobar.com in browser
2. DNS resolves www A record to 8.8.8.8
3. Browser sends HTTP request via TCP/IP to the server
4. Nginx receives the request and forwards it to the application server
5. Application server executes the code and queries MySQL if needed
6. Response travels back to the user's browser

**Issues with this infrastructure:**
- **SPOF**: Single point of failure — if the server goes down, the whole site is down
- **Maintenance downtime**: Deploying new code requires restarting Nginx, causing downtime
- **No scalability**: Cannot handle traffic spikes — one server has a fixed capacity

### 1. Distributed web infrastructure
A three-server infrastructure with HAProxy load balancer hosting `www.foobar.com`.

**Added components and why:**
- HAProxy (load balancer): distributes incoming traffic across the two servers to avoid overloading a single machine and improve availability
- Server 2 (second server): provides redundancy — if one server fails, the other keeps the site alive
- MySQL Primary-Replica cluster: separates read and write operations for performance and provides a data backup

**Load balancer algorithm:**
- Round-robin: each new request goes to the next server in turn (Server 1 → Server 2 → Server 1 → ...), distributing load evenly

**Active-Active vs Active-Passive:**
- Active-Active (our setup): both servers handle traffic simultaneously — better performance and load distribution
- Active-Passive: one server handles all traffic, the other sits idle as a hot standby — only takes over if the active server fails

**Primary-Replica (Master-Slave) database cluster:**
- The Primary node handles all WRITE operations (INSERT, UPDATE, DELETE)
- The Replica node continuously copies data from the Primary (replication) and handles READ operations
- The application writes to Primary and can read from either node

**Primary vs Replica for the application:**
- Primary: the source of truth — all writes go here
- Replica: a read-only copy — reduces load on Primary by serving read queries

**Issues with this infrastructure:**
- SPOF: HAProxy is a single point of failure — if it goes down, the entire site is unreachable
- No firewall: servers are exposed directly to the internet with no traffic filtering
- No HTTPS: traffic between users and the infrastructure is unencrypted
- No monitoring: no alerting or visibility into server health, traffic, or errors
