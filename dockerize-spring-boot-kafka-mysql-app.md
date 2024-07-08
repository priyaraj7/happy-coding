# How to Dockerize a Spring Boot Application with Kafka and MySQL

## Introduction

In this guide, I will walk you through the process of Dockerizing a Spring Boot application that uses Kafka and MySQL. As someone who isn't a Java developer and lacks experience with Kafka, I found it challenging to run the app initially. Installing Kafka and MySQL on your system and starting Kafka Zookeeper and MySQL every time can be cumbersome. Dockerizing the app simplifies this process and can be integrated into a Jenkins pipeline for CI/CD.

We will use a docker-compose.yml file to define our multi-service environment. Here is the configuration:

```yml
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    tmpfs: "/datalog"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"
    healthcheck:
      test: nc -z localhost 2181 || exit -1
      interval: 10s
      timeout: 5s
      retries: 3
    restart: on-failure

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    ports:
      - "9092:9092"
    restart: on-failure
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    depends_on:
      - zookeeper
    healthcheck:
      test: nc -z localhost 9092 || exit -1
      start_period: 15s
      interval: 5s
      timeout: 10s
      retries: 10

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: "test123"
      MYSQL_DATABASE: "spring-boot-app"
      MYSQL_PASSWORD: "test123"
      MYSQL_TCP_PORT: 3306
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 20s
      retries: 10
    ports:
      - "3306:3306"
    volumes:
      - ./doc/backup.sql:/docker-entrypoint-initdb.d/backup.sql
    restart: on-failure

  spring-boot-app:
    build:
      context: .
      dockerfile: Dockerfile
      target: tms
    volumes:
      - ~/.m2/repository:/home/raptor/.m2/repository
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/spring-boot-app
      - KAFKA_BROKER=kafka:9092
      - KAFKA_BOOTSTRAPSERVERS=kafka:9092
    depends_on:
      mysql:
        condition: service_healthy
      kafka:
        condition: service_healthy
      zookeeper:
        condition: service_healthy
    ports:
      - "8080:8080"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://spring-boot-app:8080/actuator/health"]
      timeout: 120s
      retries: 10
    restart: on-failure
```

### Service Breakdown

#### Zookeeper

Zookeeper is essential for managing and coordinating the Kafka service. We use the confluentinc/cp-zookeeper:7.5.0 image.

**Ports**: Exposes port 2181.
**Healthcheck**: Checks if Zookeeper is running.
**Restart Policy**: Restarts on failure.

#### Kafka

Kafka, a distributed streaming platform, relies on Zookeeper for coordination. We use the confluentinc/cp-kafka:7.5.0 image.

**Ports**: Exposes port 9092.
**Environment Variables:**
**KAFKA_BROKER_ID**: Unique ID for the broker.
**KAFKA_ZOOKEEPER_CONNECT**: Connects Kafka to Zookeeper.
**KAFKA_ADVERTISED_LISTENERS**: Advertises the listener address.
**KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR**: Sets the replication factor.
**Depends On**: Ensures Kafka starts after Zookeeper.
**Healthcheck**: Checks if Kafka is running.
**Restart Policy**: Restarts on failure.

#### MySQL

We use MySQL for database management. The mysql:8.0 image is specified with a root password and a database for the Spring Boot app.

**Ports: Exposes port 3306.
**Environment Variables:\*\*
**MYSQL_ROOT_PASSWORD**: Sets the root password.
**MYSQL_DATABASE**: Creates the database.
**MYSQL_PASSWOR**D: Sets the database user password.
**Volumes**: Initializes the database with a backup SQL file.
**Healthcheck**: Checks if MySQL is running.
**Restart Policy**: Restarts on failure.

#### Spring Boot Application

This service builds and runs the Spring Boot application.

**Build Context**: Specifies the build context and Dockerfile.
**Volumes**: Maps the local Maven repository.
**Environment Variables**:
**SPRING_DATASOURCE_URL**: Configures the datasource URL.
**KAFKA_BROKER**: Sets the Kafka broker address.
**KAFKA_BOOTSTRAPSERVERS**: Sets the Kafka bootstrap servers.
**Depends On**: Ensures the app starts after Kafka, MySQL, and Zookeeper.
**Ports**: Exposes port 8080.
**Healthcheck**: Checks if the application is healthy.
**Restart Policy**: Restarts on failure.

### Running the Services

To start the services defined in the docker-compose.yml file, navigate to the directory containing the file and run:

```sh
docker-compose up
```

This command will pull the necessary images (if not already present), create the containers, and start the services.

## Conclusion

Dockerizing your Spring Boot application with Kafka and MySQL simplifies the process of setting up and running your application. This setup can also be integrated into your CI/CD pipeline using Jenkins, making your development workflow more efficient and manageable.
