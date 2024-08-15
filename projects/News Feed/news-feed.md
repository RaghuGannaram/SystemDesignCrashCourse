# News Feed System Design

## System Requirements

### Functional Requirements

-   **Feed Publishing:**
    -   Users can create posts with text, images, and videos.
-   **Feed Retrieval:**
    -   Users can retrieve their news feed in reverse chronological order.
    -   Feeds can include posts from friends and groups.
-   **Notifications:**
    -   Users receive notifications when friends post new content.
    -   Real-time updates for new likes, comments, and shares.

### Non-Functional Requirements

-   **Scalability:**
    -   Must handle 10 million Daily Active Users (DAUs) and 5000 friends per user.
-   **Performance:**
    -   Feed retrieval should have a response time under 200ms.
-   **Availability:**
    -   Aim for 99.99% uptime with minimal downtime.
-   **Consistency:**
    -   Ensure eventual consistency for updates across the system.
-   **Extensibility:**
    -   Support easy addition of new features.

## Back of the Envelope Calculations

    Assunptions:

    -   Daily Active Users (DAUs):                100 million        # Facebook has 1.8 billion DAUs
    -   Average user follows:                     500                # Facebook has 338 friends per user
    -   Average post size:                        10 KB              # Text (2KB) + Image (8KB)
    -   Average No. of posts per user per day:    0.05

    Calculations:

    -   No.of daily posts:                        100 million users * 0.05 posts/user = 5 million posts
    -   Total writes per day:                     5 million writes
    -   Transactions per second:(TPS)             5 million writes / 86400 seconds = ~58 writes per second

    -   Total data generated per day:             5 million posts * 10 KB /post = 50 GB
    -   Storage for 10 years:                     50 GB * 365 days * 10 years = 182.5 TB

    -   No of posts in a typical feed view:       50
    -   Total reads per day:                      100 million users * 50 posts = 5 billion reads
    -   Queries per second (QPS):                 5 billion reads / 86400 seconds = ~58K reads per second

    -   Data read per feed view:                  50 posts * 10 KB = 500 KB
    -   Total data read for 1 day:                5 billion reads * 500 KB = 2.5 PB

    -   Network bandwidth for writes:             58 writes per second * 10 KB = 580 KB/s
    -   Network bandwidth for reads:              58K reads per second * 10 KB = 580 MB/s
    -   Total network bandwidth:                  580 KB/s + 580 MB/s = 580.58 MB/s

## API Design

We can have the following methods for our News Feed service:

1.  **Feed Publishing:** : `createPost(UserID, Post post) -> Post`

    -   Creates a new post for the user.
    -   Takes the user ID and the post object as input and returns the created post.
    -   Returns the created post object.
    -   Request

    ```http
    POST /createPost HTTP/1.1
    Host: newsfeed.com
    Content-Type: application/json

    {
        "UserID": "123",
        "Post": {
            "PostID": "456",
            "Text": "Hello World!",
            "Image": "image1.jpg",
            "Video": "video1.mp4"
        }
    }
    ```

    -   Response

    ```http
    HTTP/1.1 201 Created
    Content-Type: application/json

    {
        "PostID": "456",
        "Text": "Hello World!",
        "Image": "image1.jpg",
        "Video": "video1.mp4"
    }
    ```

2.  **Feed Retrieval:** `getNewsFeed(UserID) -> List<Post>`

    -   Retrieves the news feed for the user.
    -   Takes the user ID as input and returns the list of posts in the user's feed.
    -   Returns a list of post objects.
    -   Request:

    ```http
    GET /getNewsFeed?UserID=123 HTTP/1.1
    Host: newsfeed.com
    ```

    -   Response:

    ```http
    HTTP/1.1 200 OK
    Content-Type: application/json

    {
        "Posts": [
            {
                "PostID": "456",
                "Text": "Hello World!",
                "Image": "image1.jpg",
                "Video": "video1.mp4"
            },
            {
                "PostID": "457",
                "Text": "Feeling Awesome!",
                "Image": "image2.jpg",
                "Video": "video2.mp4"
            }
        ]
    }
    ```

## Database Design

-   We need to store data about `users`, `posts`, and their `relationships`. We also need to store the `news feed` data for each user.

### User Data

-   We need to store data about each user, such as `UserID`, `Name`, `Email`, `DoB`, `authentication credentials` and other user-specific information.
-   Database selection:

    -   We can go for Relational database as the schema is not going to change frequently.
    -   Relational databases provide ACID properties and are suitable for complex queries.
    -   `PostgreSQL` can be a good choice as it supports advanced data types and indexing and complex queries.

