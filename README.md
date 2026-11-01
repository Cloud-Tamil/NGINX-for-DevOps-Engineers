# NGINX for DevOps Engineers

> A practical, end-to-end guide to learning NGINX—from your first static website to reverse proxies, load balancing, HTTPS, Docker, Kubernetes, security, troubleshooting, and production architecture.

![NGINX](https://img.shields.io/badge/NGINX-DevOps%20Guide-009639?logo=nginx&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04-E95420?logo=ubuntu&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-supported-2496ED?logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-concepts-326CE5?logo=kubernetes&logoColor=white)
![License](https://img.shields.io/badge/license-educational-blue)

## Overview

NGINX (pronounced **engine-x**) is an open-source, event-driven web server and traffic-management platform. It is commonly used as a:

- Static web server

- Reverse proxy

- Load balancer

- SSL/TLS termination point

- HTTP cache

- Rate limiter

- TCP/UDP proxy

- API gateway component

- Kubernetes ingress or gateway component

This guide is written from a DevOps and platform-engineering perspective. It combines concepts with hands-on commands so that you can install, configure, operate, secure, troubleshoot, and deploy NGINX in realistic environments.

## What you will learn

By completing this guide, you will be able to:

- Explain NGINX's master/worker and event-driven architecture.

- Install and operate NGINX on Ubuntu.

- Serve static websites with server blocks.

- Route traffic to Python, Node.js, Java, or other backend services.

- Configure round-robin, least-connections, IP-hash, and weighted load balancing.

- Terminate HTTPS and automate certificate renewal with Certbot.

- Configure caching, compression, rate limiting, and security headers.

- Run NGINX in Docker and Docker Compose.

- Understand NGINX's role in Kubernetes networking.

- Monitor logs, inspect configuration, and troubleshoot common HTTP errors.

- Apply sensible performance and production-architecture practices.

- Prepare for common NGINX and DevOps interview questions.

## Course modules

This guide is organized around the following practical learning sequence:

1. **What is NGINX?** Understand the web-server, reverse-proxy, load-balancer, cache, and TLS-termination use cases.

1. **Serving static websites** Publish HTML, CSS, JavaScript, and image files from an NGINX document root.

1. **Reverse proxy configuration** Forward public requests to an application running on a private backend port.

1. **Load balancing setup** Distribute traffic across multiple backend instances using common balancing methods.

1. **Upstream servers** Group backend services in `upstream` blocks and route traffic to the group with `proxy_pass`.

1. **Port forwarding** Understand the difference between NGINX application proxying and network-level port forwarding, then expose a backend safely through ports `80` and `443`.

1. **NGINX folder structure** Learn where configurations, enabled sites, web files, binaries, and logs live on Ubuntu.

1. **Configuration file explanation** Read the `main`, `events`, `http`, `server`, `upstream`, and `location` contexts.

1. **SSL certificate basics** Understand certificates, private keys, HTTPS termination, HTTP-to-HTTPS redirects, and renewal.

1. **Real-time hands-on demonstrations** Apply every concept with `nginx -t`, `systemctl reload`, `curl`, log inspection, and deliberately tested backend failures.

### Real-time demonstration checklist

Each demonstration should follow the same safe operational loop:

```
Explain the concept
    -> Edit or create the configuration
    -> Test with: sudo nginx -t
    -> Reload with: sudo systemctl reload nginx
    -> Verify with: curl and browser
    -> Observe: access.log and error.log
    -> Break and troubleshoot the setup
```

Recommended live demonstrations:

| Demonstration | Observable result |
| --- | --- |
| Serve `index.html` | Browser receives a static page from NGINX |
| Proxy to a backend on port `3000` | NGINX returns the backend response on port `80` |
| Add two or more upstream servers | Requests are distributed across backend instances |
| Stop one backend | NGINX continues using available servers, subject to configuration |
| Forward public port `80` to a private service port | Clients do not need direct access to the backend port |
| Inspect `/etc/nginx` and `/var/log/nginx` | Configuration and request behavior become visible |
| Install a development or public certificate | Browser can connect over HTTPS |
| Create a syntax error, then repair it | Troubleshooting workflow is demonstrated safely |

## How NGINX fits into an application

A typical request path looks like this:

```
Users
  |
  v
DNS / CDN / WAF
  |
  v
NGINX :80 or :443
  |\
  | \__ Static files
  |
  +---- Reverse proxy ----> API services
  |
  +---- Load balancer ----> Backend 1 / Backend 2 / Backend 3
```

NGINX sits at the edge of the application. It receives client traffic, applies routing and policy, and forwards requests to the appropriate service. It does **not** replace your application code, database, queue, or monitoring platform.

## Contents

- [Prerequisites](#prerequisites)

- [Course modules](#course-modules)

- [Quick start on Ubuntu](#quick-start-on-ubuntu)

- [Important files and directories](#important-files-and-directories)

- [NGINX configuration model](#nginx-configuration-model)

- [Hands-on labs](#hands-on-labs)
  - [Lab 1: Host a static website](#lab-1-host-a-static-website)
  - [Lab 2: Configure a reverse proxy](#lab-2-configure-a-reverse-proxy)
  - [Lab 3: Add load balancing](#lab-3-add-load-balancing)
  - [Lab 4: Enable HTTPS](#lab-4-enable-https)
  - [Lab 5: Add caching, compression, and rate limiting](#lab-5-add-caching-compression-and-rate-limiting)
  - [Lab 6: Run NGINX with Docker](#lab-6-run-nginx-with-docker)

- [Kubernetes concepts](#kubernetes-concepts)

- [Security checklist](#security-checklist)

- [Monitoring and troubleshooting](#monitoring-and-troubleshooting)

- [Performance guidelines](#performance-guidelines)

- [Production-style capstone project](#production-style-capstone-project)

- [NGINX Open Source vs NGINX Plus](#nginx-open-source-vs-nginx-plus)

- [NGINX compared with alternatives](#nginx-compared-with-alternatives)

- [Command cheat sheet](#command-cheat-sheet)

- [Seven-day learning roadmap](#seven-day-learning-roadmap)

- [Interview preparation](#interview-preparation)

- [Useful resources](#useful-resources)

- [Contributing](#contributing)

## Prerequisites

You should have:

- Basic Linux command-line knowledge

- A working Ubuntu 24.04 machine, VM, or cloud instance

- `sudo` access

- At least 2 vCPUs and 4 GB RAM for the basic lab

- A web browser and `curl`

- Optional: Docker, Docker Compose, Kubernetes, `ab`, or `hey`

For a local VirtualBox lab, use a private or host-only network unless you intentionally need public access. Do not expose an experimental server directly to the internet without applying firewall and security controls.

## Quick start on Ubuntu

### 1. Update the system

```bash
sudo apt update
sudo apt upgrade -y
```

### 2. Install NGINX

```bash
sudo apt install -y nginx
```

### 3. Start and enable the service

```bash
sudo systemctl enable --now nginx
sudo systemctl status nginx
```

### 4. Verify the installation

```bash
nginx -v
sudo nginx -t
curl -I http://localhost
```

### 5. Allow web traffic through UFW

Only run this if UFW is enabled and you intend to allow HTTP/HTTPS traffic:

```bash
sudo ufw allow 'Nginx Full'
sudo ufw status
```

Find the machine's IP address:

```bash
hostname -I
```

Then visit `http://YOUR_UBUNTU_IP` in a browser.

## Important files and directories

| Path | Purpose |
| --- | --- |
| `/etc/nginx/nginx.conf` | Main NGINX configuration |
| `/etc/nginx/sites-available/` | Available Debian/Ubuntu virtual-host configurations |
| `/etc/nginx/sites-enabled/` | Enabled virtual-host configurations |
| `/etc/nginx/conf.d/` | Additional configuration fragments |
| `/var/www/html/` | Default website directory |
| `/var/log/nginx/access.log` | Records incoming requests |
| `/var/log/nginx/error.log` | Records errors and operational events |
| `/var/run/nginx.pid` | PID file location on many installations |
| `/usr/sbin/nginx` | NGINX executable |

On Ubuntu, a common workflow is:

1. Create a configuration in `sites-available`.

1. Enable it with a symbolic link in `sites-enabled`.

1. Validate it with `sudo nginx -t`.

1. Apply it with `sudo systemctl reload nginx`.

## NGINX configuration model

NGINX configuration is organized into nested contexts:

```
main
├── events
└── http
    ├── upstream
    └── server
        └── location
```

A minimal configuration looks like this:

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    sendfile on;
    keepalive_timeout 65;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

Common directives include:

| Directive | Purpose |
| --- | --- |
| `worker_processes` | Number of worker processes |
| `worker_connections` | Connections handled per worker |
| `server` | Defines a virtual server |
| `listen` | Address and port to listen on |
| `server_name` | Hostname matched by the server block |
| `location` | URL matching and routing rules |
| `root` | Static-file document root |
| `proxy_pass` | Forwards requests to an upstream service |
| `upstream` | Defines a backend server group |
| `access_log` | Request logging |
| `error_log` | Error and operational logging |

## Hands-on labs

### Lab 1: Host a static website

Create a document root and a test page:

```bash
sudo mkdir -p /var/www/mywebsite
sudo tee /var/www/mywebsite/index.html > /dev/null <<'HTML'
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>My NGINX Project</title>
</head>
<body>
  <h1>Hello from NGINX!</h1>
  <p>My first DevOps web server project.</p>
</body>
</html>
HTML
```

Create a server block:

```bash
sudo tee /etc/nginx/sites-available/mywebsite > /dev/null <<'NGINX'
server {
    listen 80;
    server_name _;

    root /var/www/mywebsite;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
NGINX
```

Enable and test it:

```bash
sudo ln -s /etc/nginx/sites-available/mywebsite /etc/nginx/sites-enabled/mywebsite
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
curl http://localhost
```

### Lab 2: Configure a reverse proxy

NGINX can expose a backend application without requiring clients to connect to the backend port directly.

Create a small Python backend:

```bash
mkdir -p ~/nginx-lab/backend
cd ~/nginx-lab/backend
cat > app.py <<'PY'
from http.server import BaseHTTPRequestHandler, HTTPServer

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        message = b"Hello from Backend Application!\n"
        self.send_response(200)
        self.send_header("Content-Type", "text/plain")
        self.send_header("Content-Length", str(len(message)))
        self.end_headers()
        self.wfile.write(message)

HTTPServer(("127.0.0.1", 3000), Handler).serve_forever()
PY
python3 app.py
```

In another terminal, create the proxy configuration:

```bash
sudo tee /etc/nginx/sites-available/reverse-proxy > /dev/null <<'NGINX'
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
NGINX

sudo ln -sf /etc/nginx/sites-available/reverse-proxy /etc/nginx/sites-enabled/reverse-proxy
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
curl http://localhost
```

Request flow:

```
Client -> NGINX :80 -> Python application :3000 -> NGINX -> Client
```

### Lab 3: Add load balancing

Define a backend pool with multiple application servers:

```
upstream backend_servers {
    server 10.0.0.11:3000;
    server 10.0.0.12:3000;
    server 10.0.0.13:3000;
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Supported examples:

```
# Default: round robin
upstream app_round_robin {
    server 10.0.0.11:3000;
    server 10.0.0.12:3000;
}

# Send new requests to the least-busy server
upstream app_least_conn {
    least_conn;
    server 10.0.0.11:3000;
    server 10.0.0.12:3000;
}

# Basic client-IP affinity
upstream app_ip_hash {
    ip_hash;
    server 10.0.0.11:3000;
    server 10.0.0.12:3000;
}

# Approximate traffic ratio: 3:2:1
upstream app_weighted {
    server 10.0.0.11:3000 weight=3;
    server 10.0.0.12:3000 weight=2;
    server 10.0.0.13:3000 weight=1;
}
```

| Method | Good fit |
| --- | --- |
| Round robin | Backends have similar capacity and request cost |
| Least connections | Requests have different processing times |
| IP hash | Basic client-IP session affinity is needed |
| Weighted | Backends have different capacities |

Open-source NGINX provides passive failure handling. Verify the feature set you need before assuming that active health checks or other advanced capabilities are available in your edition.

### Lab 4: Enable HTTPS

For a real public domain, install Certbot and the NGINX plugin:

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d example.com -d www.example.com
sudo certbot renew --dry-run
```

A simplified HTTPS server block looks like this:

```
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;

    location / {
        proxy_pass http://127.0.0.1:3000;
    }
}

server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
```

A public certificate normally requires a real domain pointing to the server and reachable validation ports. For a local VM, use HTTP or a locally trusted development certificate. Never commit or publicly expose a private TLS key.

### Lab 5: Add caching, compression, and rate limiting

#### Proxy caching

```
http {
    proxy_cache_path /var/cache/nginx
        levels=1:2
        keys_zone=my_cache:10m
        max_size=500m
        inactive=60m;

    server {
        listen 80;

        location / {
            proxy_cache my_cache;
            proxy_cache_valid 200 10m;
            proxy_pass http://127.0.0.1:3000;
        }
    }
}
```

Do not cache authenticated, personalized, or sensitive responses unless the cache key and bypass rules have been deliberately designed and tested.

#### GZIP compression

```
http {
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_comp_level 5;

    gzip_types
        text/plain
        text/css
        application/json
        application/javascript
        application/xml;
}
```

#### API rate limiting

```
http {
    limit_req_zone $binary_remote_addr
        zone=api_limit:10m
        rate=10r/s;

    server {
        listen 80;

        location /api/ {
            limit_req zone=api_limit burst=20 nodelay;
            proxy_pass http://127.0.0.1:3000;
        }
    }
}
```

This example allows an average of 10 requests per second per client IP with a burst allowance of 20 requests. Rate limiting reduces abuse and protects backends, but it is not a complete DDoS-protection strategy.

### Lab 6: Run NGINX with Docker

Example project layout:

```
nginx-docker/
├── Dockerfile
├── compose.yaml
├── nginx.conf
└── html/
    └── index.html
```

`Dockerfile`:

```
FROM nginx:stable

COPY html/ /usr/share/nginx/html/

EXPOSE 80
```

`compose.yaml`:

```yaml
services:
  nginx:
    build: .
    container_name: nginx-web
    ports:
      - "8080:80"
    restart: unless-stopped
```

Build and run it:

```bash
docker compose up -d --build
docker ps
docker logs nginx-web
curl http://localhost:8080
docker compose down
```

## Kubernetes concepts

NGINX commonly appears in Kubernetes in two ways:

1. As a reverse proxy inside an application Pod.

1. As an ingress or gateway component that routes external HTTP/HTTPS traffic to Kubernetes Services.

A common request path is:

```
Internet
   |
   v
Cloud LoadBalancer
   |
   v
Ingress / Gateway
   |
   v
Kubernetes Service
   |
   +--> Pod 1
   +--> Pod 2
   +--> Pod 3
```

Example Ingress resource:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp-service
                port:
                  number: 80
```

Apply and inspect resources:

```bash
kubectl apply -f ingress.yaml
kubectl get ingress
kubectl get pods -A
kubectl get services -A
```

Kubernetes ingress controllers and Gateway API implementations change over time. For a new production deployment, verify that the controller or gateway implementation you select is actively maintained, supported by your platform, and appropriate for your security and traffic requirements. Do not assume that every project called “NGINX ingress” is the same product.

## Security checklist

Before exposing NGINX to production traffic:

- Use HTTPS and protect certificate private keys.

- Prefer modern TLS versions such as TLS 1.2 and TLS 1.3.

- Keep NGINX, the operating system, and dependencies updated.

- Disable unnecessary version disclosure with `server_tokens off;`.

- Configure a firewall and expose only required ports.

- Apply rate limits to sensitive or expensive endpoints.

- Add security headers appropriate to the application.

- Restrict HTTP methods only after confirming application requirements.

- Protect admin, health, and internal endpoints.

- Avoid caching private or personalized content.

- Set appropriate request-body and timeout limits.

- Review file ownership and permissions for web roots and secrets.

- Log access and errors, then monitor the logs.

Example baseline headers:

```
server_tokens off;

server {
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
}
```

Security headers and method restrictions must be tailored to the application. For example, an API using `PUT` or `DELETE` must not accidentally block those methods.

## Monitoring and troubleshooting

### Essential operational commands

```bash
sudo systemctl status nginx
sudo nginx -t
sudo nginx -T
sudo systemctl reload nginx
sudo systemctl restart nginx
sudo ss -tulpn | grep nginx
ps aux | grep nginx
curl -I http://localhost
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

Prefer a configuration reload for normal configuration changes. A reload applies the new configuration while allowing existing workers to finish gracefully. Use a restart only when it is actually necessary.

### Common errors

| Error | Typical cause | First checks |
| --- | --- | --- |
| `403 Forbidden` | Permissions or access rules | File ownership, mode bits, `location` rules |
| `404 Not Found` | Missing file or route | `root`, `try_files`, URL path |
| `502 Bad Gateway` | Upstream is down or unreachable | Backend process, port, `proxy_pass`, firewall |
| `504 Gateway Timeout` | Upstream is too slow | Backend latency and proxy timeouts |
| `413 Request Entity Too Large` | Request body exceeds limit | `client_max_body_size` |
| `500 Internal Server Error` | Application or upstream error | Application and NGINX logs |
| Connection refused | No process listening | `ss`, service status, target host and port |

A useful troubleshooting sequence is:

1. Validate the NGINX configuration.

1. Check whether NGINX is running and listening on the expected port.

1. Test the backend directly with `curl`.

1. Inspect the error log.

1. Test through NGINX with headers and verbose output.

1. Confirm firewall, DNS, permissions, and timeout settings.

## Performance guidelines

A starting point for a small reverse-proxy host is:

```
worker_processes auto;

events {
    worker_connections 2048;
}

http {
    sendfile on;
    tcp_nopush on;
    keepalive_timeout 30;
    client_max_body_size 20M;
}

server {
    location / {
        proxy_connect_timeout 5s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        proxy_pass http://backend_servers;
    }
}
```

Do not blindly increase worker counts, connection limits, body sizes, or timeouts. Performance depends on CPU, memory, file descriptors, network capacity, backend behavior, workload shape, and upstream dependencies. Measure first, change one variable at a time, and test under representative load.

## Production-style capstone project

### Reverse Proxy and Load Balancer Lab

Build a small production-style environment:

```
Client
  |
  v
NGINX :8080
  |
  v
Load balancer
  |
  +--> API 1 :3001
  +--> API 2 :3002
  +--> API 3 :3003
          |
          +--> Database / Redis

Monitoring: Prometheus -> Grafana
```

Suggested phases:

| Phase | Deliverable | Skill |
| --- | --- | --- |
| 1 | Install NGINX | Linux administration |
| 2 | Host a static website | Web serving |
| 3 | Deploy three backend APIs | Application deployment |
| 4 | Configure reverse proxying | Traffic routing |
| 5 | Add load balancing | Scalability |
| 6 | Configure HTTPS | TLS |
| 7 | Add rate limiting | Security |
| 8 | Add caching | Performance |
| 9 | Containerize the stack | Docker |
| 10 | Deploy the routing layer on Kubernetes | Cloud-native networking |
| 11 | Add metrics and dashboards | Observability |
| 12 | Perform load testing | Performance engineering |

Basic load tests:

```bash
curl http://localhost:8080/
ab -n 1000 -c 20 http://localhost:8080/
# Or, if installed:
hey -n 1000 -c 20 http://localhost:8080/
```

Start with NGINX and lightweight backend processes on a 4 GB / 2 vCPU VM. Add Prometheus and Grafana later rather than running every component at once.

## NGINX Open Source vs NGINX Plus

| Capability | NGINX Open Source | NGINX Plus |
| --- | --- | --- |
| Static web serving | Yes | Yes |
| Reverse proxy | Yes | Yes |
| Basic load balancing | Yes | Yes |
| SSL/TLS termination | Yes | Yes |
| Caching | Yes | Yes |
| Failure handling | Passive capabilities | Extended capabilities |
| Built-in monitoring | More limited | Extended features |
| Support model | Community | Commercial |
| Cost | Free | Paid |

NGINX Open Source is a strong starting point for learning, personal projects, and many standard reverse-proxy deployments. Evaluate requirements carefully before selecting an enterprise edition.

## NGINX compared with alternatives

| Tool | Common strength |
| --- | --- |
| NGINX | Web serving, reverse proxying, caching, and load balancing |
| Apache HTTP Server | Traditional web hosting and a broad module ecosystem |
| HAProxy | Specialized high-performance proxying and traffic distribution |
| Traefik | Dynamic routing for container and cloud-native environments |

The best tool depends on traffic patterns, team experience, platform integration, observability, and operational requirements.

## Command cheat sheet

| Command | Purpose |
| --- | --- |
| `nginx -v` | Show version |
| `nginx -V` | Show version and build options |
| `sudo nginx -t` | Test configuration syntax |
| `sudo nginx -T` | Print the complete effective configuration |
| `sudo systemctl start nginx` | Start service |
| `sudo systemctl stop nginx` | Stop service |
| `sudo systemctl restart nginx` | Restart service |
| `sudo systemctl reload nginx` | Gracefully reload configuration |
| `sudo systemctl status nginx` | Show service status |
| `sudo nginx -s quit` | Graceful shutdown |
| `sudo nginx -s reopen` | Reopen log files |
| `sudo tail -f /var/log/nginx/access.log` | Follow access log |
| `sudo tail -f /var/log/nginx/error.log` | Follow error log |

## Seven-day learning roadmap

### Day 1 — Fundamentals

Learn NGINX's architecture, installation, worker processes, configuration contexts, and essential commands.

### Day 2 — Web server

Serve static files and practice server blocks, `location` matching, document roots, and domains.

### Day 3 — Reverse proxy

Connect NGINX to a Python, Node.js, Java, or other backend application.

### Day 4 — Load balancing

Configure upstream groups, balancing algorithms, backend failures, and request distribution.

### Day 5 — Security

Configure HTTPS, security headers, rate limiting, access control, and safe secrets handling.

### Day 6 — Docker

Build and run a containerized NGINX deployment with Docker Compose.

### Day 7 — Production practices

Review logging, monitoring, troubleshooting, performance testing, and high-availability architecture.

## Interview preparation

Be ready to explain:

1. What is NGINX and why is it widely used?

1. What is the difference between a forward proxy and a reverse proxy?

1. How does NGINX differ from Apache?

1. What is an `upstream` block?

1. What does `proxy_pass` do?

1. What is the difference between HTTP `502` and `504`?

1. What is the difference between reload and restart?

1. What is SSL/TLS termination?

1. How do you troubleshoot an NGINX outage?

1. How does NGINX distribute requests across backends?

A strong practical demonstration is often more valuable than memorizing definitions: deploy an application behind NGINX, add HTTPS and load balancing, then diagnose a deliberately broken upstream.

## Useful resources

- [NGINX Documentation](https://nginx.org/en/docs/)

- [NGINX Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html)

- [NGINX Reverse Proxy Guide](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)

- [NGINX Load Balancing Guide](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)

- [NGINX HTTPS Configuration](https://docs.nginx.com/nginx/admin-guide/security-controls/terminating-ssl-http/)

- [Certbot Documentation](https://certbot.eff.org/)

- [Kubernetes Ingress Documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/)

- [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/)

Always check the current upstream documentation before applying configuration to a production system.

## Contributing

Contributions are welcome. If you find an inaccurate command, unclear explanation, outdated Kubernetes guidance, or a missing practical example:

1. Open an issue describing the problem.

1. Include the operating system and NGINX version where relevant.

1. Explain the expected behavior.

1. Submit a focused pull request with tested documentation or configuration.

When contributing configuration examples, avoid real domains, IP addresses, credentials, private keys, and production secrets.

## License

This README is educational documentation. Add a repository-specific license if you publish the guide as a software or documentation project.

---

**Learn the concepts, run the labs, measure the behavior, and validate every production change.**
