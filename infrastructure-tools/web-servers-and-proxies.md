# Web Servers & Proxies: Apache, Nginx, HAProxy, Caddy, Traefik

> Choosing the right web server or proxy is foundational to system performance, scalability, and operational simplicity. This article covers the major players — their architectures, trade-offs, and when to reach for each.

---

## Table of Contents

1. [Apache HTTP Server](#apache-http-server)
2. [Nginx](#nginx)
3. [Apache vs Nginx: Head-to-Head](#apache-vs-nginx-head-to-head)
4. [Reverse Proxy Patterns](#reverse-proxy-patterns)
5. [HAProxy](#haproxy)
6. [Caddy](#caddy)
7. [Traefik](#traefik)
8. [Comparison Matrix](#comparison-matrix)
9. [When to Use What](#when-to-use-what)

---

## Apache HTTP Server

### History & Philosophy

Apache HTTP Server (httpd) has been the world's most popular web server for most of the internet's history. Released in 1995, it was built on Unix's process-per-connection model and emphasizes **configuration flexibility and extensibility** via a rich module ecosystem.

### Architecture: Process/Thread-Per-Request

Apache's concurrency is managed by **Multi-Processing Modules (MPMs)**:

#### Prefork MPM (Traditional)
```
┌───────────────────────────────────────────────────┐
│                Apache Parent Process               │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐   │
│  │Child │ │Child │ │Child │ │Child │ │Child │   │
│  │Proc 1│ │Proc 2│ │Proc 3│ │Proc 4│ │Proc N│   │
│  │ req  │ │ req  │ │idle  │ │ req  │ │idle  │   │
│  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘   │
└───────────────────────────────────────────────────┘
```
- Each request handled by a dedicated OS process
- Processes are pre-forked (warm pool) to reduce fork latency
- **1 process = 1 request at a time** — blocked I/O blocks nothing else
- Memory usage: ~5–20 MB per child; at 500 concurrent connections, ~2.5–10 GB RAM

#### Worker MPM
- Multiple processes, each with multiple threads
- Reduces memory vs Prefork but PHP must be thread-safe

#### Event MPM (Apache 2.4+)
- Thread pool handles keep-alive connections asynchronously
- Similar to Nginx's model; addresses the "C10K" problem for Apache
- Still not as efficient as Nginx due to internal design overhead

### Configuration Model

Apache uses `.htaccess` files for **per-directory** configuration — a powerful feature enabling decentralized config (e.g., shared hosting environments) but at a performance cost: Apache checks for `.htaccess` files in every directory in the request path, even when empty.

```apache
# /etc/apache2/sites-available/myapp.conf
<VirtualHost *:443>
    ServerName example.com
    DocumentRoot /var/www/myapp

    # Per-directory overrides (checked at runtime)
    <Directory /var/www/myapp>
        AllowOverride All       # enables .htaccess
        Require all granted
    </Directory>

    ProxyPass /api http://backend:8080/
    ProxyPassReverse /api http://backend:8080/
</VirtualHost>
```

**Dynamic module loading:** Apache loads modules at runtime (`LoadModule`), allowing fine-grained features without recompile — but also increasing memory footprint.

### Strengths

- **Mature ecosystem:** Thousands of modules (mod_rewrite, mod_security, mod_php, mod_ssl)
- **Per-directory config (`.htaccess`):** Ideal for shared hosting and CMS deployments (WordPress, Drupal)
- **mod_php integration:** PHP runs in-process (fastest PHP execution model, no FastCGI overhead)
- **Extremely well-documented** with decades of tutorials and StackOverflow answers

### Weaknesses

- **High memory per connection:** Struggles with thousands of concurrent long-lived connections
- **Keep-alive inefficiency (Prefork/Worker MPM):** A process is tied up for the entire keep-alive duration even when idle
- **`.htaccess` overhead:** Filesystem stat calls on every request path directory
- **Slower static file serving** compared to Nginx under high concurrency

---

## Nginx

### History & Philosophy

Nginx (pronounced "engine-x") was created by Igor Sysoev in 2004 explicitly to solve the **C10K problem** — serving 10,000 concurrent connections on a single server. Its architecture is fundamentally different from Apache's.

### Architecture: Asynchronous Event-Driven

```
┌──────────────────────────────────────────────────────────────┐
│                      Nginx Master Process                     │
│  ┌──────────────────────────────────────────────────────┐    │
│  │               Worker Process (1 per CPU core)        │    │
│  │                                                      │    │
│  │  ┌──────────────────────────────────────────────┐   │    │
│  │  │         Event Loop (epoll/kqueue)             │   │    │
│  │  │                                              │   │    │
│  │  │  Connection 1 ──▶ [read] ──▶ [proxy send]   │   │    │
│  │  │  Connection 2 ──▶ [waiting for upstream]    │   │    │
│  │  │  Connection 3 ──▶ [write response]          │   │    │
│  │  │  Connection N ──▶ [keepalive idle]          │   │    │
│  │  └──────────────────────────────────────────────┘   │    │
│  └──────────────────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────────────────┐    │
│  │               Worker Process (core 2)                │    │
│  └──────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

- **Master process:** Reads config, manages worker lifecycle, handles signals
- **Worker processes:** One per CPU core (configurable), each runs a non-blocking event loop
- **No thread-per-connection:** A single worker handles thousands of connections via non-blocking I/O multiplexing (epoll on Linux, kqueue on BSD/macOS)
- **Blocked I/O is the enemy:** Nginx's built-in proxy, file serving, and SSL are all async-safe; calling synchronous PHP directly is not (hence PHP-FPM)

### Memory & Concurrency

A worker process uses ~1–5 MB base memory regardless of connection count. At 10,000 concurrent connections, Nginx uses a fraction of the memory Apache's Prefork MPM would.

```
Nginx: 4 workers × 5 MB base + ~1 KB per connection overhead
       at 10K connections: ~30–50 MB total

Apache (Prefork): 10K processes × 10 MB each = ~100 GB
```

### Configuration Model

Nginx uses a **centralized, hierarchical configuration** in `/etc/nginx/nginx.conf`. There's no `.htaccess` equivalent — all config is loaded once at startup (or reload) into memory.

```nginx
# nginx.conf
worker_processes auto;          # 1 per CPU core

events {
    worker_connections 1024;    # per worker
    use epoll;                  # Linux best-practice
}

http {
    # Upstream backend pool
    upstream app_servers {
        least_conn;
        server app1:8080 weight=3;
        server app2:8080 weight=1;
        keepalive 32;            # persistent connections to backend
    }

    server {
        listen 443 ssl http2;
        server_name example.com;

        ssl_certificate     /etc/ssl/example.crt;
        ssl_certificate_key /etc/ssl/example.key;

        # Static files — served directly from OS page cache
        location /static/ {
            root /var/www;
            expires 1y;
            add_header Cache-Control "public, immutable";
        }

        # Reverse proxy to backend
        location /api/ {
            proxy_pass http://app_servers;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

### Strengths

- **Extremely efficient static file serving** — uses `sendfile()` syscall, bypasses userspace buffers
- **Low memory footprint** — handles tens of thousands of concurrent connections per worker
- **Excellent reverse proxy** — upstream keepalive pools, load balancing, buffering
- **HTTP/2 and HTTP/3 support** (HTTP/3 in preview via QUIC)
- **Zero-downtime reloads:** `nginx -s reload` reloads config without dropping connections
- **Built-in rate limiting, geo-IP, access controls**

### Weaknesses

- **No dynamic `.htaccess`** — configuration changes require a reload (acceptable for most, dealbreaker for shared hosting)
- **PHP requires PHP-FPM** — adds a small FastCGI overhead vs Apache's in-process mod_php
- **Lua/JS scripting** available but less mature than Apache's module ecosystem
- **Nginx Plus** (commercial) required for advanced features (active health checks, dynamic upstreams via API)

---

## Apache vs Nginx: Head-to-Head

| Dimension | Apache | Nginx |
|-----------|--------|-------|
| **Architecture** | Process/thread per connection | Async event-driven |
| **Concurrency model** | Blocking I/O per worker | Non-blocking I/O, epoll |
| **Memory at 10K connections** | ~10–100 GB (Prefork) | ~50–100 MB |
| **Static file performance** | Good | Excellent (sendfile) |
| **Dynamic content** | mod_php (in-process) | PHP-FPM (FastCGI) |
| **Per-directory config** | Yes (.htaccess) | No |
| **Config reload** | Graceful restart | Zero-downtime reload |
| **Config syntax** | Verbose, XML-like directives | Concise, block-based |
| **Module system** | Dynamic (LoadModule) | Static (compiled in) |
| **Reverse proxy** | mod_proxy (functional) | First-class, high-performance |
| **Community & docs** | Enormous, 25+ years | Large, modern focus |
| **Best for** | Shared hosting, CMS, legacy PHP | High-concurrency, APIs, microservices |

**Performance benchmark rule of thumb:**
- Static files: Nginx is 2–3× faster under concurrency
- Dynamic PHP: Comparable when Apache uses Event MPM + PHP-FPM vs Nginx + PHP-FPM
- High concurrency (>1K connections): Nginx wins decisively on memory and throughput

---

## Reverse Proxy Patterns

A reverse proxy sits in front of backend servers, providing load balancing, SSL termination, caching, and security.

### SSL Termination
```
Client ──HTTPS──▶ [Nginx/HAProxy] ──HTTP──▶ Backend
```
Offloads TLS handshake computation from application servers. Backends communicate over plain HTTP on internal network (assumed trusted).

### Load Balancing Algorithms

| Algorithm | Description | Best For |
|-----------|-------------|----------|
| **Round Robin** | Requests distributed sequentially | Uniform request cost |
| **Least Connections** | Routes to server with fewest active connections | Variable request duration |
| **IP Hash** | Client IP always routes to same backend | Session affinity (without sticky cookies) |
| **Weighted** | Backends get proportional traffic | Mixed-capacity server pools |
| **Random** | Random selection | Simple, surprisingly effective |

### Connection Pooling / Upstream Keepalive

```nginx
upstream backend {
    server app1:8080;
    server app2:8080;
    keepalive 64;    # maintain 64 idle connections per worker to backends
}
```

Without keepalive, every proxied request involves a new TCP handshake to the backend — devastating at high QPS. Keepalive pools reuse connections, dramatically reducing latency and backend OS overhead.

### Health Checks

**Passive health checks** (both Nginx OSS and HAProxy): Remove a backend from rotation after N consecutive failures.

**Active health checks** (HAProxy, Nginx Plus, Traefik): Proactively probe backends on a schedule, remove unhealthy ones before clients hit them.

### Request Buffering

Nginx buffers the full client request before forwarding to backend, protecting slow backends from slow clients (slow-loris protection). Configurable with `proxy_request_buffering`.

---

## HAProxy

HAProxy (High Availability Proxy) is the **gold standard for TCP/HTTP load balancing**. It is not a general-purpose web server — it does one thing (proxying) and does it better than anyone else.

### Architecture

HAProxy uses an **event-driven, single-threaded model** similar to Nginx (multi-threaded in newer versions). It is written in highly optimized C and can handle millions of connections per second on commodity hardware.

```
                    ┌─────────────────────────────────┐
                    │           HAProxy               │
Internet ──────────▶│  Frontend (bind :443)           │
                    │      │                           │
                    │  ACL matching / routing          │
                    │      │                           │
                    │  Backend pool                   │
                    │  ├── server app1 weight 1       │
                    │  ├── server app2 weight 1       │
                    │  └── server app3 weight 1       │
                    └─────────────────────────────────┘
```

### Key Features

- **Layer 4 (TCP) and Layer 7 (HTTP) proxying** — can load balance any TCP protocol, not just HTTP
- **ACL-based routing:** Route based on URL path, headers, source IP, TLS SNI, etc.
- **Active health checks** (built-in, no extra license)
- **Sticky sessions** via cookie injection or source IP
- **Statistics dashboard** (`/haproxy-stats`) — real-time backend health, request rates, error rates
- **Runtime API:** Change backend weights, drain servers, add/remove backends without restart
- **SSL/TLS termination** with SNI-based routing to multiple backends

### Sample Configuration

```haproxy
global
    maxconn 50000
    log /dev/log local0

defaults
    mode http
    timeout connect 5s
    timeout client  30s
    timeout server  30s
    option httplog
    option forwardfor
    option redispatch
    retries 3

frontend web_frontend
    bind *:443 ssl crt /etc/ssl/example.pem
    default_backend web_backend

    # Route /api to API backend
    acl is_api path_beg /api
    use_backend api_backend if is_api

backend web_backend
    balance roundrobin
    option httpchk GET /healthz
    server web1 10.0.0.1:80 check weight 1
    server web2 10.0.0.2:80 check weight 1

backend api_backend
    balance leastconn
    option httpchk GET /api/healthz
    server api1 10.0.1.1:8080 check
    server api2 10.0.1.2:8080 check
```

### When HAProxy Shines

- **Pure load balancing** at very high throughput (financial trading, telco)
- **TCP proxying** (MySQL, Redis, RabbitMQ clusters)
- **Advanced traffic management** with ACL rules and runtime reconfiguration
- **Blue/green deployments** via weight adjustments through the API

---

## Caddy

Caddy is a modern web server written in Go, distinguished by its **automatic HTTPS by default** and clean configuration syntax.

### Key Features

- **Automatic TLS via Let's Encrypt/ZeroSSL**: Caddy handles certificate issuance, renewal, and OCSP stapling automatically — zero configuration for HTTPS
- **Caddyfile**: Human-friendly configuration syntax
- **JSON API**: Full dynamic configuration via REST API at runtime
- **HTTP/2 and HTTP/3 (QUIC)** supported out of the box
- **Extensible via plugins** written in Go

### Sample Caddyfile

```caddyfile
example.com {
    # Automatic HTTPS — no ssl_certificate directives needed
    reverse_proxy /api/* localhost:8080
    file_server /static/* {
        root /var/www
    }
    encode gzip zstd
    log {
        output file /var/log/caddy/access.log
    }
}
```

That's it — Caddy handles cert issuance, renewal, HTTP→HTTPS redirect, and HTTP/2 automatically.

### When to Use Caddy

- **Small to medium projects** where operational simplicity is paramount
- **Teams without a dedicated ops person** — zero-touch TLS is a huge win
- **Self-hosted services** where Let's Encrypt integration saves hours of cert management
- **Development environments** needing HTTPS (Caddy can issue locally-trusted certs via `caddy serve --internal-certs`)
- **Not ideal** for very high-throughput production where Nginx/HAProxy's C-based performance matters

---

## Traefik

Traefik is a **cloud-native, dynamic reverse proxy** designed for microservices and container orchestration environments.

### Key Features

- **Auto-discovery:** Traefik watches Docker, Kubernetes, Consul, and etcd — it automatically creates routes when containers start/stop, no manual config required
- **Dynamic configuration:** Routes update in real time without restarts
- **Let's Encrypt integration** (similar to Caddy)
- **Middleware system:** Rate limiting, authentication, circuit breakers, retry logic as composable layers
- **Dashboard UI** for real-time traffic visualization
- **TCP and UDP proxying** in addition to HTTP

### Kubernetes Ingress Example

```yaml
# Traefik IngressRoute (CRD)
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: myapp
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`example.com`) && PathPrefix(`/api`)
      kind: Rule
      services:
        - name: api-service
          port: 8080
      middlewares:
        - name: rate-limit
        - name: auth-middleware
  tls:
    certResolver: letsencrypt
```

No YAML reload needed when `api-service` scales up — Traefik detects new endpoints automatically.

### When to Use Traefik

- **Kubernetes environments** where dynamic routing is essential (containers start/stop constantly)
- **Docker Compose microservices** — label-based auto-routing
- **Teams that want to avoid writing and maintaining nginx/haproxy config** for every new service
- **Not ideal** for bare-metal or VM-based setups where HAProxy/Nginx's performance characteristics and operational familiarity are preferred

---

## Comparison Matrix

| Feature | Apache | Nginx | HAProxy | Caddy | Traefik |
|---------|--------|-------|---------|-------|---------|
| **Primary Role** | Web server | Web server + proxy | Load balancer | Web server + proxy | Dynamic proxy |
| **Architecture** | Process/thread | Async event | Async event | Async (Go) | Async (Go) |
| **Static file serving** | ✅ Good | ✅ Excellent | ❌ No | ✅ Good | ⚠️ Basic |
| **Reverse proxy** | ✅ mod_proxy | ✅ Excellent | ✅ Best-in-class | ✅ Good | ✅ Good |
| **Auto HTTPS** | ❌ Manual | ❌ Manual | ❌ Manual | ✅ Native | ✅ Native |
| **Dynamic config (no restart)** | ⚠️ .htaccess only | ⚠️ Reload needed | ✅ Runtime API | ✅ JSON API | ✅ Native |
| **L4 (TCP) proxy** | ❌ | ✅ stream{} | ✅ | ✅ | ✅ |
| **Health checks (active)** | ❌ | ❌ (Plus only) | ✅ | ✅ | ✅ |
| **Container/K8s native** | ❌ | ⚠️ (with config) | ⚠️ (with config) | ⚠️ | ✅ Native |
| **Stats/Observability** | mod_status | stub_status | Rich dashboard | Prometheus metrics | Dashboard + Prometheus |
| **Performance ceiling** | Medium | Very High | Highest | High | High |
| **Config complexity** | High | Medium | Medium | Low | Low-Medium |
| **Ecosystem maturity** | Highest | Very High | High | Medium | Medium |

---

## When to Use What

### Use **Apache** when:
- Running **shared hosting** or environments where tenants need `.htaccess` control
- Migrating or maintaining **legacy PHP applications** tightly coupled to mod_php
- You need a specific **Apache-only module** (mod_security WAF, mod_auth_kerb, etc.)
- Your ops team has deep Apache expertise and no appetite to retrain

### Use **Nginx** when:
- You need a **high-performance reverse proxy and/or static file server**
- Building **microservices** behind an API gateway
- **SSL/TLS termination** at scale for many backend services
- You want a **battle-tested, high-concurrency** server with broad hosting support
- **Container deployments** using standard Nginx images with custom configs

### Use **HAProxy** when:
- Pure **TCP or HTTP load balancing** at extreme throughput is the requirement
- You need **fine-grained traffic management** with ACL rules and runtime API
- Load balancing **non-HTTP protocols** (MySQL, PostgreSQL, Redis, gRPC)
- **Zero-downtime deployments** with server drain and weight manipulation
- Financial/telco environments where raw performance and reliability SLAs are paramount

### Use **Caddy** when:
- **Automatic HTTPS** without cert management overhead is a priority
- Building **small to medium services** where operational simplicity beats raw performance
- **Developer environments** needing local HTTPS
- You want a **single binary** with no external dependencies or complex config

### Use **Traefik** when:
- Running **Kubernetes** and need an Ingress controller that auto-discovers services
- Using **Docker Compose** with rapidly changing service topology
- You want **middleware composability** (auth, rate limiting, retry) without writing custom Lua/config
- **GitOps workflows** where routing config lives as Kubernetes CRDs alongside app manifests

---

## Architecture Pattern: Layered Proxy Stack

In production systems, multiple layers often work together:

```
Internet
    │
    ▼
[CDN — Cloudflare/CloudFront]    ← Edge caching, DDoS protection, global anycast
    │
    ▼
[HAProxy / Cloud LB]             ← TCP/L4 routing, health checks, failover
    │
    ▼
[Nginx / Traefik]                ← SSL termination, L7 routing, rate limiting
    │
    ▼
[App Servers / Services]
    │
    ▼
[Redis / Databases]
```

**When to collapse layers:** For smaller systems, Nginx alone can handle SSL termination and load balancing without a dedicated HAProxy tier. Caddy or Traefik can replace both layers in container environments. Add layers only when their specific capabilities are genuinely needed.

---

## Further Reading

- [Nginx Documentation](https://nginx.org/en/docs/)
- [Apache HTTP Server Docs](https://httpd.apache.org/docs/)
- [HAProxy Configuration Manual](https://www.haproxy.org/download/2.8/doc/configuration.txt)
- [The C10K Problem](http://www.kegel.com/c10k.html) — Dan Kegel's original paper motivating Nginx's design
- [Caddy Documentation](https://caddyserver.com/docs/)
- [Traefik Documentation](https://doc.traefik.io/traefik/)
- [NGINX vs Apache: Our View After 15 Years](https://www.nginx.com/blog/nginx-vs-apache-our-view/) (Nginx blog)
- [HAProxy Performance Tuning](https://www.haproxy.com/blog/performance-tuning-haproxy/)