-   User Schema:

    ```sql

    | Column Name   | Data Type    | Constraints             | Description                                              |
    | ------------- | ------------ | ----------------------- | -------------------------------------------------------- |
    | user_id       | UUID         | PRIMARY KEY, DEFAULT    | Unique identifier for each user, generated automatically |
    | username      | VARCHAR(50)  | NOT NULL, UNIQUE        | Unique username for each user                            |
    | email         | VARCHAR(100) | NOT NULL, UNIQUE        | Unique email address for each user                       |
    | password_hash | VARCHAR(255) | NOT NULL                | Securely stored hashed password                          |
    | created_at    | TIMESTAMP    | NOT NULL, DEFAULT NOW() | Timestamp when the user account was created              |
    | updated_at    | TIMESTAMP    | NOT NULL, DEFAULT NOW() | Timestamp of the last profile update                     |

    ```

### Post Data

-   We need to store data about each post, such as `PostID`, `UserID`, `Text`, `Image`, `Video`, `Likes`, `Comments`, `Shares`, and other post-specific information.
-   Database selection:

    -   We can go for a NoSQL database as the schema is flexible and the data is not relational.
    -   `NoSQL document stores` like `MongoDB` or `Cassandra` are suitable for handling large-scale, unstructured data with horizontal scaling capabilities.
    -   These databases offer high write throughput and flexible schema designs, which are essential for storing large volumes of user-generated content efficiently.
    -   `Cassandra` can be a good choice as it provides high write throughput and linear scalability.

-   Post Data Model:

    ```sql

    | Column Name    | Data Type | Description                                         | Primary Key                      |
    | -------------- | --------- | --------------------------------------------------- | -------------------------------- |
    | user_id        | UUID      | Unique identifier for the user who created the post | Partition Key                    |
    | created_at     | TIMESTAMP | Timestamp when the post was created                 | Clustering Column (Ordered DESC) |
    | post_id        | UUID      | Unique identifier for each post                     | Clustering Column                |
    | content        | TEXT      | The textual content of the post                     | -                                |
    | media_url      | TEXT      | URL to any media associated with the post           | -                                |
    | likes_count    | COUNTER   | Counter for the number of likes                     | -                                |
    | comments_count | COUNTER   | Counter for the number of comments                  | -                                |
    | shares_count   | COUNTER   | Counter for the number of shares                    | -                                |

    ```

### Relationship Data

-   We need to store data about the relationships between users, such as `FollowerID`, `FolloweeID`, and other relationship-specific information.
-   Database selection:

    -   Social relationships, such as friendships and follower-following relationships, naturally form a graph structure where nodes represent users and edges represent relationships.
    -   Graph databases like `Neo4j` are well-suited for storing and querying graph data structures.
    -   They allow for quick traversal of relationships, making operations like finding mutual friends, suggesting new friends, and querying follower networks much faster and more efficient compared to traditional SQL or NoSQL databases.

    -   User Node:

        ```sql
        | Property | Type   | Description                     |
        | -------- | ------ | ------------------------------- |
        | user_id  | UUID   | Unique identifier for each user |
        | username | String | The username of the user        |

        ```

    -   Relationship Edge:

        ```sql
        | Type         | Description                                         |
        | ------------ | --------------------------------------------------- |
        | FRIENDS_WITH | Represents a bidirectional friendship between users |
        | Properties   | None                                                |
        ```

### News Feed Data

-   We need to store the news feed data for each `user`, which includes `posts` from the user's friends and groups.
-   Database selection:

    -   News feed data requires fast read and write operations to ensure that users can view their feed in real-time and that updates (new posts) are reflected quickly.
    -   By combining `Redis` for caching and `Cassandra` for long-term storage, we can achieve a balance between speed and durability in managing the news feed.
    -   `Redis` is an in-memory key-value store that excels at low-latency data retrieval, making it ideal for caching frequently accessed news feed data.
    -   For persistence, `Cassandra` can be used to store the actual feed items, leveraging its ability to handle high write volumes and large datasets with eventual consistency.

-   Persistent Storage (Cassandra):

    ```sql

    | Column Name | Data Type | Description                                         | Primary Key                      |
    | ----------- | --------- | --------------------------------------------------- | -------------------------------- |
    | user_id     | UUID      | Unique identifier for the user whose feed is stored | Partition Key                    |
    | post_id     | UUID      | Unique identifier for each post                     | Clustering Column                |
    | created_at  | TIMESTAMP | Timestamp when the post was created                 | Clustering Column (Ordered DESC) |

    ```

-   Cache (Redis):

    -   We can store the news feed data in Redis using a `sorted set` where the score is the timestamp of the post and the value is the post ID.
    -   This allows us to retrieve the latest posts quickly and efficiently.

    ```sql

    | Key Format     | Value            | Description                                                                                     |
    | -------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
    | feed:{user_id} | List of post_ids | A list of post_ids representing the posts in a user's feed, ordered by time (most recent first) |

    ```

