# 🐳 Docker for Java Developers — Quick Reference

> Personal notes from the [Docker Full Course](https://www.youtube.com/watch?v=lpKwgOpxpqg) covering Docker basics, Spring Boot containerization, Docker Compose, and Volumes.

---

## 📌 Table of Contents

- [Why Docker?](#why-docker)
- [Virtualization vs Containerization](#virtualization-vs-containerization)
- [Core Concepts](#core-concepts)
- [Essential Docker Commands](#essential-docker-commands)
- [Docker Architecture](#docker-architecture)
- [Running a JDK Container](#running-a-jdk-container)
- [Containerizing a Spring Boot App (Manual)](#containerizing-a-spring-boot-app-manual)
- [Dockerfile](#dockerfile)
- [Spring Boot + PostgreSQL](#spring-boot--postgresql)
- [Docker Compose](#docker-compose)
- [Docker Volumes](#docker-volumes)
- [Video Timestamps](#video-timestamps)

---

## Why Docker?

- Solves the **"it works on my machine"** problem
- Ensures consistent environments across dev, staging, and production
- Simplifies sharing projects among team members

---

## Virtualization vs Containerization

| Feature | Virtualization | Containerization |
|---|---|---|
| Runs | Full OS per VM | Shares host OS kernel |
| Speed | Slower startup | Fast startup |
| Resource use | High (RAM/CPU) | Lightweight |
| Portability | VM snapshots | Container images |

---

## Core Concepts

| Term | Description |
|---|---|
| **Image** | Lightweight blueprint with instructions to create a container |
| **Container** | Running instance of an image |
| **Docker Hub** | Public registry to pull/push images |
| **Dockerfile** | Script to automate image creation |
| **Docker Compose** | Tool to manage multi-container applications |
| **Volume** | Persistent storage that survives container restarts |

---

## Essential Docker Commands

```bash
# Pull an image from Docker Hub
docker pull <image>:<tag>

# Run a container
docker run -dit --name <container_name> <image>:<tag>

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# List images
docker images

# Stop a container
docker stop <container_name>

# Remove a container
docker rm <container_name>

# Remove an image
docker rmi <image>:<tag>

# Copy file into a container
docker cp <local_file> <container_name>:<container_path>

# Execute a command inside a running container
docker exec -it <container_name> bash

# View container logs
docker logs <container_name>
```

---

## Docker Architecture

```
Developer (CLI)
      |
  Docker Client
      |
  Docker Daemon (dockerd)
      |
  +---+---+
  |       |
Images  Containers
  |
Docker Hub (Public / Private Registry)
```

---

## Running a JDK Container

```bash
# Pull and run OpenJDK 22
docker run -dit openjdk:22-jdk

# Enter the container interactively
docker exec -it <container_name> bash

# Run a simple Java program inside the container
javac Hello.java
java Hello
```

---

## Containerizing a Spring Boot App (Manual)

> Build the jar locally, copy it into a container, then commit as an image.

```bash
# 1. Package the Spring Boot app (skip tests)
mvn package -DskipTests

# 2. Run an OpenJDK container (detached)
docker run -dit openjdk:22-jdk

# 3. Copy the jar into the container
docker cp target/rest-demo.jar bold_kilby:/tmp

# 4. Commit the container as a new image
docker commit --change='CMD ["java","-jar","/tmp/rest-demo.jar"]' bold_kilby telusko/rest-demo:v2

# 5. Run the new image with port mapping
docker run -p 8080:8080 telusko/rest-demo:v2
```

> **Port mapping:** `-p <host_port>:<container_port>`

---

## Dockerfile

> Automates the manual steps above.

```dockerfile
FROM openjdk:22-jdk
COPY target/rest-demo.jar /tmp/rest-demo.jar
CMD ["java", "-jar", "/tmp/rest-demo.jar"]
EXPOSE 8080
```

```bash
# Build the image from Dockerfile
docker build -t telusko/rest-demo:v3 .

# Run it
docker run -p 8080:8080 telusko/rest-demo:v3
```

---

## Spring Boot + PostgreSQL

### `application.yml` (inside container, point to DB container name)

```yaml
spring:
  datasource:
    url: jdbc:postgresql://postgres-db:5432/studentdb
    username: postgres
    password: yourpassword
  jpa:
    hibernate:
      ddl-auto: update
```

> ⚠️ Use the **container name** (e.g., `postgres-db`) instead of `localhost` when containers talk to each other.

---

## Docker Compose

> Manages multiple containers with a single file.

```yaml
version: '3'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - postgres-db
    networks:
      - mynetwork

  postgres-db:
    image: postgres:15
    environment:
      POSTGRES_DB: studentdb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: yourpassword
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - mynetwork

volumes:
  pgdata:

networks:
  mynetwork:
```

```bash
# Start all services
docker-compose up --build

# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

---

## Docker Volumes

> Keeps data safe even when containers are stopped or removed.

```bash
# Create a named volume
docker volume create pgdata

# Mount a volume when running a container
docker run -v pgdata:/var/lib/postgresql/data postgres:15

# List volumes
docker volume ls

# Remove a volume
docker volume rm pgdata
```

---

## Video Timestamps

| Topic | Time |
|---|---|
| What problem Docker solves | [00:04](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=4s) |
| Solution with Containerization | [00:21:36](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=1296s) |
| What is Docker | [00:26:06](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=1566s) |
| Docker Setup | [00:33:19](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=1999s) |
| Running first container | [00:43:33](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=2613s) |
| Docker Commands | [00:53:52](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=3232s) |
| Docker Architecture | [00:57:22](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=3442s) |
| Running JDK Docker Container | [01:05:00](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=3900s) |
| Packing the Spring Boot web app | [01:13:05](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=4385s) |
| Running Spring Boot on Docker | [01:21:47](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=4907s) |
| Dockerfile for Docker Images | [01:30:12](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=5412s) |
| Web app with Postgres | [01:46:17](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=6377s) |
| Docker Compose | [01:58:57](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=7137s) |
| Running multiple containers | [02:06:39](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=7599s) |
| Docker Volumes | [02:10:50](https://www.youtube.com/watch?v=lpKwgOpxpqg&t=7850s) |

---

> 💡 **Tip:** 16 GB RAM recommended for smooth Docker usage on local machines.
