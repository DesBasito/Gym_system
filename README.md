# Gym CRM System

A comprehensive Gym Customer Relationship Management system built with microservices architecture using Spring Boot and Spring Cloud.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Services](#services)
- [Technology Stack](#technology-stack)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [API Documentation](#api-documentation)
- [Testing](#testing)
- [Project Structure](#project-structure)
- [Configuration](#configuration)
- [Security](#security)
- [Monitoring](#monitoring)
- [Contributing](#contributing)

---

## 🎯 Overview

The Gym CRM System is a microservices-based application designed to manage gym operations including trainers, trainees, training sessions, and workload tracking. The system uses modern Spring technologies and follows clean architecture principles.

### Key Features

- 👥 **User Management**: Trainee and Trainer registration and profiles
- 📅 **Training Sessions**: Create, update, and track training sessions
- 📊 **Workload Tracking**: Real-time trainer workload calculation with monthly summaries
- 🔐 **Security**: JWT-based authentication and authorization
- 🔄 **Service Discovery**: Eureka-based microservice discovery
- 🛡️ **Resilience**: Circuit breaker pattern for fault tolerance
- 📝 **Logging**: Two-level logging with MDC correlation IDs
- 📖 **API Documentation**: Swagger/OpenAPI 3.0 documentation

---

## 🏗 Architecture

### Microservices Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Eureka Server (8761)                     │
│                    Service Discovery                         │
└─────────────────────────────────────────────────────────────┘
                            │
            ┌───────────────┴───────────────┐
            │                               │
┌───────────▼──────────┐       ┌───────────▼──────────┐
│  Gym-CRM-System      │       │  Workload-Service    │
│  Port: 8080          │◄─────►│  Port: 8082          │
│                      │ Feign │                      │
│ - Trainers           │       │ - Workload Tracking  │
│ - Trainees           │       │ - Monthly Summaries  │
│ - Trainings          │       │ - In-Memory Storage  │
│ - PostgreSQL         │       │ - JWT Auth           │
└──────────────────────┘       └──────────────────────┘
```

### Communication Pattern

- **Service Discovery**: Eureka Server
- **Inter-Service Communication**: OpenFeign (declarative REST client)
- **Resilience**: Resilience4j Circuit Breaker with fallback
- **Load Balancing**: Client-side load balancing via Eureka

---

## 🔧 Services

### 1. Discovery Service (Eureka Server)
- **Application Name**: `Gym-discovery-service`
- **Port**: 8761
- **Purpose**: Service registry and discovery
- **URL**: `http://localhost:8761`
- **Configuration**:
  - Does not register itself with Eureka (`register-with-eureka: false`)
  - Does not fetch registry (`fetch-registry: false`)
  - Acts as standalone service discovery server

### 2. Gym-CRM-System
- **Application Name**: `gym-crm-system`
- **Port**: 8081 (dev), 8080 (prod)
- **Purpose**: Main CRM application
- **Database**: PostgreSQL
- **Key Features**:
  - User management (Trainers, Trainees)
  - Training session management
  - Authentication & Authorization (JWT)
  - Metrics and monitoring with Prometheus
  - Integration with workload-service via OpenFeign
- **Circuit Breaker**:
  - Sliding window size: 10 calls
  - Failure rate threshold: 50%
  - Wait duration in open state: 5s
- **Eureka Configuration**:
  - Registers with Eureka: `true`
  - Fetches registry: `true`
  - Lease renewal interval: 30s

### 3. Workload-Service
- **Application Name**: `workload-service`
- **Port**: 8082
- **Purpose**: Trainer workload tracking
- **Storage**: In-memory (ConcurrentHashMap)
- **Key Features**:
  - Real-time workload calculation
  - Monthly summary aggregation
  - Thread-safe operations
  - Stateless JWT authentication
- **Logging**:
  - Root level: INFO
  - Application level: DEBUG
  - Log file: `logs/workload-service.log`
- **Eureka Configuration**:
  - Registers with Eureka: `true`
  - Fetches registry: `true`
  - Lease renewal interval: 30s

---

## 🛠 Technology Stack

### Backend
- **Java**: 17
- **Spring Boot**: 3.4.5
- **Spring Cloud**: 2024.0.0
- **Spring Security**: JWT Authentication
- **Spring Data JPA**: Database access
- **PostgreSQL**: Main database
- **Liquibase**: Database migration
- **Netflix Eureka**: Service discovery
- **OpenFeign**: Declarative REST client
- **Resilience4j**: Circuit breaker

### Tools & Libraries
- **Lombok**: Code generation
- **MapStruct**: Object mapping
- **Swagger/OpenAPI**: API documentation
- **JUnit 5**: Testing framework
- **Mockito**: Mocking framework
- **Actuator**: Application monitoring

### Build & DevOps
- **Maven**: Build automation
- **Git**: Version control (mono-repo approach)

---

## 📦 Prerequisites

### Required
- **Java 17** or higher
- **Maven 3.8+**
- **PostgreSQL 14+**
- **Git**

### Optional
- **Docker** (for containerized deployment)
- **Postman** (for API testing)

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/DesBasito/Gym_system.git
cd Gym_system
```

### 2. Setup PostgreSQL Database

```sql
CREATE DATABASE gym_crm;
CREATE USER gym_user WITH PASSWORD 'gym_password';
GRANT ALL PRIVILEGES ON DATABASE gym_crm TO gym_user;
```

### 3. Configure Application Properties

Update `services/Gym-CRM-system/src/main/resources/application.yml`:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/gym_crm
    username: gym_user
    password: gym_password
```

### 4. Start Services (in order)

#### Step 1: Start Discovery Service (Eureka Server)
```bash
cd services/discovery-service
mvn spring-boot:run
```
Wait for Eureka to start at `http://localhost:8761`

#### Step 2: Start Workload Service
```bash
cd services/workload-service
mvn spring-boot:run
```
Service will register with Eureka at port 8082

#### Step 3: Start Gym-CRM-System
```bash
cd services/Gym-CRM-system
mvn spring-boot:run -Dspring-boot.run.profiles=dev
```
Service will register with Eureka at port 8081 (dev profile)

> **Note**: For production, use `-Dspring-boot.run.profiles=prod` (port 8080)

### 5. Verify Services

- **Eureka Dashboard**: http://localhost:8761
  - Check that both `gym-crm-system` and `workload-service` are registered
- **Gym-CRM Swagger**: http://localhost:8081/swagger-ui.html (dev)
  - API documentation and testing interface
- **Workload Swagger**: http://localhost:8082/swagger-ui.html
  - Workload service API documentation
- **Actuator Endpoints**:
  - Gym-CRM: http://localhost:8081/actuator/health
  - Workload: http://localhost:8082/actuator/health

---

## 📖 API Documentation

### Gym-CRM-System API

**Swagger UI**: `http://localhost:8080/swagger-ui.html`

#### Authentication
```bash
POST /api/auth/register
POST /api/auth/login
POST /api/auth/logout
PUT  /api/auth/change-password
```

#### Trainee Management
```bash
POST   /api/trainees
GET    /api/trainees/{username}
PUT    /api/trainees/{username}
DELETE /api/trainees/{username}
PUT    /api/trainees/{username}/trainers
```

#### Trainer Management
```bash
POST   /api/trainers
GET    /api/trainers/{username}
PUT    /api/trainers/{username}
GET    /api/trainers/not-assigned/{traineeUsername}
```

#### Training Management
```bash
POST   /api/trainings
GET    /api/trainings/trainee
GET    /api/trainings/trainer
GET    /api/trainings/types
```

### Workload-Service API

**Swagger UI**: `http://localhost:8082/swagger-ui.html`

```bash
POST /api/workload              # Update trainer workload
GET  /api/workload/{username}   # Get trainer workload
GET  /api/workload              # Get all workloads
```

---

## 🧪 Testing

### Run All Tests

```bash
# Gym-CRM-System
cd services/Gym-CRM-system
mvn clean test

# Workload-Service
cd services/workload-service
mvn clean test
```

### Test Coverage

```bash
mvn clean test jacoco:report
# Report: target/site/jacoco/index.html
```

### Integration Tests

```bash
mvn verify -P integration-tests
```

---

## 📂 Project Structure

```
Gym_system/
├── services/
│   ├── discovery-service/       # Eureka Server (Service Discovery)
│   │   ├── src/
│   │   │   ├── main/
│   │   │   │   ├── java/
│   │   │   │   │   └── epam/gym/
│   │   │   │   │       └── GymDiscoveryServiceApplication.java
│   │   │   │   └── resources/
│   │   │   │       └── application.yaml
│   │   │   └── test/
│   │   └── pom.xml
│   │
│   ├── Gym-CRM-system/          # Main CRM application
│   │   ├── src/
│   │   │   ├── main/
│   │   │   │   ├── java/
│   │   │   │   │   └── epam/gym/
│   │   │   │   │       ├── config/
│   │   │   │   │       │   ├── SecurityConfig.java
│   │   │   │   │       │   ├── OpenApiConfig.java
│   │   │   │   │       │   └── AsyncConfiguration.java
│   │   │   │   │       ├── constants/
│   │   │   │   │       ├── domain/
│   │   │   │   │       │   ├── dto/
│   │   │   │   │       │   ├── models/
│   │   │   │   │       │   └── services/
│   │   │   │   │       └── infrastructure/
│   │   │   │   │           ├── client/         # Feign clients
│   │   │   │   │           │   ├── WorkloadServiceClient.java
│   │   │   │   │           │   ├── WorkloadServiceClientFallbackFactory.java
│   │   │   │   │           │   ├── config/
│   │   │   │   │           │   └── dto/
│   │   │   │   │           ├── controllers/
│   │   │   │   │           ├── entities/
│   │   │   │   │           ├── repositories/
│   │   │   │   │           ├── security/
│   │   │   │   │           └── monitoring/
│   │   │   │   └── resources/
│   │   │   │       ├── application.yml
│   │   │   │       ├── application-dev.yml
│   │   │   │       ├── application-test.yml
│   │   │   │       └── application-prod.yml
│   │   │   └── test/
│   │   └── pom.xml
│   │
│   └── workload-service/        # Workload tracking microservice
│       ├── src/
│       │   ├── main/
│       │   │   ├── java/
│       │   │   │   └── abu/epam/com/workloadservice/
│       │   │   │       ├── config/
│       │   │   │       │   ├── SecurityConfig.java
│       │   │   │       │   └── OpenApiConfig.java
│       │   │   │       ├── domain/
│       │   │   │       │   ├── dto/
│       │   │   │       │   │   └── WorkloadRequest.java
│       │   │   │       │   ├── model/
│       │   │   │       │   │   ├── TrainerWorkload.java
│       │   │   │       │   │   ├── TrainerYearlySummary.java
│       │   │   │       │   │   └── TrainerMonthlySummary.java
│       │   │   │       │   └── service/
│       │   │   │       │       └── WorkloadService.java
│       │   │   │       └── infrastructure/
│       │   │   │           ├── controller/
│       │   │   │           │   ├── WorkloadController.java
│       │   │   │           │   └── GlobalExceptionHandler.java
│       │   │   │           ├── filter/
│       │   │   │           │   └── TransactionLoggingFilter.java
│       │   │   │           └── security/
│       │   │   │               ├── JwtAuthenticationFilter.java
│       │   │   │               └── JwtUtil.java
│       │   │   └── resources/
│       │   │       ├── application.yml
│       │   │       └── logback-spring.xml
│       │   └── test/
│       │       └── java/
│       │           └── abu/epam/com/workloadservice/
│       │               └── domain/service/
│       │                   └── WorkloadServiceTest.java
│       └── pom.xml
│
├── .gitmodules                  # Git submodules configuration
├── Task_Microservices.pdf      # Requirements documentation
└── README.md                    # This file
```

---

## ⚙️ Configuration

### Environment Variables

```bash
# Database (Gym-CRM-System)
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=gym_crm
export DB_USER=gym_user
export DB_PASSWORD=gym_password

# JWT (Shared across services)
export JWT_SECRET=404E635266556A586E3272357538782F413F4428472B4B6250645367566B5970
export JWT_EXPIRATION=86400000  # 24 hours in milliseconds

# Eureka
export EUREKA_URL=http://localhost:8761/eureka/
```

### Service Ports

| Service | Dev Port | Prod Port |
|---------|----------|-----------|
| Discovery Service | 8761 | 8761 |
| Gym-CRM-System | 8081 | 8080 |
| Workload-Service | 8082 | 8082 |

### Application Profiles

- **dev**: Development profile (port 8081, detailed logging)
- **test**: Testing profile (H2 in-memory database)
- **prod**: Production profile (port 8080, optimized settings)

```bash
# Development
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# Production
mvn spring-boot:run -Dspring-boot.run.profiles=prod
```

### OpenFeign Configuration (Gym-CRM-System)

```yaml
spring:
  cloud:
    openfeign:
      circuitbreaker:
        enabled: true
      client:
        config:
          default:
            connect-timeout: 5000    # 5 seconds
            read-timeout: 5000       # 5 seconds
            logger-level: basic
```

### Circuit Breaker Configuration

Both services use Resilience4j with default configuration:
- **Sliding window size**: 10 calls
- **Minimum calls**: 5 (before calculating failure rate)
- **Failure rate threshold**: 50%
- **Wait duration in open state**: 5 seconds
- **Half-open state calls**: 3 (permitted calls)
- **Automatic transition**: Enabled (from OPEN to HALF_OPEN)

---

## 🔐 Security

### JWT Authentication

All API endpoints (except `/api/auth/*`) require JWT authentication.

#### JWT Configuration

Both services use the same JWT secret for token validation:
```yaml
jwt:
  secret: 404E635266556A586E3272357538782F413F4428472B4B6250645367566B5970
  expiration: 86400000  # 24 hours
```

#### Get JWT Token

```bash
# Login via Gym-CRM-System
curl -X POST http://localhost:8081/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john.doe",
    "password": "password123"
  }'
```

Response:
```json
{
  "token": "eyJhbGciOiJIUzUxMiJ9...",
  "type": "Bearer"
}
```

#### Use Token

```bash
# Access Gym-CRM-System endpoints
curl -X GET http://localhost:8081/api/trainees/john.doe \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9..."

# Access Workload-Service endpoints
curl -X GET http://localhost:8082/api/workload/jane.smith \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9..."
```

> **Note**: The same JWT token works for both services as they share the same secret key

### Authorization

- **TRAINEE**: Can manage own profile and trainings
- **TRAINER**: Can manage own profile and view trainees
- **ADMIN**: Full system access

---

## 📝 Logging

### Two-Level Logging with MDC

#### Transaction Level
```
[TRANSACTION_ID: abc-123] Request: GET /api/trainers/john.doe
[TRANSACTION_ID: abc-123] Response: 200 OK
```

#### Operation Level
```
[TRANSACTION_ID: abc-123] Finding trainer by username: john.doe
[TRANSACTION_ID: abc-123] Trainer found, mapping to DTO
```

### Log Configuration

Logs are configured in `logback-spring.xml`:
- **Console**: Colored output for development
- **File**: `logs/application.log` (rolled daily)
- **MDC**: Correlation IDs for request tracing

---

## 🔄 Microservices Communication

### Circuit Breaker Pattern

```java
@FeignClient(
    name = "workload-service",
    fallbackFactory = WorkloadServiceClientFallbackFactory.class
)
public interface WorkloadServiceClient {
    @PostMapping("/api/workload")
    void updateWorkload(@RequestBody WorkloadRequest request);
}
```

### Fallback Behavior

When workload-service is unavailable:
1. Circuit breaker catches exception
2. Fallback logs error
3. Main operation continues successfully
4. No data loss - workload can be recalculated

---

## 🎯 Usage Examples

### Register a Trainee

```bash
curl -X POST http://localhost:8081/api/trainees \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "John",
    "lastName": "Doe",
    "dateOfBirth": "1990-01-15",
    "address": "123 Main St"
  }'
```

Response:
```json
{
  "username": "john.doe",
  "password": "aBcD1234"
}
```

### Login to Get JWT Token

```bash
curl -X POST http://localhost:8081/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john.doe",
    "password": "aBcD1234"
  }'
```

Response:
```json
{
  "token": "eyJhbGciOiJIUzUxMiJ9...",
  "type": "Bearer"
}
```

### Create a Training Session

```bash
curl -X POST http://localhost:8081/api/trainings \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "traineeUsername": "john.doe",
    "trainerUsername": "jane.smith",
    "trainingName": "Morning Workout",
    "trainingType": "FITNESS",
    "trainingDate": "2026-02-15",
    "trainingDuration": 60
  }'
```

> **Note**: This request automatically triggers a workload update to workload-service via OpenFeign

### Check Trainer Workload

```bash
curl -X GET http://localhost:8082/api/workload/jane.smith \
  -H "Authorization: Bearer <token>"
```

Response:
```json
{
  "username": "jane.smith",
  "firstName": "Jane",
  "lastName": "Smith",
  "isActive": true,
  "years": {
    "2026": {
      "months": {
        "2": {
          "totalDuration": 60
        }
      }
    }
  }
}
```

---

## 📊 Monitoring

### Actuator Endpoints

**Gym-CRM-System**: `http://localhost:8081/actuator` (dev)
- `/actuator/health` - Health check with full details
- `/actuator/metrics` - Application metrics (read-only)
- `/actuator/info` - Application info
- `/actuator/prometheus` - Prometheus metrics export (read-only)

Configuration:
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus,metrics
  endpoint:
    health:
      show-details: always
      show-components: always
  metrics:
    tags:
      application: gym-crm-system
```

**Workload-Service**: `http://localhost:8082/actuator`
- `/actuator/health` - Health check with full details
- `/actuator/metrics` - Application metrics (read-only)
- `/actuator/info` - Application info
- `/actuator/prometheus` - Prometheus metrics export (read-only)

Configuration:
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  metrics:
    tags:
      application: workload-service
```

### Eureka Dashboard

Monitor all registered services at `http://localhost:8761`

**Features**:
- View all registered service instances
- Check service health status
- Monitor lease renewal intervals
- View service metadata

---

## 🐛 Troubleshooting

### Service Not Registering with Eureka

1. Check Eureka server is running
2. Verify `eureka.client.service-url.defaultZone` in `application.yml`
3. Check network connectivity

### JWT Token Issues

1. Ensure same JWT secret in both services
2. Check token expiration time
3. Verify `Authorization: Bearer <token>` header format

### Database Connection Failed

1. Verify PostgreSQL is running
2. Check database credentials
3. Ensure database exists

---

## 🤝 Contributing

### Branch Strategy

- `main` - Production-ready code
- `feature/*` - New features
- `bugfix/*` - Bug fixes
- `hotfix/*` - Production hotfixes

### Commit Messages

Follow conventional commits:
```
feat: Add trainer workload endpoint
fix: Resolve JWT token expiration issue
docs: Update API documentation
test: Add unit tests for TrainingService
```

---

## 📄 License

This project is part of EPAM Systems specialization task.

---

## 👥 Authors

- **Developer**: Abu
- **Organization**: EPAM Systems

---

## 🔗 Related Links

- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Spring Cloud Documentation](https://spring.io/projects/spring-cloud)
- [Netflix Eureka](https://github.com/Netflix/eureka)
- [OpenFeign](https://spring.io/projects/spring-cloud-openfeign)

---

**Last Updated**: February 2026