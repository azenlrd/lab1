## Product Choice
**Product:** Yandex Go
**Link:** [https://go.yandex.com/](https://go.yandex.com/)
**Description:** Yandex Go is a super-app providing ride-hailing, food delivery, and logistics services within a single ecosystem. It connects users with drivers and couriers to enable fast and convenient urban mobility and delivery.

## Main components
![Yandex Go Component Diagram](https://raw.githubusercontent.com/inno-se-toolkit/lab-01-market-product-and-git/4020c8c7e67e80677cd44fc46e6ee6893fce94b7/docs/diagrams/out/yandex-go/architecture-component/Component%20Diagram.svg)

[Yandex Go Component Diagram Code](../../docs/diagrams/src/yandex-go/component-diagram.puml)

**Components Description:**
1.  **Mobile App:** The primary client interface for users to interact with the ecosystem (order rides, food, etc.). It communicates with the backend via the API Gateway.
2.  **API Gateway:** A single entry point that accepts REST API requests from clients and routes them to appropriate backend microservices via gRPC.
3.  **Dispatch Service:** The core service responsible for matching passenger orders with available drivers based on location and status.
4.  **Pricing Service:** Calculates the trip cost dynamically, taking into account distance, traffic, and demand surges.
5.  **Maps & Routing Service:** Provides routing data and ETA calculations, integrating with Yandex Maps API.
6.  **Operational DB (YDB):** A distributed SQL database used for storing critical operational data with strong consistency.

## Data flow
![Yandex Go Sequence Diagram](https://raw.githubusercontent.com/inno-se-toolkit/lab-01-market-product-and-git/4020c8c7e67e80677cd44fc46e6ee6893fce94b7/docs/diagrams/out/yandex-go/architecture-sequence/Sequence%20Diagram.svg)

[Yandex Go Sequence Diagram Code](../../docs/diagrams/src/yandex-go/sequence-diagram.puml)

**Group Description (Booking & Async Dispatch):**
This group describes the process of confirming an order and finding a driver.
1.  The **Mobile App** sends a `createOrder` request to the **API Gateway**, which forwards it to the **Dispatch Service**.
2.  The **Dispatch Service** validates the payment method with the **Payment Service**.
3.  Upon successful validation, the order status is persisted as `SEARCHING`.
4.  The system performs a geospatial search for candidate drivers and runs a matching algorithm.
5.  When a driver is found, the **Dispatch Service** updates the order status to `ASSIGNED` and publishes a `RideAssigned` event to the **Kafka Event Bus**.
6.  The **Push Service** consumes this event and sends a notification ("Driver Found") to the **Mobile App**.

## Deployment
![Yandex Go Deployment Diagram](https://raw.githubusercontent.com/inno-se-toolkit/lab-01-market-product-and-git/4020c8c7e67e80677cd44fc46e6ee6893fce94b7/docs/diagrams/out/yandex-go/architecture-deployment/Deployment%20Diagram.svg)

[Yandex Go Deployment Diagram Code](../../docs/diagrams/src/yandex-go/deployment-diagram.puml)

**Description:**
The system is deployed on **Yandex Cloud Infrastructure**. User requests coming from Smartphones or Web Browsers pass through a **Load Balancer** to the **API Gateway**. The application tier consists of microservices (Dispatch, Pricing, etc.) running as **Pods** within a **Kubernetes Cluster**. They communicate via gRPC. Data persistence is handled by managed clusters: **Redis** for caching, **Kafka** for message brokering, and **Data Storage Clusters** (ClickHouse for analytics, YDB for operations). External integration involves calls to Yandex Pay and Maps APIs.

## Assumptions
1.  I assume that **YDB (Operational DB)** is used for high-throughput transactional data (OLTP) due to its strict consistency guarantees, while **ClickHouse** is used strictly for analytical queries (OLAP).
2.  I assume the **Dispatch Service** uses an in-memory geospatial index (likely within the Redis Cluster or internal memory) to perform driver searches within milliseconds.

## Open questions
1.  How does the system handle "split-brain" scenarios in the **Kubernetes Cluster** if the network between availability zones fails?
2.  What is the specific retention policy for the **Kafka Event Bus** topics? Do they store ride events indefinitely for history, or is data offloaded to ClickHouse immediately?

