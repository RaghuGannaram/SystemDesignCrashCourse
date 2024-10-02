# Chat Application System Design

## System Requirements

### Functional Requirements

-   **Messaging:**

    -   Users can send text messages to other users.
    -   Users can create groups and send messages to multiple users.
    -   Support for multimedia messages (images, videos, files).

-   **User Management:**

    -   User registration and authentication.
    -   User profile management (update profile picture, status etc...).
    -   Search for users and groups.

-   **Notifications:**
    -   Push notifications for new messages.
    -   User status indicators (online, offline, away, typing...).
    -   Message related notifications (e.g. unread message count, message delivered, message read).
    -   Group notifications (e.g., user added to group, user removed from group).
    -   Notification settings (mute notifications for specific users/groups).

### Non-Functional Requirements

-   **Security and Privacy:**

    -   Implement end-to-end encryption for messages and multimedia.
    -   Ensure secure user authentication with OAuth2 and multi-factor authentication.
    -   Comply with data protection regulations (e.g., GDPR).
    -   Secure storage and transmission of user data.

-   **Scalability and Performance:**

    -   Design the system to handle millions of concurrent users.
    -   Optimize system for low-latency messaging and quick response times.

-   **Resilience and Fault Tolerance:**

    -   Ensure the system can automatically recover from server failures.
    -   Design for minimal downtime during maintenance and upgrades.

-   **Maintainability and Extensibility:**

    -   Ensure the system is extensible for adding new features with minimal disruption.
    -   Implement logging and monitoring for system health and performance.

## Back of the Envelope Calculations

    Assumptions:

        -   Daily Active Users (DAUs):                          100 million
        -   Average No. of messages per user per day:           10 messages
        -   Average message size:                               1 KB
        -   Average image size in a message:                    500 KB
        -   Average video size in a message:                    5 MB
        -   Percentage of messages with images:                 5%
        -   Percentage of messages with videos:                 0.5%
        -   Average group size:                                 100 users
        -   write-to-read ratio:                                1:20

     Calculations:

        -   No. of daily messages:                              100 million users * 10 messages/user = 1 billion messages
        -   Total writes per day:                               1 billion writes
        -   Transactions per second (TPS):                      1 billion writes / 86400 seconds = ~11.6K writes per second

        -   Total data generated per day:                       (1 billion messages  * 1 KB ) + (1 billion messages * 5% * 500 KB ) + ( 1 billion messages * 0.5% * 5 MB) = 51 TB
        -   Storage for 10 years:                               51 TB * 365 days * 10 years = 186.15 PB

        -   Total reads per day:                                total writes * write-to-read ratio = 1 billion writes * 20 = 20 billion reads
        -   Queries per second (QPS):                           20 billion reads / 86400 seconds = ~231K reads per second

        -   Network bandwidth for writes:                       (1 billion messages * 1 KB )/ 86400 seconds = 11.5 MB/s +
                                                                (1 billion messages * 5% * 500 KB ) / 86400 seconds = 289.3 MB/s +
                                                                (1 billion messages * 0.5% * 5 MB ) / 86400 seconds = 289.3 MB/s
                                                                = 590.1 MB/s
        -   Network bandwidth for reads:                        590.1 MB/s * 20 = 11.8 GB/s

## API Design