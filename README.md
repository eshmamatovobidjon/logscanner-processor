# LogScanner Backend

Spring Boot REST API for LogScanner - a powerful log file analysis tool.

![Java](https://img.shields.io/badge/Java-21-orange.svg)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.4-brightgreen.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

## Overview

This is the backend service for LogScanner, providing:

- REST API for log file upload and processing
- Full-text search powered by Elasticsearch
- Async job processing with Redis
- Multi-format log parsing (JSON, CSV, text, standard formats)

## Tech Stack

- **Java 21**
- **Spring Boot 3.4**
- **Elasticsearch 8.x** - Log storage and search
- **Redis** - Job queue and caching
- **Maven** - Build tool

## Quick Start

### Prerequisites

- Java 21+
- Maven 3.9+
- Docker (for Elasticsearch & Redis)

### Development Setup

```bash
# Clone the repository
git clone https://github.com/YOUR_ORG/logscanner-backend.git
cd logscanner-backend

# Start infrastructure (Elasticsearch + Redis)
docker-compose -f docker-compose.dev.yml up -d

# Run the application
./mvnw spring-boot:run

# API available at http://localhost:8080
```

### Using Docker

```bash
# Build Docker image
docker build -t logscanner-backend .

# Run with infrastructure
docker-compose up -d
```

## Project Structure

```
src/main/java/com/star/logscanner/
├── config/                 # Configuration classes
│   ├── AsyncConfig.java
│   ├── CorsConfig.java
│   ├── ElasticsearchConfig.java
│   └── RedisConfig.java
├── controller/             # REST controllers
│   └── LogController.java
├── service/                # Business logic
│   ├── LogProcessingService.java
│   └── LogQueryService.java
├── parser/                 # Log parsers
│   ├── LogParser.java
│   ├── JsonLogParser.java
│   ├── CsvLogParser.java
│   ├── TextLogParser.java
│   └── LogParserFactory.java
├── processor/              # File processing
│   ├── FileStreamProcessor.java
│   └── BatchProcessor.java
├── query/                  # Search query builder
│   └── LogQueryBuilder.java
├── entity/                 # Domain entities
│   ├── LogEntry.java
│   └── JobStatus.java
├── dto/                    # Data transfer objects
│   └── query/
├── repository/             # Data repositories
│   ├── LogEntryRepository.java
│   └── JobStatusRepository.java
└── exception/              # Exception handling
    └── GlobalExceptionHandler.java
```

## API Endpoints

### Upload

```http
POST /logs/upload
Content-Type: multipart/form-data

file: <log file>
```

Response:
```json
{
  "success": true,
  "data": {
    "jobId": "abc123",
    "fileName": "app.log",
    "status": "PROCESSING"
  }
}
```

### Job Status

```http
GET /logs/jobs/{jobId}
```

Response:
```json
{
  "success": true,
  "data": {
    "jobId": "abc123",
    "status": "COMPLETED",
    "progress": 100,
    "totalLines": 15000,
    "processedLines": 15000,
    "errorCount": 0
  }
}
```

### Query Logs

```http
POST /logs/query
Content-Type: application/json

{
  "searchText": "error",
  "levels": ["ERROR", "WARN"],
  "startDate": "2024-01-01T00:00:00Z",
  "endDate": "2024-01-31T23:59:59Z",
  "page": 0,
  "size": 50,
  "sortField": "timestamp",
  "sortDirection": "DESC"
}
```

### Export

```http
POST /logs/export
Content-Type: application/json

{
  "searchText": "error",
  "format": "CSV"
}
```

### List Jobs

```http
GET /logs/jobs
```

### Delete Job

```http
DELETE /logs/jobs/{jobId}
```

## Configuration

### application.yml

```yaml
server:
  port: 8080

spring:
  servlet:
    multipart:
      max-file-size: 500MB
      max-request-size: 500MB

  elasticsearch:
    uris: ${ELASTICSEARCH_URL:http://localhost:9200}

  redis:
    host: ${REDIS_HOST:localhost}
    port: ${REDIS_PORT:6379}

app:
  file:
    max-size: 524288000  # 500MB
    allowed-extensions: log,txt,json,csv

  processing:
    batch-size: 1000
    thread-pool-size: 4
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `ELASTICSEARCH_URL` | http://localhost:9200 | Elasticsearch URL |
| `REDIS_HOST` | localhost | Redis host |
| `REDIS_PORT` | 6379 | Redis port |
| `SERVER_PORT` | 8080 | Application port |

## Supported Log Formats

### JSON Logs
```json
{"timestamp": "2024-01-15T10:30:00Z", "level": "ERROR", "message": "Connection failed"}
```

### Standard Log Format
```
2024-01-15 10:30:00.123 ERROR [main] com.app.Service - Connection failed
```

### CSV Logs
```csv
timestamp,level,message,source
2024-01-15T10:30:00Z,ERROR,Connection failed,app.log
```

### Plain Text
```
[ERROR] 2024-01-15 10:30:00 - Connection failed
```

## Building

```bash
# Build JAR
./mvnw clean package

# Build JAR (skip tests)
./mvnw clean package -DskipTests

# Build Docker image
docker build -t logscanner-backend .
```

## Testing

```bash
# Run all tests
./mvnw test

# Run specific test class
./mvnw test -Dtest=LogParserTest

# Run with coverage
./mvnw test jacoco:report
```

## Docker

### Dockerfile

Multi-stage build for optimized image:

```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-21-alpine AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Build & Push

```bash
# Build
docker build -t your-dockerhub/logscanner-backend:1.0.0 .

# Push
docker push your-dockerhub/logscanner-backend:1.0.0
```

## Development

### Prerequisites

1. Install Java 21
2. Install Maven 3.9+
3. Start Elasticsearch & Redis:

```bash
docker run -d --name elasticsearch \
  -p 9200:9200 \
  -e "discovery.type=single-node" \
  -e "xpack.security.enabled=false" \
  docker.elastic.co/elasticsearch/elasticsearch:8.11.0

docker run -d --name redis \
  -p 6379:6379 \
  redis:7-alpine
```

### IDE Setup

**IntelliJ IDEA:**
1. Import as Maven project
2. Enable annotation processing
3. Set SDK to Java 21

**VS Code:**
1. Install "Extension Pack for Java"
2. Open folder and let it import

## Related Repositories

- [logscanner](https://github.com/YOUR_ORG/logscanner) - Docker Compose & documentation
- [logscanner-ui](https://github.com/YOUR_ORG/logscanner-ui) - React frontend

## Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open Pull Request

## License

MIT License - see [LICENSE](LICENSE) for details.
