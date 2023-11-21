# Rate limiter System Design

## 1. Functional requirements:

-   Limit the number of requests a client can make to an API within a specific time window.
-   Allow dynamic configuration of rate limits for different APIs or clients.
-   Design the rate limiter to be distributed, allowing scalability across multiple servers.

## 2. Non-functional Requirements:
-   The system should handle a large number of requests and scale horizontally as the load increases.
-   Requests should be processed with minimal latency, and rate limiting decisions should be made quickly.
-   The system should be resilient to failures, ensuring continued functionality even if some components fail.
