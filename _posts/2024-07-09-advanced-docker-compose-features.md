---
layout: post
title: Advanced Docker Compose Features
---

I recently set up a Prefect environment using Docker Compose and wanted to document some of the slightly more advanced features I used. This setup deploys a PostgreSQL database, Prefect server (API and front end), and two workers - one for running flows as subprocesses and another for running them as Docker containers.

While this configuration is specific to Prefect, the overall structure and many of the techniques are widely applicable. The pattern of bringing up a database alongside one or more application services is common in many Docker Compose stacks. As such, the advanced features explored here can be valuable in a variety of containerized deployments, from web applications to microservices architectures.

The full Docker Compose file, along with the rest of the code, is available in [this repo](https://github.com/EmilRex/prefect-docker-compose).

Here are the key Docker Compose features I found particularly useful:

### 1. YAML Anchors and Aliases

YAML anchors allow reusing common configuration across multiple services:

```yaml
x-prefect-common: &prefect-common
    build:
        context: .
    networks:
        - prefect
    restart: always

services:
    server:
        <<: *prefect-common
        command: prefect server start --host server --port 4200
        # Additional server-specific config
```

The `&prefect-common` anchor defines a set of common configurations. The `<<: *prefect-common` syntax then merges these common configs into each service that needs them. This keeps the compose file DRY and easier to maintain.

### 2. Health Checks

The compose file defines detailed health checks for services:

```yaml
healthcheck:
    test: ["CMD", "curl", "-f", "http://server:4200/api/health"]
    start_interval: 5s
    start_period: 15s
```
The `start_interval` specifies more frequent health checks during startup to make everything come up quicker, but only for the initial startup period. The `start_period` defines a grace period before starting these health checks. This allows for more nuanced control over service initialization, balancing quick startup with allowing sufficient time for services to initialize.

### 3. Startup Order

Health checks are used in combination with `depends_on` to ensure proper service startup order:

```yaml
docker-worker:
    <<: *prefect-common
    depends_on:
        server:
            condition: service_healthy
```

This configuration ensures that the `docker-worker` service only starts after the `server` service is not just running, but actually healthy according to its defined health check.

### 4. Docker-in-Docker

The Docker worker service includes a specific mount to enable interaction with the host's Docker daemon:

```yaml
volumes:
    - /var/run/docker.sock:/var/run/docker.sock
```

This mount allows the worker to call the Docker daemon on the host system, enabling it to spin up new containers for flow runs. This is crucial for the Docker worker's ability to create and manage containers for Prefect flows.
