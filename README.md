# Nginx-Tomcat Load Balancer

## Project Overview

This project implements a highly available web application hosting
architecture using Nginx, Apache HTTP Server and Apache Tomcat.

Nginx will act as a load balancer and distribute incoming requests
between two Tomcat application servers using the Round-Robin algorithm.

## Architecture

```text
                    Client
                       |
                       v
               +---------------+
               |    Nginx      |
               | Load Balancer |
               +-------+-------+
                       |
              +--------+--------+
              |                 |
              v                 v
       +-------------+   +-------------+
       |  Tomcat 1   |   |  Tomcat 2   |
       |  Server 1   |   |  Server 2   |
       +-------------+   +-------------+
