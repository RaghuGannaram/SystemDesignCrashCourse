# URL Shortening Service System Design

## 1. Back of the envelope estimations:

-   **Assumptions**:

        -   Number of URLs generated per day:       100 million
        -   Read/Write Ratio:                       100:1
        -   Average length of long URL:             2000 characters
        -   Data Retention Period:                  10 years

-   **Calculations**:

        -   Number of URLs generated per second: 100M:                  100 million / (24 x 60 x 60)   ~= 1,160/second.
        -   Number of URLs gnerated in the span of 10 years:            100 million x 365days x 10years ~= 365 billion
        -   Number of URLs shortened (Write Operations) per second:     1,160/second.
        -   Number of URLs redirected (Read Operations) per second:     1,160 x 100 = 116,000/second.
        -   Storage per URL per second (Using ASCII encoding):          2000B/URL.
        -   Total Storage requirement:                                  (100 million x 365days x 10years)url x (2000B/long_url + 7B/short_url) ~= 732TB
        -   Transactions Per Second (TPS):                              1,160/second.
        -   Queries Per Second (QPS):                                   1,160 + 116,000 = 117,160/second.
        -   Maximum Required Cache (MRC):                               (20/100) x 116,000 x (2000B + 7B) x 86,400sec = 46.6MB/sec x 86,400sec ~= 4TB

## 2. User Interface:

### API Endpoints:

-   **Shorten URL:**

    -   Endpoint: `/generate-hash`
    -   Method: `POST`
    -   Description: Shortens a long URL.
    -   Request:

        ```http
        POST /generate-hash HTTP/1.1
        Host: short.ly
        Content-Type: application/json

        {
          "url": "https://www.example.com/long-url-to-be-shortened"
        }
        ```

    -   Response:

        ```http
        HTTP/1.1 200 OK
        Content-Type: application/json

        {
          "short_url": "https://short.ly/zxD32f"
        }
        ```

-   **Retrieve Original URL:**

    -   Endpoint: `/{short_code}`
    -   Method: `GET`
    -   Description: Retrieves the original URL associated with the short code.
    -   Request:
        ```http
        GET /zxD32f HTTP/1.1
        Host: short.ly
        ```
    -   Response:

        ```http
        HTTP/1.1 302 Found
        Location: https://www.example.com/long-url-to-be-shortened
        ```

### UI Components:

-   **Web Interface:**

    -   Provide a web-based interface for users to manually shorten or expand URLs.
    -   Include input fields for entering long URLs and displaying the shortened URLs.

-   **Browser Extensions:**

    -   Consider developing browser extensions for popular browsers to allow users to shorten URLs directly from their browsers.

-   **Mobile App:**

    -   Develop a mobile application to offer URL shortening and redirection features on mobile devices.
    -   Include a user-friendly interface optimized for mobile screens.

-   **QR Code Generation:**

    -   Implement functionality to generate QR codes for shortened URLs, facilitating easy access for mobile users.

### UI Considerations:

-   **User Authentication:**

    -   Implement user accounts to provide personalized features and analytics.
    -   Allow users to manage their shortened URLs.

-   **Link Management Dashboard:**

    -   Develop a dashboard where users can view and manage their shortened URLs.
    -   Include analytics such as click counts and geographic information.

-   **URL Preview:**

    -   Offer a preview feature that allows users to preview the original URL before being redirected.

-   **Custom Short URLs:**

    -   Optionally, provide users with the ability to customize their short URLs.

## 3. Application Layer:

### URL Shortening Service:

-   **Shortening Algorithm:**

    Utilize a unique ID generator to create a unique identifier for each long URL. This unique identifier can serve as the short code.

    -   Commonly used unique ID generation techniques include:

        -   **UUID (Universally Unique Identifier):**
            -   Generate a UUID (e.g., UUID4) for each long URL. UUIDs are designed to be globally unique.
        -   **Hashing Algorithms:**
            -   Apply a hash function (e.g., SHA-256) to the long URL to produce a fixed-length hash, which can be used as the short code.

    -   **Distributed Systems and Unique ID Generation:**
        -   In a distributed system, ensuring the uniqueness of generated IDs across multiple nodes is crucial.
        -   **Zookeeper for Sequential Znodes:**
            -   ZooKeeper can be used to implement a distributed unique ID generation system. Each node can create sequential znodes, and the sequence number appended to the znode can serve as a unique ID.
            -   This ensures a monotonically increasing series of IDs across the distributed nodes.

