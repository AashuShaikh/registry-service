# Registry Service

Service discovery server for the ChatApp microservices architecture. Every service registers here on startup, and other services use it to locate each other by name rather than hardcoded URLs.

---

## Overview

| | |
|---|---|
| **Port** | `8761` |
| **Context path** | `/` |
| **Framework** | Spring Boot 4.x + Spring Cloud Netflix Eureka Server |
| **Java** | 25 |

---

## Responsibilities

- Acts as the **Eureka Server** — the central registry for all microservices
- Maintains a real-time registry of service instances (name, host, port, health status)
- Enables **client-side load balancing** — when a service calls another by name, Eureka returns all healthy instances and the client picks one
- Provides a **web dashboard** at `http://localhost:8761` to monitor all registered services

---

## How It Works

```
Service starts up
    → registers with Eureka (name + host + port)
    → sends heartbeat every 30s to stay registered

Gateway / other services
    → ask Eureka: "where is the user service?"
    → Eureka returns: "192.168.1.x:8082"
    → service calls that address directly
```

This means no service needs to know another service's IP or port — only its application name.

---

## Registered Services

| Service Name | Port |
|---|---|
| `gateway` | 8080 |
| `auth` | 8081 |
| `user` | 8082 |
| `chat` | 8083 |
| `message` | 8084 |
| `notification` | 8085 |

---

## Configuration

```properties
# application.properties
server.port=8761

eureka.instance.hostname=localhost
eureka.client.register-with-eureka=false   # does not register itself
eureka.client.fetch-registry=false          # does not fetch its own registry
```

The `register-with-eureka=false` and `fetch-registry=false` settings prevent the registry from trying to register with itself.

---

## Running Locally

```bash
./gradlew bootRun
```

Once running, open the Eureka dashboard:

```
http://localhost:8761
```

You will see all registered service instances, their status (`UP` / `DOWN`), and metadata.

---

## Dependencies

| Dependency | Purpose |
|---|---|
| `spring-cloud-starter-netflix-eureka-server` | Eureka server implementation |

---

## Important Notes

- **Must be started first.** All other services register with Eureka on startup. If the registry is not running, services will fail to register and inter-service communication will break.
- In production, run **multiple Eureka instances** in a peer-to-peer cluster for high availability. Each instance registers with the others.
- The registry itself does **not** handle any business logic or authentication.
