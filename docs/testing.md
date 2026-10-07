# Testing

## Overview

This document describes the testing approach for the Nginx-Tomcat load
balancer project.

Testing covers:

- Network connectivity
- Tomcat application availability
- Nginx reverse proxy functionality
- Load-balancing behavior
- Backend request distribution
- Performance testing
- Backend failure handling
- Recovery after backend restoration

---

# 1. Network Connectivity Testing

Before testing the application components, network connectivity between
the load balancer and backend servers must be verified.

## Test 1.1 - Nginx Server to Tomcat Server 1

**Source:**

```text
Machine 1 - 10.156.195.129
```

**Destination:**

```text
Machine 2 - 10.156.195.38
```

Command:

```bash
ping -c 4 10.156.195.38
```

Expected result:

```text
4 packets transmitted
4 packets received
0% packet loss
```

---

## Test 1.2 - Nginx Server to Tomcat Server 2

**Source:**

```text
Machine 1 - 10.156.195.129
```

**Destination:**

```text
Machine 3 - 10.156.195.84
```

Command:

```bash
ping -c 4 10.156.195.84
```

Expected result:

```text
4 packets transmitted
4 packets received
0% packet loss
```

---

# 2. Tomcat Server Testing

Each Tomcat server is tested independently before introducing Nginx
into the request path.

Tomcat Server 1 listens on port `8080`, while Tomcat Server 2 listens
on port `8081`.

---

## Test 2.1 - Tomcat Service Status

Tomcat must be running as a systemd service.

Command:

```bash
sudo systemctl status tomcat
```

Expected result:

```text
Active: active (running)
```

---

## Test 2.2 - Tomcat Port Verification

Tomcat must listen on its configured application port.

For Tomcat Server 1:

```bash
sudo ss -ltnp | grep ':8080'
```

Expected:

```text
LISTEN ... *:8080 ...
```

For Tomcat Server 2:

```bash
sudo ss -ltnp | grep ':8081'
```

Expected:

```text
LISTEN ... *:8081 ...
```

---

## Test 2.3 - Tomcat Local Connectivity

The Tomcat application is tested locally on each backend server.

### Tomcat Server 1

```bash
curl -I http://localhost:8080
```

Expected response:

```text
HTTP/1.1 200
Content-Type: text/html;charset=UTF-8
```

### Tomcat Server 2

```bash
curl -I http://localhost:8081
```

Expected response:

```text
HTTP/1.1 200
Content-Type: text/html;charset=UTF-8
```

---

# 3. Tomcat Server 1 Testing

## Server Details

| Parameter | Value |
|---|---|
| Hostname | `APMOSYSLT0953` |
| IP Address | `10.156.195.38` |
| Java Version | OpenJDK 17.0.20.1 |
| Tomcat Version | 10.1.60 |
| Port | 8080 |
| Service Manager | systemd |

---

## Test 3.1 - Tomcat Server 1 Service

Command:

```bash
sudo systemctl status tomcat
```

Expected result:

```text
Active: active (running)
```

---

## Test 3.2 - Tomcat Server 1 Port

Command:

```bash
sudo ss -ltnp | grep ':8080'
```

Expected result:

```text
LISTEN ... *:8080 ...
```

---

## Test 3.3 - Tomcat Server 1 Local Access

Command:

```bash
curl http://localhost:8080/server-info/
```

Expected application response:

```text
Tomcat Application Server 1
Hostname: APMOSYSLT0953
Server IP: 10.156.195.38
Tomcat Port: 8080
Server Role: Backend Application Server 1
```

---

## Test 3.4 - Tomcat Server 1 Remote Access

The application must also be accessible from the Nginx load-balancer
server.

**Source:**

```text
Machine 1 - 10.156.195.129
```

**Destination:**

```text
Machine 2 - 10.156.195.38:8080
```

Command:

```bash
curl http://10.156.195.38:8080/server-info/
```

Expected response:

```text
Tomcat Application Server 1
Hostname: APMOSYSLT0953
Server IP: 10.156.195.38
Tomcat Port: 8080
Server Role: Backend Application Server 1
```

