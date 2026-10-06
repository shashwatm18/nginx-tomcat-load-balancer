# Nginx-Tomcat Load Balancer

## Overview

This project demonstrates a web application hosting architecture using **Nginx, Apache HTTP Server, and Apache Tomcat**.

Nginx is configured as a load balancer to distribute incoming HTTP requests across two Apache Tomcat application servers using the **Round-Robin** load-balancing algorithm.

The project also includes application hosting, reverse proxy configuration, performance testing, and backend failure testing.

---

## Architecture

```text
                              Client
                                |
                                | HTTP :80
                                v
                     +----------------------+
                     |      Nginx LB        |
                     |      Machine 1       |
                     |   10.156.195.129     |
                     +----------+-----------+
                                |
                         Round-Robin
                       /              \
                      /                \
                     v                  v
          +-------------------+  +-------------------+
          |     Tomcat 1      |  |     Tomcat 2      |
          |     Machine 2     |  |     Machine 3     |
          |  10.156.195.38    |  |  10.156.195.84   |
          |      :8080        |  |      :8080        |
          +-------------------+  +-------------------+
```

---

## Infrastructure

| Component | Hostname | IP Address | Port | Role |
|---|---|---|---|---|
| Nginx | `APMOSYSLT0363` | `10.156.195.129` | `80` | Load Balancer |
| Tomcat Server 1 | `APMOSYSLT0953` | `10.156.195.38` | `8080` | Application Server |
| Tomcat Server 2 | `shashwat-mishra-K55VJ` | `10.156.195.84` | `8080` | Application Server |

All three servers are connected through the same network:

```text
10.156.195.0/24
```

---

## Technologies

- Linux
- Nginx
- Apache HTTP Server
- Apache Tomcat
- Java
- Git
- Bash
- ApacheBench
- JMeter

---

## Project Components

### Nginx

Nginx acts as the frontend load balancer.

Responsibilities:

- Accept incoming HTTP requests
- Act as a reverse proxy
- Distribute requests between Tomcat servers
- Perform Round-Robin load balancing
- Handle backend server failures
- Forward client requests to available application servers

Nginx listens on:

```text
10.156.195.129:80
```

---

### Apache Tomcat

Two independent Tomcat application servers are used as backend servers.

#### Tomcat Server 1

```text
Host: APMOSYSLT0953
IP:   10.156.195.38
Port: 8080
```

#### Tomcat Server 2

```text
Host: shashwat-mishra-K55VJ
IP:   10.156.195.84
Port: 8080
```

Both servers host the application independently.

---

### Apache HTTP Server

Apache HTTP Server is used to demonstrate HTTP application hosting and
reverse-proxy functionality.

The Apache configuration is kept separate from the Nginx load-balancer
entry point to avoid port conflicts and to allow the components to be
tested independently.

---

## Load Balancing

Nginx uses an upstream group containing both Tomcat servers.

Example configuration:

```nginx
upstream tomcat_backend {
    server 10.156.195.38:8080;
    server 10.156.195.84:8080;
}

server {
    listen 80;

    location / {
        proxy_pass http://tomcat_backend;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

When no other load-balancing method is specified, Nginx distributes
requests using Round-Robin.

Example:

```text
Request 1  -> Tomcat Server 1
Request 2  -> Tomcat Server 2
Request 3  -> Tomcat Server 1
Request 4  -> Tomcat Server 2
Request 5  -> Tomcat Server 1
Request 6  -> Tomcat Server 2
```

The actual distribution depends on request patterns, connection
behavior, and backend response characteristics.

---

## Application Identification

Each Tomcat server hosts a small server-information application.

The application identifies the backend that processed the request.

### Tomcat Server 1

```text
http://10.156.195.38:8080/server-info/
```

Example response:

```text
Tomcat Application Server 1

Hostname: APMOSYSLT0953
Server IP: 10.156.195.38
Tomcat Port: 8080
Server Role: Backend Application Server 1
```

### Tomcat Server 2

```text
http://10.156.195.84:8080/server-info/
```

Example response:

```text
Tomcat Application Server 2

Hostname: shashwat-mishra-K55VJ
Server IP: 10.156.195.84
Tomcat Port: 8080
Server Role: Backend Application Server 2
```

This application makes it possible to visually verify which backend
server handled a request during load-balancing tests.

---

## Request Flow

A normal request follows this path:

```text
Client
  |
  | HTTP Request
  v
Nginx :80
  |
  | Round-Robin
  +----------------------+
  |                      |
  v                      v
Tomcat 1              Tomcat 2
:8080                  :8080
  |                      |
  +----------+-----------+
             |
             v
        HTTP Response
             |
             v
           Client
```

---

## Backend Failure Handling

The architecture is designed to continue serving requests when one
backend becomes unavailable.

Normal state:

```text
             Nginx
            /     \
           /       \
          v         v
     Tomcat 1   Tomcat 2
       UP          UP
```

If Tomcat Server 1 becomes unavailable:

```text
             Nginx
            /     \
           X       \
          X         v
     Tomcat 1   Tomcat 2
       DOWN        UP
```

Nginx can detect backend failures based on its upstream failure
handling configuration and route subsequent requests to an available
backend.

The behavior is validated through backend failure testing.

---

## Tomcat Configuration

Tomcat runs as a dedicated Linux service.

Example systemd service:

```ini
[Unit]
Description=Apache Tomcat Application Server
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_BASE=/opt/tomcat"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Tomcat is accessed on port:

```text
8080
```

---

## Nginx Configuration

