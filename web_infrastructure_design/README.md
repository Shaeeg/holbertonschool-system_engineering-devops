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

### 2. Secured and monitored web infrastructure
A three-server infrastructure secured with firewalls, HTTPS, and monitoring for `www.foobar.com`.

**Added components and why:**
- 3 firewalls: filter incoming and outgoing traffic to block unauthorized access and attacks
- SSL certificate: encrypts traffic between the user and the site over HTTPS, protecting data in transit
- 3 monitoring clients: collect metrics, logs, and performance data from each component and send to Sumo Logic

**What firewalls do:**
- Act as a barrier between the internet and the servers, allowing only legitimate traffic (e.g. port 443 for HTTPS) and blocking everything else

**Why HTTPS:**
- Encrypts all traffic between the user's browser and the server so that passwords, personal data, and session tokens cannot be intercepted by a third party

**What monitoring is used for:**
- Tracks server health, uptime, error rates, response times, and traffic patterns
- Sends alerts when something goes wrong (high CPU, downtime, errors)

**How the monitoring tool collects data:**
- A lightweight agent (monitoring client) runs on each server
- It collects logs, system metrics (CPU, RAM, disk), and application metrics
- It ships the data to Sumo Logic in real time for aggregation and visualization

**How to monitor QPS (Queries Per Second) on Nginx:**
- Enable the Nginx stub_status module to expose request metrics
- Configure the monitoring agent to read and forward those metrics to Sumo Logic
- Create a dashboard in Sumo Logic to visualize QPS over time

**Issues with this infrastructure:**
- SSL terminates at the load balancer: traffic between HAProxy and the backend servers is plain HTTP — unencrypted inside the network, which is a risk if the internal network is compromised
- Single MySQL writer: only the Primary can accept writes — if it goes down, no writes are possible, causing partial downtime
- Mixed-role servers: each server runs the database, app server, and web server together — a spike in DB load affects web performance, and scaling one component means scaling all of them unnecessarily