## System Design

### Post Service

-   Responsibilities:

    -   Handle the creation, editing, and deletion of posts.
    -   Persist post data in `Cassandra`.
    -   Broadcast new posts to followers' feeds.

-   Technology Stack:

    -   `Java (Spring Boot)` for its enterprise-grade features and robustness.

-   Communication:

    -   Communicates with the Feed Service via asynchronous messaging (e.g., `RabbitMQ`) to update feeds.
    -   Exposes RESTful APIs for the frontend to create and manage posts.

-   Example Code (Java with Spring Boot):

    ```java
    @RestController
    @RequestMapping("/posts")
    public class PostController {

        @Autowired
        private PostService postService;

        @PostMapping("/create")
        public ResponseEntity<Post> createPost(@RequestBody PostRequest postRequest) {
            Post createdPost = postService.createPost(postRequest);
            // Publish event to update followers' feeds
            eventPublisher.publishEvent(new PostCreatedEvent(this, createdPost));
            return new ResponseEntity<>(createdPost, HttpStatus.CREATED);
        }
    }
    ```

### Feed Service

-   Responsibilities:

    -   Retrieve the news feed for users.
    -   Aggregate posts from friends and groups.
    -   Integrate with caching layers (`Redis`) and storage (`Cassandra`).

-   Technology Stack:

    -   `Go` or `Node.js` for high performance and handling a large number of concurrent connections.

-   Communication:

    -   Communicates with `Redis` for caching and `Cassandra` for persistent storage.
    -   Exposes RESTful APIs for the frontend to fetch user feeds.

-   Example Code (Go):

    ```go
    // FetchUserFeed retrieves the news feed for a given user
    func FetchUserFeed(userID string) ([]Post, error) {
        // Try to get the feed from Redis cache
        feed, err := redisClient.Get(fmt.Sprintf("feed:%s", userID)).Result()
        if err == redis.Nil {
            // Cache miss, fetch from Cassandra
            feed, err = fetchFeedFromCassandra(userID)
            if err != nil {
                return nil, err
            }
            // Cache the result
            redisClient.Set(fmt.Sprintf("feed:%s", userID), feed, time.Hour)
        }
        return parseFeed(feed), nil
    }

    ```

### User Service

-   Responsibilities:

    -   Manage user profiles, authentication, and authorization.
    -   Handle user-related data, such as relationships (friends).
    -   Integrate with the graph database (`Neo4j`) for managing social relationships.

-   Technology Stack:

    -   `Node.js` with `Express.js` or `Python Flask` for quick development and integration with OAuth for authentication.

-   Communication:

    -   Provides data to the Feed Service about user relationships.
    -   Exposes RESTful APIs for user management (e.g., profile updates, relationship management).

-   Example Code (Node.js with Express.js):

    ```javascript
    // Fetch user profile
    app.get("/users/:id", async (req, res) => {
        try {
            const user = await User.findById(req.params.id);
            res.json(user);
        } catch (err) {
            res.status(500).json({ message: err.message });
        }
    });

    // Update user profile
    app.put("/users/:id", async (req, res) => {
        try {
            const user = await User.findByIdAndUpdate(req.params.id, req.body, { new: true });
            res.json(user);
        } catch (err) {
            res.status(500).json({ message: err.message });
        }
    });
    ```

## Scalability Considerations

### Load Balancing

-   Objective: Distribute incoming requests evenly across multiple application servers to ensure high availability and reliability.

-   Approach:

    -   Layer 7 Load Balancer: Use L7 load balancing to route requests based on URL paths (e.g., `/getNewsFeed` for feed retrieval, `/createPost` for posting content).
    -   This allows requests to be directed to the appropriate microservice, optimizing performance and resource utilization.
    -   Scaling: Auto-scale application servers based on real-time traffic to handle spikes in user activity.

-   Example: NGINX Configuration for Layer 7 Load Balancing

    ```nginx
    http {
        upstream feed_service {
            server feed_service_1:8080;
            server feed_service_2:8080;
        }

        upstream post_service {
            server post_service_1:8080;
            server post_service_2:8080;
        }

        server {
            listen 80;

            location /getNewsFeed {
                proxy_pass http://feed_service;
            }

            location /createPost {
                proxy_pass http://post_service;
            }
        }
    }
    ```

### Database Replication

-   Objective: Ensure high availability, data redundancy, and fault tolerance through data replication.

-   Approach:

    -   Database Replication: Implement **multi-master replication** for databases like `Cassandra`, where all nodes can accept write operations, enhancing availability and scalability.
    -   Failover Strategy: Implement automated failover mechanisms to switch to a replica node in case of a primary node failure, ensuring uninterrupted service.

