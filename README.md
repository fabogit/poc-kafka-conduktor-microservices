# poc-kafka-conduktor-microservices

> A distributed event-driven microservices architecture featuring Fastify, KafkaJS, MongoDB persistence, and Conduktor Platform observability.

**Inception (First Commit):** `2023-01-08` • **Batch / Horizon:** `batch-2023` • **Architecture Domain:** `Distributed Event Streaming / Kafka & Conduktor`

---

## 📌 Architecture & Overview

`node-kafka-conduktor` demonstrates an asynchronous event-driven microservice topology combining producer and consumer services coordinated through Apache Kafka and monitored via Conduktor Platform.

The architecture comprises two decoupled Node.js services alongside a production-ready container orchestration stack:
1. **Messages Service (Producer):** Fastify REST microservice (port 3000) that accepts message payloads, persists state in MongoDB, and publishes domain events (`message-created`) to an Apache Kafka topic.
2. **Notifications Service (Consumer):** Independent Fastify consumer (port 4000) that subscribes to `message-created`, processes event streams asynchronously, and manages local persistence.
3. **Conduktor Ecosystem:** Containerized Kafka broker, Zookeeper, Schema Registry, and Conduktor Platform (port 8080) for visual partition inspection, consumer lag monitoring, and broker telemetry via Prometheus JMX.

```
                              +-----------------------------------------------+
                              |             node-kafka-conduktor              |
                              |                                               |
[ HTTP Client / Postman ]     |  +-----------------------------------------+  |
       |                      |  | Messages Service (Port: 3000 - Producer)|  |
       | POST / (text)        |  +--------------------+--------------------+  |
       v                      |                       |                       |
+---------------------------+ |                       +--------+              |
| Fastify + JSON Schema     +-+-> [ MongoDB (DB) ]             | Publish      |
+---------------------------+ |                                v              |
                              |                 +---------------------------+ |
                              |                 | Apache Kafka Broker (9092)| |
                              |                 | Topic: 'message-created'  | |
                              |                 +-------------+-------------+ |
                              |                               |               |
                              |                               | Consume Event |
                              |                               v               |
                              |  +-----------------------------------------+  |
                              |  | Notifications Service (Port: 4000)      |  |
                              |  | (Consumer Group: notifications-app)     |  |
                              |  +--------------------+--------------------+  |
                              |                       |                       |
                              |                       v                       |
                              |                [ MongoDB (DB) ]               |
                              +-----------------------------------------------+
                                                      |
                                                      v
                                        +---------------------------+
                                        | Conduktor Platform (8080) |
                                        | & Prometheus JMX Exporter |
                                        +---------------------------+
```

* **Context:** Distributed systems reference architecture designed to evaluate Kafka producer-consumer lifecycles, schema management, and UI-driven cluster observability via Conduktor Platform.
* **Key Patterns:**
  * **Asynchronous Event-Driven Messaging:** Decoupling write ingestion from notification processing via Apache Kafka pub/sub topics.
  * **Consumer Group Load Balancing:** Scalable message distribution across consumer instances utilizing Kafka consumer group semantics.
  * **Graceful Teardown Lifecycle:** Comprehensive process signal listeners (`SIGINT`, `SIGTERM`, `SIGQUIT`) ensuring clean disconnects of Kafka producers/consumers and Fastify HTTP sockets.
  * **Centralized Cluster Observability:** Web UI partition inspection, message replay, and consumer lag analysis via Conduktor Platform.

---

## 🛠 Tech Stack

* **Streaming & Transport:** Apache Kafka (Confluent / Conduktor Platform 1.6.0), KafkaJS 2.2.3, Zookeeper, Schema Registry
* **Runtime & Framework:** Node.js (>= 18), Fastify 4.10.2, TypeScript 4.9.3, `tsx` 3.12.1
* **Database & ODM:** MongoDB, Mongoose 6.7.5
* **Validation & Tooling:** `json-schema-to-ts` 2.6.2, `nanoid` 4.0.0, Docker Compose, Postman Collections

---

## ✨ Features & Endpoints

* **Message Production API:** Fastify endpoint validating request schemas with `json-schema-to-ts` and publishing events to Kafka.
* **Stream Ingestion Consumer:** Real-time event subscription loop processing messages from `message-created`.
* **Conduktor Observability Suite:** Web console (port 8080) providing topic creation, partition rebalancing telemetry, and message payload browsing.
* **Docker Compose Topology:** Complete multi-container environment defining Kafka, Zookeeper, Conduktor Platform, and monitoring agents.

### Microservice Interfaces & Endpoints

| Service | Port | Protocol / Method | Endpoint / Topic | Description |
|---|---|---|---|---|
| `messages` | 3000 | `POST` | `/` | Ingest message payload, save to MongoDB, and publish event. |
| `kafka` | 9092 | `TOPIC` | `message-created` | Event bus topic carrying JSON serialized message records. |
| `notifications`| 4000 | `CONSUMER`| `message-created` | Asynchronously processes incoming message events. |
| `conduktor` | 8080 | `HTTP` | `/` | Web administration UI for Kafka cluster operations. |

---

## 🚀 Getting Started

### Prerequisites

* Docker & Docker Compose
* Node.js `>= 18.0.0`
* npm or pnpm

### Installation

```bash
# Clone the repository
git clone https://github.com/fabogit/poc-kafka-conduktor-microservices.git
cd poc-kafka-conduktor-microservices

# Install dependencies in both microservices
cd services/messages && npm install
cd ../notifications && npm install
cd ../..
```

### Running Kafka Infrastructure via Docker

```bash
# Start Kafka broker, Zookeeper, and Conduktor Platform
cd conduktor
docker compose up -d
cd ..
```

Access the Conduktor Platform web interface at [http://localhost:8080](http://localhost:8080).

### Running Microservices

```bash
# Terminal 1: Launch Messages Producer Service (Port 3000)
cd services/messages
npm run dev

# Terminal 2: Launch Notifications Consumer Service (Port 4000)
cd services/notifications
npm run dev
```

### Publishing a Test Message

```bash
curl -X POST http://localhost:3000/ \
  -H "Content-Type: application/json" \
  -d '{"text": "Hello Conduktor Event Bus"}'
```

---

## 📄 License & Author

Developed by [@fabogit](https://github.com/fabogit).