This verifies that the Tomcat Server 1 application is accessible
remotely over the network.

---

# 4. Tomcat Server 2 Testing

## Server Details

| Parameter | Value |
|---|---|
| Hostname | `shashwat-mishra-K55VJ` |
| IP Address | `10.156.195.84` |
| Java Version | OpenJDK 17.0.20.1 |
| Tomcat Version | 10.1.60 |
| Port | 8081 |
| Service Manager | systemd |

Tomcat Server 2 uses port `8081` because port `8080` is already
occupied by an existing Docker/KIND service on the host.

---

## Test 4.1 - Tomcat Server 2 Service

Command:

```bash
sudo systemctl status tomcat
```

Expected result:

```text
Active: active (running)
```

---

## Test 4.2 - Tomcat Server 2 Port

Command:

```bash
sudo ss -ltnp | grep ':8081'
```

Expected result:

```text
LISTEN ... *:8081 ...
```

---

## Test 4.3 - Tomcat Server 2 Local Access

Command:

```bash
curl http://localhost:8081/server-info/
```

Expected response:

```text
Tomcat Application Server 2
Hostname: shashwat-mishra-K55VJ
Server IP: 10.156.195.84
Tomcat Port: 8081
Server Role: Backend Application Server 2
```

---

## Test 4.4 - Tomcat Server 2 Remote Access

The application must also be accessible from the Nginx load-balancer
server.

**Source:**

```text
Machine 1 - 10.156.195.129
```

**Destination:**

```text
Machine 3 - 10.156.195.84:8081
```

Command:

```bash
curl http://10.156.195.84:8081/server-info/
```

Expected response:

```text
Tomcat Application Server 2
Hostname: shashwat-mishra-K55VJ
Server IP: 10.156.195.84
Tomcat Port: 8081
Server Role: Backend Application Server 2
```

This verifies that the Tomcat Server 2 application is accessible
remotely over the network.

---

# 5. Nginx Testing

After both Tomcat servers are independently verified, Nginx is tested
as the frontend reverse proxy.

## Test 5.1 - Nginx Service

Command:

```bash
sudo systemctl status nginx
```

Expected result:

```text
Active: active (running)
```

---

## Test 5.2 - Nginx Configuration

Before restarting or reloading Nginx, validate the configuration:

```bash
sudo nginx -t
```

Expected result:

```text
syntax is ok
test is successful
```

---

## Test 5.3 - Nginx Port

Nginx should listen on HTTP port `80`.

Command:

```bash
sudo ss -ltnp | grep ':80'
```

Expected result:

```text
LISTEN ... *:80 ...
```

---

# 6. Nginx Reverse Proxy Testing

The Nginx server must successfully forward client requests to the
Tomcat backend.

Command:

```bash
curl http://10.156.195.129/
```

Expected result:

```text
HTTP response from the Tomcat application
```

The response should originate from one of the configured Tomcat
servers.

---

# 7. Round-Robin Load Balancing Testing

The objective of this test is to verify that Nginx distributes
requests between both Tomcat servers.

The server-information applications make it possible to identify which
backend processed each request.

Example request:

```bash
curl http://10.156.195.129/server-info/
```

Repeat the request multiple times.

Example:

```bash
for i in {1..10}; do
    curl -s http://10.156.195.129/server-info/ | grep "Tomcat Application"
done
```

Expected behavior:

```text
Tomcat Application Server 1
Tomcat Application Server 2
Tomcat Application Server 1
Tomcat Application Server 2
...
```

The exact distribution should be measured during testing rather than
assuming a perfectly equal split.

---

# 8. Load Testing

Load testing is performed against the Nginx load-balancer endpoint.

The backend Tomcat servers should not be targeted directly when
validating the load-balancing behavior.

Test endpoint:

```text
http://10.156.195.129/
```

---

## 8.1 ApacheBench

A basic concurrency test can be executed using:

```bash
ab -n 1000 -c 50 http://10.156.195.129/
```

Where:

| Parameter | Description |
|---|---|
| `-n` | Total number of requests |
| `-c` | Number of concurrent requests |

Important metrics:

