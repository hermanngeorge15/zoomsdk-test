
# README for API Rate Limiting Project Using Docker, NGINX, and Wiremock

## Overview
This project enhances an open-source initiative by addressing API rate limiting issues encountered with the Zoom APIs. We aim to implement a robust solution that helps developers adhere to daily request limits while avoiding HTTP 429 "Too Many Requests" errors.

## Project Components
- **Wiremock**: Mocks the API responses to simulate real API limits and behavior.
- **NGINX**: Acts as a proxy server to manage and enforce rate limits effectively.
- **Docker Compose**: Orchestrates the setup of NGINX and Wiremock in a controlled environment.

## Key Features
- **Rate Limit Simulation**: Simulates API rate limits (e.g., 4 requests per second up to 2000 requests per day) to understand and manage API usage effectively.
- **Custom NGINX Configuration**: Implements rate limiting using the leaky bucket algorithm to smooth out bursty traffic.
- **Flexible Testing Environment**: Allows for dynamic adjustments and testing of API responses and rate limits.

## Configuration
Below is a critical part of our `docker-compose.yaml` that outlines the setup for NGINX and Wiremock:

```yaml
version: '3.8'
services:
  nginx:
    container_name: nginx
    image: nginx:latest
    ports:
      - "8991:80"
    volumes:
      - ./nginx/conf/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/logs:/var/log/nginx
    depends_on:
      - wiremock

  wiremock:
    container_name: wiremock
    image: rodolpheche/wiremock
    ports:
      - "8992:8080"
    volumes:
      - ./wiremock:/home/wiremock

networks:
  app-network:
    driver: bridge
```

## NGINX Rate Limiting Setup
We configure NGINX for rate limiting with detailed setup instructions in the `nginx.conf`:

```nginx
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    log_format main '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$http_x_forwarded_for"';
    access_log /var/log/nginx/access.log main;

    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=4r/s;

    server {
        listen 80;
        location / {
            limit_req zone=mylimit burst=3 nodelay;
            error_page 503 =429 /custom_429.html;
            proxy_set_header Host $host;
            proxy_pass http://wiremock:8080;
        }
        location = /custom_429.html {
            internal;
            default_type text/html;
            return 429 'Too many requests. Please try again later.';
        }
    }
}
```

## Testing
Utilize tools like Apache Benchmark (`ab`) to test the rate limiting configuration:

```bash
ab -n 10 -c 5 http://<address_to_nginx>/v2/users
```

## Documentation and Further Reading
For a detailed description of rate limiting and how it is implemented, refer to the relevant [documentation here](https://zoom.us/developer).

## Conclusion
This README provides a comprehensive guide to setting up and managing rate limits for APIs using NGINX and Wiremock within a Docker environment. The configurations and tools outlined here will help ensure that API usage stays within the set limits, thereby avoiding service interruptions and enhancing overall system resilience.