-   **Redirection:**

    -   When a short URL is accessed, redirect the user to the original URL.
    -   Ensure the provided URL is valid.
    -   Redirect to default 404 page for invalid short URLs.

### Additional Services Considerations:

-   **Rate Limiting:**

    -   Implement rate limiting to prevent abuse and ensure fair usage.

-   **User Authentication and Authorization:**

    -   Develop a secure authentication system for user accounts.
    -   Enforce proper authorization to ensure that users can only modify or view their own URLs.

-   **URL Analytics and Tracking:**

    -   Implement a component for tracking user interactions with short URLs.
    -   Collect data on the number of clicks, geographic information, and user agents.

-   **URL Expiration and Archiving:**

    -   Implement a mechanism to set expiration dates for short URLs.
    -   Archive or delete expired URLs to manage the database efficiently.

### Application Layer Considerations:

-   **Microservices Architecture:**

    -   Explore the possibility of adopting a microservices architecture for scalability and maintainability.
        1.  Unique ID Generation Service
        2.  Analytics Service
        3.  Authentication Service
        4.  URL Shortening and Redirection Service

-   **Caching Mechanism:**

    -   Implement caching for the URL Shortening and Redirection Service to store recently accessed short URLs.
    -   Use a caching system like **Redis** or **Memcached**.
    -   **Cache System Configuration Recommendations**

            -   Cache Hit Rate:             Cache hit rate is commonly around 80%, so let's estimate it as 80%.
            -   Cache Miss Rate:            Cache miss rate = 100% - Cache hit rate = 20%.
            -   Eviction Policy:            Use a Least Recently Used (LRU) eviction policy for simplicity.
            -   Expiration Time:            Set an expiration time of 5 minutes for cached entries.
            -   Cache Compression Ratio:    Assume a conservative compression ratio of 2:1 for cached data.
            -   Cache Warm-Up Strategy:     Implement a cache warm-up strategy by prepopulating frequently accessed URLs during system initialization.
            -   Cache Persistence:          Implement a hybrid approach with partial data persistence to disk and in-memory caching for frequently accessed entries.
            -   Concurrency and Locking:    Implement a concurrency control mechanism, such as fine-grained locking, to handle concurrent access.
            -   Monitoring and Metrics:     Implement monitoring tools to track cache hit rates, miss rates, and resource usage.
            -   Scalability:                Choose a cache system that supports horizontal scaling to accommodate increased loads.
            -   Fault Tolerance:            Implement redundant cache instances and a failover mechanism to handle cache failures.
            -   Cost Analysis:              Evaluate memory costs based on the cache size needed to accommodate frequently accessed data.
            -   Testing and Optimization:   Conduct thorough testing to optimize cache configurations for optimal performance.

-   **Asynchronous Processing:**

    -   Consider implementing asynchronous processing for non-time-sensitive tasks to enhance system responsiveness.
    -   Implement asynchronous processing for tasks like analytics updates, sending notifications, or batch processing of data.
    -   Use message queues (e.g., **RabbitMQ**, **Apache Kafka**) to manage asynchronous task execution.

-   **Scalability Strategies:**

    -   Achieve scalability using strategies such as horizontal scaling, load balancing, and auto-scaling based on demand.

## 4. Data Storage:

### Database Selection:

Given the characteristics and requirements of the URL shortening service, a NoSQL database is the preferred choice. In particular, MongoDB is well-suited for its flexibility, scalability, and efficient handling of read and write operations.

### Justification:

1. **Schema Flexibility:**

    - MongoDB's document-oriented structure provides schema flexibility, allowing easy adaptation to changes in the data structure over time without requiring a predefined schema.

2. **High Write and Read Throughput:**

    - MongoDB is known for its ability to handle high write and read throughput, making it suitable for the URL shortening service's expected traffic.

3. **Scalability:**

    - MongoDB's horizontal scaling capabilities align with the service's need for scalability as it grows in terms of users and data.