- Requests per second
- Time per request
- Failed requests
- Transfer rate
- Connection times
- Response time

---

## 8.2 JMeter

JMeter can be used for more advanced load-testing scenarios.

The test flow should be:

```text
JMeter
   |
   v
Nginx :80
   |
   +------------------+
   |                  |
   v                  v
Tomcat 1           Tomcat 2
:8080              :8081
```

JMeter can be used to evaluate:

- Concurrent users
- Throughput
- Response time
- Percentile response time
- Error rate
- Backend request distribution
- Sustained load behavior

---

# 9. Backend Failure Testing

Backend failure testing verifies the behavior of Nginx when one
application server becomes unavailable.

---

## Test 9.1 - Tomcat Server 1 Failure

Stop Tomcat Server 1:

```bash
sudo systemctl stop tomcat
```

Verify the service:

```bash
sudo systemctl status tomcat
```

Generate requests through Nginx:

```bash
curl http://10.156.195.129/server-info/
```

Multiple requests should continue to be processed by the available
Tomcat Server 2.

---

## Test 9.2 - Restore Tomcat Server 1

Start Tomcat Server 1:

```bash
sudo systemctl start tomcat
```

Verify:

```bash
sudo systemctl status tomcat
```

Then test:

```bash
curl http://10.156.195.129/server-info/
```

Tomcat Server 1 should become available again after Nginx considers
the backend healthy.

---

## Test 9.3 - Tomcat Server 2 Failure

Stop Tomcat Server 2:

```bash
sudo systemctl stop tomcat
```

Generate requests through Nginx:

```bash
curl http://10.156.195.129/server-info/
```

Requests should continue through Tomcat Server 1.

Restore Tomcat Server 2:

```bash
sudo systemctl start tomcat
```

---

# 10. Performance Metrics

The following metrics should be captured during load testing.

| Metric | Description |
|---|---|
| Throughput | Requests processed per second |
| Average Response Time | Average request processing time |
| Minimum Response Time | Fastest response |
| Maximum Response Time | Slowest response |
| Percentile Response Time | Response-time distribution |
| Error Rate | Percentage of failed requests |
| Concurrent Users | Number of simultaneous users |
| Backend Distribution | Requests processed by each backend |
| CPU Utilization | CPU consumption |
| Memory Utilization | Memory consumption |
| Network Utilization | Network traffic |

---

# 11. System Monitoring

During performance testing, system resources can be monitored using
standard Linux utilities.

## CPU and Memory

```bash
top
```

```bash
free -h
```

## Disk

```bash
df -h
```

## Network

```bash
ss -tulnp
```

## Processes

```bash
ps aux
```

---

# 12. Nginx Log Analysis

Nginx access logs can be monitored using:

```bash
sudo tail -f /var/log/nginx/access.log
```

Error logs:

```bash
sudo tail -f /var/log/nginx/error.log
```

These logs can be used to investigate:

- Request failures
- HTTP status codes
- Backend connection failures
- Request volume
- Client connections
- Proxy errors

---

# 13. Tomcat Log Analysis

Tomcat logs can be monitored using:

```bash
sudo tail -f /opt/tomcat/logs/catalina.out
```

Tomcat logs can be used to investigate:

- Application errors
- Startup issues
- Shutdown events
- Request processing issues
- JVM-related problems

---

# 14. Test Summary

The complete testing flow is:

```text
Network Connectivity
        |
        v
Tomcat Server 1
        |
        v
Tomcat Server 2
        |
        v
Nginx Reverse Proxy
        |
        v
Round-Robin Load Balancing
        |
        v
Performance Testing
        |
        v
Backend Failure Testing
        |
        v
Recovery Testing
```

The final validation should demonstrate that:

1. Both Tomcat servers are independently accessible.
2. Nginx can communicate with both backend servers.
3. Nginx successfully proxies requests to Tomcat.
4. Requests are distributed across both Tomcat servers.
5. The system can handle concurrent requests.
6. Backend failure behavior is correctly handled.
7. Backend recovery restores normal request distribution.
8. Performance and system-resource metrics can be collected.