The Nginx configuration consists of two primary components:

### Upstream

Defines the backend application servers:

```nginx
upstream tomcat_backend {
    server 10.156.195.38:8080;
    server 10.156.195.84:8080;
}
```

### Reverse Proxy

Forwards incoming requests to the upstream group:

```nginx
location / {
    proxy_pass http://tomcat_backend;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

---

## Connectivity Verification

Basic connectivity between the Nginx server and backend servers can be
verified using:

```bash
ping -c 4 10.156.195.38
```

and:

```bash
ping -c 4 10.156.195.84
```

Tomcat connectivity can be verified using:

```bash
curl -I http://10.156.195.38:8080
```

and:

```bash
curl -I http://10.156.195.84:8080
```

The load balancer can then be tested using:

```bash
curl -I http://10.156.195.129
```

---

## Performance Testing

Performance testing is performed against the Nginx load-balancer
endpoint.

Testing tools include:

- ApacheBench
- JMeter

### Key Metrics

The following metrics are evaluated:

- Throughput
- Requests per second
- Average response time
- Minimum response time
- Maximum response time
- Percentile response times
- Error rate
- Successful requests
- Failed requests
- Backend request distribution
- Resource utilization

---

## ApacheBench Example

A basic concurrent request test can be performed using:

```bash
ab -n 1000 -c 50 http://10.156.195.129/
```

Where:

```text
-n  Number of requests
-c  Number of concurrent requests
```

For example:

```text
1000 total requests
50 concurrent requests
```

The results can be used to evaluate Nginx throughput and backend
response behavior.

---

## JMeter Testing

JMeter can be used for more advanced performance testing scenarios.

The test plan can include:

- Thread Groups
- HTTP Request samplers
- HTTP Header Manager
- Assertions
- Listeners
- CSV Data Set Config
- Response time measurements
- Throughput measurements

JMeter requests should target the Nginx load-balancer endpoint rather
than directly targeting an individual Tomcat server when validating
load-balancing behavior.

```text
JMeter
   |
   v
Nginx :80
   |
   +----> Tomcat 1 :8080
   |
   +----> Tomcat 2 :8080
```

---

## Failure Testing

Backend failure scenarios include:

### Tomcat Server 1 Failure

Stop Tomcat Server 1:

```bash
sudo systemctl stop tomcat
```

Generate requests through Nginx and verify that the application remains
available through Tomcat Server 2.

Restore Tomcat Server 1:

```bash
sudo systemctl start tomcat
```

### Tomcat Server 2 Failure

The same test can be performed against Tomcat Server 2.

The purpose of failure testing is to verify:

- Backend failure detection
- Request routing behavior
- Application availability
- Recovery after backend restoration

---

## Monitoring

During performance and failure testing, system-level metrics can be
observed using standard Linux commands.

### CPU and Memory

```bash
top
```

or:

```bash
free -h
```

### Disk Usage

```bash
df -h
```

### Network Connections

```bash
ss -tulnp
```

### Process Monitoring

```bash
ps aux
```

### Nginx Logs

```bash
sudo tail -f /var/log/nginx/access.log
```

```bash
sudo tail -f /var/log/nginx/error.log
```

### Tomcat Logs

```bash
sudo tail -f /opt/tomcat/logs/catalina.out
```

---

## Security Considerations

The project can be extended with the following security controls:

- Restrict backend Tomcat ports to trusted hosts
- Allow external HTTP traffic only to Nginx
- Configure Linux firewall rules
- Disable unnecessary services
- Run Tomcat using a dedicated non-root user
- Configure HTTPS termination at Nginx
- Implement security headers
- Protect administrative endpoints
- Apply appropriate file and directory permissions

---

## Future Enhancements

The architecture can be extended with:

- HTTPS/TLS termination
- Domain-based routing
- Health-check mechanisms
- Session persistence
- Rate limiting
- Access control
- Centralized logging
- Prometheus monitoring
- Grafana dashboards
- Distributed tracing
- Automated deployment using CI/CD
- Docker-based deployment
- Kubernetes-based deployment

---

## Project Structure

```text
nginx-tomcat-load-balancer/
│
├── README.md
│
├── .gitignore
│
├── nginx/
│   └── nginx.conf
│
├── apache/
│   └── httpd.conf
│
├── tomcat/
│   ├── server1/
│   │   └── server-info/
│   │       └── index.jsp
│   │
│   └── server2/
│       └── server-info/
│           └── index.jsp
│
├── scripts/
│   ├── health-check.sh
│   └── load-test.sh
│
└── docs/
    ├── architecture.md
    └── testing.md
```

---

## Target Architecture

```text
                                  Client
                                    |
                                    | HTTP / HTTPS
                                    v
                         +----------------------+
                         |       Nginx          |
                         |    Load Balancer     |
                         |      :80 / :443      |
                         |                      |
                         |  Reverse Proxy       |
                         |  Round-Robin LB      |
                         +----------+-----------+
                                    |
                     +--------------+--------------+
                     |                             |
                     v                             v
          +---------------------+       +---------------------+
          |     Tomcat 1        |       |     Tomcat 2        |
          |   10.156.195.38     |       |   10.156.195.84     |
          |       :8080         |       |       :8080         |
          |                     |       |                     |
          | Application Server  |       | Application Server  |
          +---------------------+       +---------------------+
```

The project demonstrates how Nginx can be used as a reverse proxy and
load balancer in front of multiple Tomcat application servers, while
Apache HTTP Server is used to demonstrate additional web-server and
reverse-proxy functionality.
