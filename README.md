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

### 1. Eureka Server
- **Port**: 8761
- **Purpose**: Service registry and discovery
- **URL**: `http://localhost:8761`

### 2. Gym-CRM-System
- **Port**: 8080
- **Purpose**: Main CRM application
- **Database**: PostgreSQL
- **Key Features**:
  - User management (Trainers, Trainees)
  - Training session management
  - Authentication & Authorization (JWT)
  - Metrics and monitoring
  - Integration with workload-service

### 3. Workload-Service
- **Port**: 8082
- **Purpose**: Trainer workload tracking
- **Storage**: In-memory (ConcurrentHashMap)
- **Key Features**:
  - Real-time workload calculation
  - Monthly summary aggregation
  - Thread-safe operations
  - Stateless JWT authentication

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

#### Step 1: Start Eureka Server
```bash
cd services/eureka-server
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
mvn spring-boot:run
```
Service will register with Eureka at port 8080

### 5. Verify Services

- **Eureka Dashboard**: http://localhost:8761
- **Gym-CRM Swagger**: http://localhost:8080/swagger-ui.html
- **Workload Swagger**: http://localhost:8082/swagger-ui.html

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
│   ├── Gym-CRM-system/          # Main CRM application
│   │   ├── src/
│   │   │   ├── main/
│   │   │   │   ├── java/
│   │   │   │   │   └── epam/gym/
│   │   │   │   │       ├── config/
│   │   │   │   │       ├── constants/
│   │   │   │   │       ├── domain/
│   │   │   │   │       │   ├── dto/
│   │   │   │   │       │   ├── models/
│   │   │   │   │       │   └── services/
│   │   │   │   │       └── infrastructure/
│   │   │   │   │           ├── client/         # Feign clients
│   │   │   │   │           ├── controllers/
│   │   │   │   │           ├── entities/
│   │   │   │   │           ├── repositories/
│   │   │   │   │           └── security/
│   │   │   │   └── resources/
│   │   │   └── test/
│   │   └── pom.xml
│   │
│   ├── workload-service/        # Workload tracking microservice
│   │   ├── src/
│   │   │   ├── main/
│   │   │   │   ├── java/
│   │   │   │   │   └── abu/epam/com/workloadservice/
│   │   │   │   │       ├── config/
│   │   │   │   │       ├── domain/
│   │   │   │   │       │   ├── dto/
│   │   │   │   │       │   ├── model/
│   │   │   │   │       │   └── service/
│   │   │   │   │       └── infrastructure/
│   │   │   │   │           ├── controller/
│   │   │   │   │           ├── filter/
│   │   │   │   │           └── security/
│   │   │   │   └── resources/
│   │   │   └── test/
│   │   └── pom.xml
│   │
│   └── eureka-server/           # Service discovery
│       ├── src/
│       └── pom.xml
│
├── Task_Microservices.pdf      # Requirements documentation
└── README.md                    # This file
```

---

## ⚙️ Configuration

### Environment Variables

```bash
# Database
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=gym_crm
export DB_USER=gym_user
export DB_PASSWORD=gym_password

# JWT
export JWT_SECRET=your-secret-key-here
export JWT_EXPIRATION=86400000

# Eureka
export EUREKA_URL=http://localhost:8761/eureka/
```

### Application Profiles

- **default**: Development profile
- **test**: Testing profile (H2 in-memory database)
- **prod**: Production profile

```bash
mvn spring-boot:run -Dspring-boot.run.profiles=prod
```

---

## 🔐 Security

### JWT Authentication

All API endpoints (except `/api/auth/*`) require JWT authentication.

#### Get JWT Token

```bash
# Login
curl -X POST http://localhost:8080/api/auth/login \
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
curl -X GET http://localhost:8080/api/trainees/john.doe \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9..."
```

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
curl -X POST http://localhost:8080/api/trainees \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "John",
    "lastName": "Doe",
    "dateOfBirth": "1990-01-15",
    "address": "123 Main St"
  }'
```

### Create a Training Session

```bash
curl -X POST http://localhost:8080/api/trainings \
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

### Check Trainer Workload

```bash
curl -X GET http://localhost:8082/api/workload/jane.smith \
  -H "Authorization: Bearer <token>"
```

---

## 📊 Monitoring

### Actuator Endpoints

**Gym-CRM-System**: `http://localhost:8080/actuator`
- `/actuator/health` - Health check
- `/actuator/metrics` - Application metrics
- `/actuator/info` - Application info

**Workload-Service**: `http://localhost:8082/actuator`

### Eureka Dashboard

Monitor all registered services at `http://localhost:8761`

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