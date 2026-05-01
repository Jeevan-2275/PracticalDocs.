# Cloud Computing Practical Exam Questions and Solutions

## Q1. System Design: Collaborative Coding Platform
**Scenario:** 
You need to design a platform for coding, contests, pair programming, and real-time feedback. 

**Constraints:** 
* Remote code execution
* High concurrency (especially during contests)
* Multi-language support
* Low latency

**Tasks & Focus Areas:** 
Design a secure execution system and plan the container infrastructure. Focus on Containerization (Docker), Isolation, Security, Auto-scaling, and Load Balancing.

**Solution / Architecture Design:**

**1. Secure Remote Code Execution & Containerization (Docker):**
* **Isolation (Sandboxing):** User-submitted code must never run directly on host servers. Every code execution happens inside an ephemeral, isolated Docker container. 
* **Multi-language Support:** We will maintain pre-built Docker images containing compilers/interpreters for various languages (e.g., `python:3.9-alpine`, `openjdk:17-alpine`, `gcc`).
* **Security & Constraints:** 
  * Containers are run with read-only filesystems.
  * We strip all network access (`--network none`) to prevent downloading malicious payloads or launching DDoS attacks.
  * Use Linux `cgroups` via Docker limits to restrict resources: Maximum RAM (e.g., 256MB), CPU limits, and a strict Time-to-Live (TTL) execution timeout (e.g., 5 seconds) to prevent infinite loop attacks.

**2. Auto-Scaling Infrastructure:**
* Use a container orchestration tool like **Kubernetes (K8s)** (e.g., AWS EKS).
* Implement the **Horizontal Pod Autoscaler (HPA)**. During a coding contest, when CPU utilization or the queue length of code submissions spikes, K8s will automatically spin up additional worker nodes to handle the high concurrency. It will scale down post-contest to save costs.

**3. Load Balancing & Low Latency:**
* **Real-time Pair Programming:** Use **WebSockets** connected to an in-memory datastore like Redis (Pub/Sub) to synchronize keystrokes instantly between users with ultra-low latency.
* **Execution Pipeline:** Put an **Application Load Balancer (ALB)** in front of the web servers. When code is submitted, the web server pushes the payload into a message queue (like **RabbitMQ** or **Kafka**). A fleet of worker nodes pulls these tasks, runs the Docker container, and writes the output back, ensuring the web servers remain non-blocking.

---

## Q2. System Design: Smart Traffic System
**Scenario:** 
A city-wide system with live monitoring, AI-controlled traffic signals, congestion alerts, and administrative dashboards.

**Constraints:** 
* Massive real-time data from sensors/cameras
* Low latency required for signal switching
* Cost constraints
* 24/7 pipeline uptime

**Tasks & Focus Areas:** 
Design a real-time pipeline, choose between cloud vs. edge computing, and propose a scalable architecture. Focus on IoT integration, Event-driven systems, and Fault tolerance.

**Solution / Architecture Design:**

**1. Edge vs. Cloud Computing (Cost & Latency Strategy):**
* **Edge Computing:** Video feeds from intersection cameras generate massive data. Sending this raw video to the cloud violates cost constraints and introduces latency. Instead, we deploy lightweight Edge AI devices (like NVIDIA Jetson) at the intersections. These devices process the video locally, detect vehicle counts, and make immediate, low-latency decisions to change AI traffic signals.
* **Cloud Computing:** The Edge devices only send lightweight, aggregated metadata (e.g., "15 cars passed Northbound at 10:05 AM") to the Cloud. This drastically reduces bandwidth costs and powers city-wide historical dashboards.

**2. IoT Integration & Real-Time Event-Driven Pipeline:**
* **Ingestion:** Use a managed IoT broker like **AWS IoT Core** (using the lightweight MQTT protocol) to securely ingest data from thousands of intersection sensors.
* **Event Pipeline:** Route the incoming metadata into a highly scalable, event-driven streaming platform like **Apache Kafka** or **Amazon Kinesis**. 
* **Stream Processing:** Use a real-time analytics engine (like **Apache Flink** or **Kinesis Data Analytics**) connected to the stream. If the system detects a massive anomaly (e.g., zero cars moving for 5 minutes), it immediately triggers a "Congestion Alert" event to a notification service (like AWS SNS).

**3. Storage & Dashboards:**
* Send the live telemetry data to a **Time-Series Database** (like Amazon Timestream or InfluxDB) which is optimized for time-stamped metrics. This connects directly to a Grafana dashboard for city administrators.
* Dump older data into a cheap data lake (Amazon S3) for long-term storage and training future AI traffic models.

**4. Fault Tolerance:**
* **Cloud Level:** Deploy all cloud infrastructure across **Multi-AZ (Multiple Availability Zones)**. If one data center goes offline, the system seamlessly fails over to another.
* **Edge Level:** If an intersection loses internet connectivity, the edge device must have a fallback "failsafe" mechanism. It will abandon AI control and revert to a standard, pre-programmed timed traffic light sequence to prevent physical accidents until the connection is restored.