-   Example: Cassandra Multi-Master Configuration

    ```yaml
    cluster_name: "NewsFeedCluster"
    num_tokens: 256
    seed_provider:
        - class_name: org.apache.cassandra.locator.SimpleSeedProvider
        parameters:
            - seeds: "192.168.1.1,192.168.1.2"

    hinted_handoff_enabled: true
    max_hint_window_in_ms: 10800000 # 3 hours
    auto_bootstrap: true
    ```

### Database Sharding

-   Objective: Efficiently distribute data across multiple database nodes to handle high volumes of traffic and data.

-   Approach:

    -   User Data: Shard based on `user_id` to ensure that user-specific data (e.g., profile, settings) is evenly distributed.
    -   Post Data: Shard based on `post_id` to distribute posts across nodes, enabling the system to handle large-scale writes and queries efficiently.

-   Technique: Use _consistent hashing_ to distribute data across shards, which helps in maintaining an even distribution and simplifies the process of adding or removing nodes.

-   Example: Consistent Hashing in Java

    ```java
    import java.util.SortedMap;
    import java.util.TreeMap;

    public class ConsistentHashing {
        private final SortedMap<Integer, String> circle = new TreeMap<>();

        public void addNode(String node) {
            int hash = node.hashCode();
            circle.put(hash, node);
        }

        public String getNode(String key) {
            if (circle.isEmpty()) {
                return null;
            }
            int hash = key.hashCode();
            if (!circle.containsKey(hash)) {
                SortedMap<Integer, String> tailMap = circle.tailMap(hash);
                hash = tailMap.isEmpty() ? circle.firstKey() : tailMap.firstKey();
            }
            return circle.get(hash);
        }
    }
    ```

### Caching

-   Objective: Improve read performance and reduce the load on primary databases by caching frequently accessed data.

-   Approach:

    -   Redis Cache: Cache critical data like user feeds, user profiles, and frequently accessed posts to reduce the number of database queries.
    -   Hot Data: Store frequently accessed posts and user data in an in-memory cache to provide low-latency access.
    -   Cache Invalidation: Implement cache invalidation strategies (e.g., time-based or event-based) to ensure data consistency.
    -   Write-Through Caching:
        -   Immediately write data to both the cache and the database. This ensures that the cache is always in sync with the database, providing consistent reads.
    -   Write-Around Caching:
        -   Write data directly to the database and only cache it when read operations occur. This prevents caching of infrequently accessed data, reducing cache pollution.

-   Example: Caching in Redis with Node.js

    ```javascript
    const redis = require("redis");
    const client = redis.createClient();

    // Caching a user's feed
    function cacheUserFeed(userId, feedData) {
        client.setex(`feed:${userId}`, 3600, JSON.stringify(feedData)); // Cache for 1 hour
    }

    // Retrieving cached feed
    function getCachedUserFeed(userId, callback) {
        client.get(`feed:${userId}`, (err, data) => {
            if (err) return callback(err);
            if (data) {
                return callback(null, JSON.parse(data));
            } else {
                return callback(null, null); // Cache miss
            }
        });
    }
    ```

### Content Delivery Network (CDN)

-   Objective: Improve content delivery speed and reduce latency by caching static assets closer to end-users.

-   Approach:

    -   CDN Integration: Use a CDN to cache and deliver images, videos, and other static assets to users based on their geographic location.
    -   Edge Caching: Cache content at edge servers located near end-users to reduce the distance data needs to travel, improving load times.
    -   Dynamic Caching: Cache dynamic content (e.g., personalized feeds) at the edge to reduce the load on origin servers and improve response times.

-   Benefits:

    -   Faster Load Times: CDN caches content closer to users, reducing latency and improving page load times.
    -   Scalability: Offloads traffic from origin servers, enabling them to handle more requests and scale efficiently.
    -   Reliability: Redundant CDN infrastructure ensures high availability and fault tolerance.

-   Example: CDN Configuration with Amazon CloudFront

    ```yaml
    distributions:
        - id: "CDN_Distribution"
          origin: "newsfeed.com"
          enabled: true
          default_ttl: 3600
          min_ttl: 60
          max_ttl: 86400
          compress: true
          price_class: "PriceClass_All"
          restrictions:
              geo_restriction: "none"
          logging:
              bucket: "cdn-logs"
              prefix: "newsfeed"
          comment: "CDN distribution for News Feed service"
    ```

## Future Improvements

-   **Real-Time Updates:** Implement WebSockets or Server-Sent Events (SSE) to provide real-time updates for new posts, likes, comments, and shares.
-   **Content Moderation:** Integrate AI-based content moderation tools to filter out inappropriate or harmful content.
-   **Personalization:** Use machine learning algorithms to personalize the news feed based on user preferences and behavior.
