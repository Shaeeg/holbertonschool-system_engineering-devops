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