4. **Quick Access to Data:**
    - MongoDB's document model and indexing features enable quick access to data, supporting the service's requirement for efficient retrieval of short URLs.

### Database Schema (MongoDB):

#### Collections:

-   **ShortenedURLs:**

        -   `_id`:              (Primary Key): The unique identifier for each record in the collection.
        -   `short_code`:       The generated short code associated with the original URL.
        -   `original_url`:     The long URL that is shortened.
        -   `creation_date`:    The timestamp indicating when the short URL was created.
        -   `expiration_date`:  (optional): An optional field representing the date when the short URL expires, if applicable.

#### Indexing:

-   Index on `short_code`: Enhances the retrieval performance of short URLs.

#### Sharding:

-   Shard the `ShortenedURLs` collection based on `short_code` to distribute data and facilitate horizontal scaling.

#### Considerations:

1. **Data Partitioning:**

    - Implement a sharding strategy for efficient data distribution and retrieval.

2. **Secondary Indexing:**

    - Consider additional secondary indexes based on query patterns, e.g., an index on `creation_date` for chronological queries.

3. **Data Compression:**

    - Evaluate and implement data compression techniques to optimize storage usage.

4. **Backup and Recovery:**

    - Establish a robust backup and recovery mechanism to ensure data durability and availability.

5. **Security Measures:**

    - Implement authentication and authorization mechanisms to secure the database access.

6. **Monitoring and Alerts:**

    - Set up monitoring tools to track database performance and configure alerts for potential issues.

7. **Schema Evolution:**

    - Plan for schema evolution to accommodate future changes in data structure.

8. **Data Encryption:**

    - Consider encrypting sensitive fields or the entire database for enhanced security.

9. **Optimized Queries:**

    - Optimize queries based on common use cases to improve overall system responsiveness.

10. **Database Versioning:**
    - Stay updated with the latest database version and apply patches for security and performance improvements.

## 5. Infrastructure:

### Components:

-   **Web Server:**

    -   Utilizes a lightweight HTTP server like **Nginx** or **Apache** to handle incoming HTTP requests.
    -   Forwards requests to the URL Shortening Service using HTTP/HTTPS protocols.
    -   Deployed on scalable virtual machines or containers.

-   **URL Shortening Service:**

    -   Developed using a programming language such as **Node.js**, **Python** (using frameworks like Flask or Django), or **Java** (using Spring Boot).
    -   Implements shortening and expanding logic, generating unique short codes.
    -   Communicates with the MongoDB database for storing and retrieving URL mappings.

-   **Database (MongoDB):**

    -   Utilizes **MongoDB** as the chosen NoSQL database for its flexibility, scalability, and efficient handling of high-throughput read and write operations.
    -   Configured for horizontal scaling and sharding to distribute the workload across multiple nodes.
    -   MongoDB drivers, such as **Mongoose** for Node.js or **PyMongo** for Python, are used for seamless integration.

-   **Cache (Optional):**

    -   Implements caching using tools like **Redis** or **Memcached** to store frequently accessed short codes.
    -   Configuration for cache eviction policies and expiration to manage memory usage effectively.
    -   Optionally, use caching libraries like **redis-py** for Python or **node-cache** for Node.js.

-   **Load Balancer:**
    -   Utilizes a robust load balancing solution such as **HAProxy**, **Nginx Load Balancer**, or cloud-based load balancers (e.g., **AWS ELB**, **Google Cloud Load Balancer**).
    -   Ensures even distribution of incoming requests across multiple Web Server instances.
    -   Configured with health checks and auto-scaling policies for dynamic resource allocation.

### Considerations:

-   **Scalability:**

    -   Utilizes container orchestration tools like **Docker** and **Kubernetes** for easy scaling of Web Server and URL Shortening Service instances.
    -   Cloud providers like **AWS**, **Google Cloud**, or **Azure** for on-demand resource scaling.

-   **Fault Tolerance:**

    -   Implements redundant Web Server instances and database replication for fault tolerance.
    -   Regularly backs up MongoDB data using tools like **mongodump** for data recovery.

-   **Optional Cache:**
    -   Monitors cache hit rates and adjusts cache size dynamically using tools like **redis-cli** or **memcached-tool**.
    -   Configures cache persistence for essential data to prevent cache warm-up delays.
