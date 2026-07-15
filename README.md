# kafkaProducer

A Spring Boot microservice scaffold for a REST-to-Kafka message producer.

> **Status: Work in progress.** The project dependencies, configuration, and DTO are in place, but the REST controller and Kafka producer service that actually publish messages have not been implemented yet. This README documents the project as it currently exists and outlines what's needed to complete it.

## Overview

`kafkaProducer` is intended to be a small microservice that accepts a message over HTTP and publishes it to an Apache Kafka topic (`student_email`). It's built with Spring Boot and Spring Kafka, generated from Spring Initializr.

## Tech Stack

| Layer      | Technology |
|------------|------------|
| Language   | Java 21 |
| Framework  | Spring Boot 4.1.0 |
| Messaging  | Apache Kafka (via `spring-kafka`, `kafka-streams`) |
| Web        | Spring MVC (`spring-boot-starter-webmvc`) |
| Build tool | Maven (with Maven Wrapper) |
| Testing    | JUnit 5, `spring-boot-starter-kafka-test`, `spring-boot-starter-webmvc-test` |

## Project Structure

```
kafkaProducer/
├── pom.xml
├── mvnw / mvnw.cmd
├── src/
│   ├── main/
│   │   ├── java/com/cfs/kafkaproducer/
│   │   │   ├── KafkaProducerApplication.java   # Spring Boot entry point
│   │   │   └── dto/
│   │   │       └── MessageRequest.java         # record MessageRequest(String message)
│   │   └── resources/
│   │       └── application.properties          # Kafka + server config
│   └── test/
│       └── java/com/cfs/kafkaproducer/
│           └── KafkaProducerApplicationTests.java  # basic context-load test
```

## Intended Architecture

```
Client
  │  POST /messages  { "message": "..." }
  ▼
Spring MVC Controller   (not yet implemented)
  │
  ▼
Kafka Producer Service  (not yet implemented, uses KafkaTemplate)
  │
  ▼
Kafka Topic: student_email  (broker: localhost:9092)
```

The `MessageRequest` DTO (`record MessageRequest(String message)`) defines the expected JSON request shape and is the strongest signal of the intended REST contract.

## Configuration

Defined in `src/main/resources/application.properties`:

```properties
spring.application.name=kafkaProducer
server.port=8081
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.admin.auto-create=true
app.kafka.topic=student_email
```

- App runs on port **8081**
- Expects a Kafka broker at **localhost:9092**
- Producer uses `String` key/value serializers
- Topics are auto-created if missing
- Target topic name is configurable via `app.kafka.topic` (currently `student_email`)

## Prerequisites

- Java 21 (JDK)
- Maven (or use the included `./mvnw` wrapper — no local Maven install required)
- A running Kafka broker on `localhost:9092`

### Starting a local Kafka broker (Docker)

```bash
docker run -d --name kafka -p 9092:9092 apache/kafka:latest
```

Or use a `docker-compose.yml` with Kafka + Zookeeper/KRaft if you prefer a more permanent local setup.

## How to Run

1. Clone the repo:
   ```bash
   git clone https://github.com/mohitpawar61/kafkaProducer.git
   cd kafkaProducer
   ```

2. Start Kafka locally (see above), listening on `localhost:9092`.

3. Run the app using the Maven wrapper:
   ```bash
   ./mvnw spring-boot:run
   ```

   Or build a jar and run it directly:
   ```bash
   ./mvnw clean package
   java -jar target/kafkaProducer-0.0.1-SNAPSHOT.jar
   ```

4. The app starts on `http://localhost:8081`.

   **Note:** As of the current codebase, there are no REST endpoints implemented, so there is nothing to call yet. See "Next Steps" below.

## Running Tests

```bash
./mvnw test
```

Currently only a basic Spring context-load smoke test (`contextLoads()`) exists.

## Next Steps to Complete the Producer

To make this a working message producer, add:

1. **A Kafka producer service**, e.g.:
   ```java
   @Service
   public class KafkaProducerService {

       private final KafkaTemplate<String, String> kafkaTemplate;

       @Value("${app.kafka.topic}")
       private String topic;

       public KafkaProducerService(KafkaTemplate<String, String> kafkaTemplate) {
           this.kafkaTemplate = kafkaTemplate;
       }

       public void sendMessage(String message) {
           kafkaTemplate.send(topic, message);
       }
   }
   ```

2. **A REST controller**, e.g.:
   ```java
   @RestController
   @RequestMapping("/messages")
   public class MessageController {

       private final KafkaProducerService producerService;

       public MessageController(KafkaProducerService producerService) {
           this.producerService = producerService;
       }

       @PostMapping
       public ResponseEntity<String> publish(@RequestBody MessageRequest request) {
           producerService.sendMessage(request.message());
           return ResponseEntity.ok("Message sent to Kafka topic");
       }
   }
   ```

3. **Optional enhancements**:
   - Error handling / retry configuration for the producer
   - Integration tests using an embedded Kafka broker (`spring-boot-starter-kafka-test` is already a dependency)
   - A `KafkaConsumer` companion service to verify messages land on the topic
   - Externalized config for different environments (dev/staging/prod broker addresses)
   - Dockerfile / docker-compose for running the app alongside Kafka

## License

No license file is currently present in the repository.
