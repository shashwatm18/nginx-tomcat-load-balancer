# Infrastructure Architecture

## Overview

This project implements a three-machine application hosting and load
balancing environment.

Nginx will act as the frontend load balancer and distribute incoming
HTTP requests between two Apache Tomcat application servers using the
Round-Robin algorithm.

Apache HTTP Server will also be configured and tested as a web server
and reverse proxy.

## Infrastructure

| Machine | Hostname | IP Address | Role |
|---|---|---|---|
| Machine 1 | APMOSYSLT0363 | 10.156.195.129 | Nginx Load Balancer |
| Machine 2 | APMOSYSLT0953 | 10.156.195.38 | Tomcat Application Server 1 |
| Machine 3 | shashwat-mishra-K55VJ | 10.156.195.84 | Tomcat Application Server 2 |

## Network

All three machines are connected to the same IPv4 subnet:

    10.156.195.0/24

Network connectivity from Machine 1 to both backend servers has been
verified successfully.

### Connectivity Test

Machine 1 -> Machine 2

    ping 10.156.195.38

Result:

    4 packets transmitted
    4 packets received
    0% packet loss

Machine 1 -> Machine 3

    ping 10.156.195.84

Result:

    4 packets transmitted
    4 packets received
    0% packet loss

## Planned Architecture

```text
                         Client
                           |
                           | HTTP :80
                           |
                           v
                  +-------------------+
                  |     Machine 1     |
                  |   10.156.195.129  |
                  |                   |
                  |      Nginx        |
                  |  Load Balancer    |
                  +---------+---------+
                            |
                  Round-Robin Distribution
                     /              \
                    /                \
                   v                  v
        +------------------+  +------------------+
        |    Machine 2     |  |    Machine 3     |
        | 10.156.195.38    |  | 10.156.195.84    |
        |                  |  |                  |
        |     Tomcat 1     |  |     Tomcat 2     |
        |     :8080        |  |     :8080        |
        +------------------+  +------------------+
