# IoT Platform API

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
![Java](https://img.shields.io/badge/Java-1.8%2B-ED8B00?style=flat&logo=java&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-2.x-6DB33F?style=flat&logo=spring&logoColor=white)
![Kafka](https://img.shields.io/badge/Apache_Kafka-E23127?style=flat&logo=apache-kafka&logoColor=white)
![InfluxDB](https://img.shields.io/badge/InfluxDB-22ADF6?style=flat&logo=influxdb&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)

## Project Overview
The **IoT Platform API** is a high-performance **Spring Boot** application architected to act as a scalable data ingestion and reporting service for IoT devices, primarily temperature sensors. 

By leveraging **Apache Kafka** for decoupled event-streaming and **InfluxDB** for optimized time-series data storage, this platform successfully manages real-time telemetry pipelines and aggregates data seamlessly, ensuring zero dropping under heavy data payload scenarios.

## Key Features
*   **High-Throughput Ingestion**: Exposes HTTP REST endpoints capable of consuming both individual data points and bulk sensor readings efficiently.
*   **Asynchronous Event Streaming**: Buffers incoming readings via an **Apache Kafka** producer/consumer architectural pattern, maximizing resilience and avoiding synchronous database bottlenecks.
*   **Time-Series Optimization**: Writes streams to **InfluxDB**, utilizing sequential time-range optimizations that make metric lookups and aggregations lightning fast.
*   **Real-time Reporting API**: Allows for customized, time-bounded data retrieval so frontend clients and dashboards can reliably reconstruct sensor history and visualize trends.
*   **Ready-to-Deploy**: Fully integrated with **Docker & Docker Compose** for streamlined bootstrapping of messaging and storage infrastructure alongside the core backend.

## Architecture Workflow
1. **Data Ingestion**: A sensor emits `.json` payloads securely via REST (`POST /temperatures`).
2. **Streaming**: The controller immediately wraps the payload and publishes it asynchronously to an Apache Kafka topic, quickly freeing up the HTTP thread.
3. **Persistence**: A Kafka Consumer seamlessly pulls the payload in the background, mapping it cleanly to an InfluxDB `Point` and persisting it in structural time-series order.
4. **Data Querying**: Users reconstruct sensor activity securely through the `GET /temperatures?startTime={}&endTime={}` endpoint, retrieving highly concentrated historic data points directly from the aggregated database.

<br/>

## Tech Stack
*   **Frameworks & Languages:** Java 8+, Spring Boot, Maven
*   **Message Broker:** Apache Kafka, ZooKeeper
*   **Database:** InfluxDB
*   **Containerization:** Docker, Docker Compose

---

## Local Setup & Execution

### Prerequisites
- **JDK 1.8+** (Tested with Oracle JDK)
- **Maven 3.6.x+**
- **Docker** & **Docker Compose**

### 1. Spin up Infrastructure (Kafka, ZooKeeper, InfluxDB)
```bash
$ docker-compose up -d
```

### 2. Build the Application
```bash
$ mvn clean compile install
```

### 3. Run the Backend Server
```bash
$ mvn spring-boot:run
# or
$ java -jar target/iot-platform-api-1.0.0.jar
```
The application will start on `localhost:8080`.

---

## API Endpoints & Usage

### 1. Record Single Temperature Reading
**Request:**
```bash
curl -X POST localhost:8080/temperatures \
-H 'Content-Type: application/json' \
-d '{ "unixTimestamp": 1563142700, "temperatureInFahrenheit": 20 }'
```
**Response:**
```json
{"success":true}
```

### 2. Record Batch Temperature Readings (Offline Data Sync)
**Request:**
```bash
curl -X POST localhost:8080/batch/temperatures \
-H 'Content-Type: application/json' \
-d '[{ "unixTimestamp": 1563142701, "temperatureInFahrenheit": 21 }, { "unixTimestamp": 1563142702, "temperatureInFahrenheit": 22 }]'
```
**Response:**
```json
[{"success":true},{"success":true}]
```

### 3. Fetch Aggregated Data Report (By Time-Range)
**Request:**
```bash
curl -X GET 'http://localhost:8080/temperatures?startTime=1563142700&endTime=1563142800' \
-H 'Content-Type: application/json'
```
**Response:**
```json
{
   "deviceId":"e01a7bc8-ee40-48ba-80ee-f8acbaba5f14",
   "data":[
      {
         "unixTimestamp":1563142700,
         "temperatureInFahrenheit":20.0
      },
      {
         "unixTimestamp":1563142701,
         "temperatureInFahrenheit":21.0
      }
   ]
}
```

---

## Testing

The repository incorporates robust unit and integration testing workflows:

**Run Unit Tests:**
```bash
mvn clean test-compile test
```

**Run Integration Tests:**
```bash
mvn clean test-compile verify -DskipTests=true
```

**Run All Tests (Unit + Integration):**
```bash
mvn clean test-compile verify
```

**Run Mutation Tests (Pitest):**
```bash
mvn clean test-compile test
mvn org.pitest:pitest-maven:mutationCoverage
```

---

## Future Technical Improvements
* Expand authentication pipelines natively utilizing JWT / OAuth2 standards rather than mocked credentials.
* Adopt MQTT protocol streaming integrations for lighter data transfer overhead dynamically bridged to Kafka.
* Further enrich the internal CI/CD pipelines to build/publish modular lightweight Docker images sequentially.

## License
Licensed under the [MIT License](LICENSE).
