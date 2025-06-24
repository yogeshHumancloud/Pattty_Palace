# 🍔 Patty Palace - Burger Delivery Platform

A comprehensive microservice-based burger delivery platform built with modern technologies and cloud-native architecture.

## 📋 Table of Contents

- [System Overview](#-system-overview)
- [Frontend Applications](#-frontend-applications)
- [Backend Microservices](#-backend-microservices)
- [Real-Time Delivery Tracking](#-real-time-delivery-tracking-with-kafka)
- [Infrastructure & DevOps](#-infrastructure--devops)
- [Security Implementation](#-security-implementation)
- [Monitoring & Observability](#-monitoring--observability)
- [Getting Started](#-getting-started)
- [Deployment](#-deployment)
- [Contributing](#-contributing)

## 🎯 System Overview

The platform consists of three main client applications backed by a scalable microservice architecture:

1. **Customer Mobile App** (React Native + Expo)
2. **Outlet Management Website** (Next.js)
3. **Delivery Partner App** (React Native + Expo)

## 📱 Frontend Applications

### 1. Customer Mobile App (React Native + Expo)

**Features:**
- Browse menu and customize burgers
- Apply coupons and loyalty points
- Real-time order tracking
- Payment integration
- Customer reviews and ratings

**Tech Stack:**
- React Native with Expo
- Redux Toolkit for state management
- React Query for data fetching
- Push notifications

**Key Screens:**
- Menu browsing and customization
- Shopping cart and checkout
- Order tracking with live updates
- User profile and rewards
- Payment methods management

### 2. Outlet Management Website (Next.js)

**Features:**
- Real-time order management
- Inventory control and alerts
- Menu updates and pricing
- Analytics dashboard
- Staff management

**Tech Stack:**
- Next.js 14 with TypeScript
- Tailwind CSS for styling
- Zustand for state management
- Real-time updates via WebSocket

**Key Pages:**
- Dashboard with key metrics
- Order queue and status management
- Inventory tracking and alerts
- Menu item management
- Sales analytics and reports

### 3. Delivery Partner App (React Native + Expo)

**Features:**
- Available order notifications
- Route optimization and navigation
- Real-time location tracking
- Earnings tracking
- OTP verification for deliveries

**Tech Stack:**
- React Native with Expo
- Maps integration (Google Maps)
- Background location tracking
- Push notifications

**Key Screens:**
- Available orders list
- Active delivery tracking
- Navigation and route optimization
- Earnings and payment history
- Profile and vehicle management

## 🏗️ Backend Microservices

### Core Services Architecture

#### 1. API Gateway Service
- **Technology:** Bun + Hono
- **Port:** 3000
- **Responsibilities:**
  - Request routing and load balancing
  - Rate limiting and throttling
  - API versioning
  - Request/response transformation
  - CORS handling

#### 2. Identity & Authentication Service
- **Technology:** Bun + Hono + OAuth 2.0
- **Database:** PostgreSQL
- **Port:** 3001
- **Responsibilities:**
  - User registration and authentication
  - OAuth 2.0 implementation (Google, Facebook, Apple)
  - JWT token management
  - Role-based access control
  - Password reset and email verification

#### 3. User Management Service
- **Technology:** Bun + Hono
- **Database:** PostgreSQL
- **Port:** 3002
- **Responsibilities:**
  - User profiles and preferences
  - Address management
  - Customer support tickets
  - User activity logging

#### 4. Menu & Catalog Service
- **Technology:** Bun + Hono
- **Database:** MongoDB
- **Port:** 3003
- **Responsibilities:**
  - Menu items and categories
  - Pricing and variants
  - Nutritional information
  - Menu availability by outlet
  - Seasonal items and customizations

#### 5. Inventory Management Service
- **Technology:** Bun + Hono
- **Database:** PostgreSQL
- **Cache:** Redis
- **Port:** 3004
- **Responsibilities:**
  - Real-time inventory tracking
  - Stock level monitoring
  - Automatic reorder alerts
  - Ingredient-based inventory
  - Multi-outlet inventory management

#### 6. Order Management Service
- **Technology:** Bun + Hono
- **Database:** PostgreSQL
- **Message Queue:** RabbitMQ
- **Port:** 3005
- **Responsibilities:**
  - Order creation and validation
  - Order status management
  - Order history
  - Bulk order processing
  - Order cancellation logic

#### 7. Payment Service
- **Technology:** Bun + Hono
- **Database:** PostgreSQL (encrypted)
- **Port:** 3006
- **Responsibilities:**
  - Payment processing (Stripe, Razorpay)
  - Refund management
  - Payment method storage
  - Transaction history
  - Wallet integration

#### 8. Delivery Management Service
- **Technology:** Bun + Hono
- **Database:** PostgreSQL + Redis
- **Message Streaming:** Kafka
- **Port:** 3007
- **Responsibilities:**
  - Delivery partner assignment
  - Route optimization
  - Real-time location streaming
  - Delivery time estimation
  - OTP generation and verification
  - Geofence event processing

#### 9. Loyalty & Rewards Service
- **Technology:** Bun + Hono
- **Database:** PostgreSQL
- **Port:** 3008
- **Responsibilities:**
  - Burger points calculation
  - Loyalty program management
  - Coupon validation
  - Referral program
  - Tier-based rewards

#### 10. Notification Service
- **Technology:** Bun + Hono
- **Message Queue:** RabbitMQ
- **Port:** 3009
- **Responsibilities:**
  - Push notifications (FCM)
  - SMS notifications
  - Email notifications
  - In-app notifications
  - Notification preferences

#### 11. Analytics & Reporting Service
- **Technology:** Bun + Hono
- **Database:** PostgreSQL + ClickHouse
- **Port:** 3010
- **Responsibilities:**
  - Sales analytics
  - Customer behavior tracking
  - Inventory reports
  - Performance metrics
  - Business intelligence

## 📡 Real-Time Delivery Tracking with Kafka

### Kafka Implementation Strategy

#### Producer Configuration

```typescript
// Delivery Partner App - Location Producer
const locationProducer = kafka.producer({
  maxInFlightRequests: 1,
  idempotent: true,
  transactionTimeout: 30000
});

await locationProducer.send({
  topic: 'delivery.location.updates',
  messages: [{
    key: deliveryId,
    value: JSON.stringify({
      deliveryId,
      driverId,
      orderId,
      customerId,
      coordinates: {
        latitude: 40.7128,
        longitude: -74.0060,
        accuracy: 5.0
      },
      metadata: {
        speed: 25.5,
        heading: 180,
        timestamp: Date.now(),
        batteryLevel: 85
      }
    }),
    partition: getPartitionByDeliveryId(deliveryId)
  }]
});
```

#### Consumer Groups

```typescript
// Customer App Consumer - Real-time tracking
const customerTrackingConsumer = kafka.consumer({
  groupId: 'customer-tracking-group',
  sessionTimeout: 30000,
  heartbeatInterval: 3000
});

// Analytics Consumer - Data processing
const analyticsConsumer = kafka.consumer({
  groupId: 'analytics-processing-group',
  sessionTimeout: 30000
});

// Outlet Dashboard Consumer - Management view
const outletConsumer = kafka.consumer({
  groupId: 'outlet-dashboard-group',
  sessionTimeout: 30000
});
```

#### Topic Partitioning Strategy

```
delivery.location.updates:
├── Partition 0: deliveryId % 10 == 0
├── Partition 1: deliveryId % 10 == 1
├── ...
└── Partition 9: deliveryId % 10 == 9

Benefits:
- Maintains order per delivery
- Enables parallel processing
- Distributes load evenly
```

### Real-Time Data Flow

```
Delivery Partner App
    ↓ (GPS coordinates every 10s)
Kafka Producer
    ↓ (delivery.location.updates)
Kafka Cluster (3 brokers)
    ↓ (Multiple consumers)
┌─────────────────┬─────────────────┬─────────────────┐
│  Customer App   │  Outlet Portal  │ Analytics Engine│
│  (Live tracking)│ (Fleet monitor) │ (Route analysis)│
└─────────────────┴─────────────────┴─────────────────┘
```

#### Geofence Event Processing

```typescript
// Geofence Producer
await geofenceProducer.send({
  topic: 'delivery.geofence.alerts',
  messages: [{
    key: `${deliveryId}-${zoneId}`,
    value: JSON.stringify({
      deliveryId,
      eventType: 'PICKUP_ZONE_ENTERED',
      zoneId: 'restaurant-zone-123',
      timestamp: Date.now(),
      location: { lat: 40.7128, lng: -74.0060 }
    })
  }]
});
```

### Kafka Configuration

#### Cluster Setup

```yaml
# docker-compose.yml
version: '3.8'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka-broker-1:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-broker-1:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: false
      KAFKA_LOG_RETENTION_HOURS: 168
```

#### Topic Configuration

```bash
# Create delivery tracking topics
kafka-topics --create \
  --topic delivery.location.updates \
  --partitions 10 \
  --replication-factor 3 \
  --config retention.ms=604800000 \
  --config segment.ms=3600000

kafka-topics --create \
  --topic delivery.geofence.alerts \
  --partitions 5 \
  --replication-factor 3 \
  --config retention.ms=259200000
```

### Event Sourcing for Delivery History

#### Benefits
- **Replay Capability:** Reconstruct delivery routes for analysis
- **Audit Trail:** Complete history of delivery events
- **Debug Support:** Investigate delivery issues with full context
- **Analytics:** Generate insights from historical location data

#### Implementation

```typescript
// Event Store Consumer
const eventStoreConsumer = kafka.consumer({
  groupId: 'event-store-group',
  fromBeginning: true
});

await eventStoreConsumer.subscribe({
  topics: [
    'delivery.location.updates',
    'delivery.status.events',
    'delivery.geofence.alerts'
  ]
});

// Store events for replay
await eventStoreConsumer.run({
  eachMessage: async ({ topic, partition, message }) => {
    await eventStore.store({
      topic,
      partition,
      offset: message.offset,
      key: message.key.toString(),
      value: JSON.parse(message.value.toString()),
      timestamp: message.timestamp
    });
  }
});
```

## 🏗️ Infrastructure & DevOps

### Message Queue & Streaming Architecture

#### RabbitMQ for Traditional Event Messaging

```
RabbitMQ Exchanges:
├── order.exchange
│   ├── order.created
│   ├── order.updated
│   ├── order.cancelled
│   └── order.completed
├── inventory.exchange
│   ├── inventory.updated
│   └── inventory.low_stock
├── payment.exchange
│   ├── payment.success
│   ├── payment.failed
│   └── refund.processed
└── notification.exchange
    ├── notification.push
    ├── notification.sms
    └── notification.email
```

#### Kafka for High-Volume Streaming Data

```
Kafka Topics:
├── delivery.location.updates
│   ├── Real-time GPS coordinates
│   ├── Speed and heading data
│   └── Accuracy measurements
├── delivery.route.events
│   ├── Route optimization changes
│   ├── Traffic condition updates
│   └── ETA recalculations
├── delivery.geofence.alerts
│   ├── Pickup zone entry/exit
│   ├── Dropoff zone proximity
│   └── Restricted area violations
├── delivery.status.stream
│   ├── Partner availability updates
│   ├── Delivery milestone events
│   └── Real-time capacity metrics
└── analytics.events.stream
    ├── User behavior tracking
    ├── Performance metrics
    └── Business intelligence data
```

### Database Strategy

- **PostgreSQL:** Transactional data (Users, Orders, Payments, Inventory)
- **MongoDB:** Flexible schemas (Menu, Reviews, Logs)
- **Redis:** Caching, session storage, real-time data
- **ClickHouse:** Analytics and reporting (optional)
- **Kafka:** Event streaming and location data persistence

### Caching Strategy

```
Redis Cache Layers:
├── Application Cache (L1)
│   ├── Menu items (TTL: 1 hour)
│   ├── User sessions (TTL: 24 hours)
│   └── Inventory levels (TTL: 5 minutes)
├── Database Query Cache (L2)
│   ├── Frequently accessed data
│   └── Aggregated reports
└── CDN Cache (L3)
    ├── Static assets
    └── API responses
```

### Containerization & Orchestration

```dockerfile
# Docker Services
├── API Gateway (Nginx + Bun)
├── Microservices (Bun + Hono)
├── Databases (PostgreSQL, MongoDB, Redis)
├── Message Queue (RabbitMQ)
├── Event Streaming (Kafka + Zookeeper)
├── Monitoring (Prometheus + Grafana)
└── Log Aggregation (ELK Stack)
```

```yaml
# Kubernetes Deployment
├── Namespaces
│   ├── production
│   ├── staging
│   └── development
├── Services & Deployments
│   ├── API Gateway (3 replicas)
│   ├── Core Services (2 replicas each)
│   └── Databases (StatefulSets)
└── ConfigMaps & Secrets
    ├── Environment configs
    └── Database credentials
```

## ☁️ AWS Services Integration

### Core AWS Services

- **EKS:** Kubernetes cluster management
- **RDS:** PostgreSQL managed database
- **DocumentDB:** MongoDB-compatible database
- **ElastiCache:** Redis managed service
- **S3:** File storage (images, documents)
- **CloudFront:** CDN for static assets
- **Route 53:** DNS management
- **Application Load Balancer:** Traffic distribution

### Additional AWS Services

- **SES:** Email notifications
- **SNS:** Push notifications
- **MSK:** Managed Kafka service for event streaming
- **Lambda:** Serverless functions for analytics
- **CloudWatch:** Monitoring and logging
- **Secrets Manager:** Credential management
- **IAM:** Access control

## 🔒 Security Implementation

### Authentication & Authorization

- **JWT Tokens:** Stateless authentication
- **OAuth 2.0:** Third-party login (Google, Facebook, Apple)
- **RBAC:** Role-based access control
- **API Keys:** Service-to-service communication

### Data Security

- **Encryption at Rest:** Database encryption
- **Encryption in Transit:** TLS/SSL
- **PII Protection:** Data masking and tokenization
- **GDPR Compliance:** Data privacy controls

## 📊 Monitoring & Observability

### Application Monitoring

- **Prometheus:** Metrics collection
- **Grafana:** Visualization dashboards
- **Jaeger:** Distributed tracing
- **ELK Stack:** Centralized logging

### Key Metrics

- **Business Metrics:** Orders/hour, revenue, customer satisfaction
- **Technical Metrics:** Response time, error rate, throughput
- **Infrastructure Metrics:** CPU, memory, disk usage

## 📈 Scalability Considerations

### Horizontal Scaling

- **Auto-scaling groups** for Kubernetes pods
- **Database read replicas** for read-heavy operations
- **Message queue clustering** for high availability

### Performance Optimization

- **Database indexing** strategies
- **Connection pooling** for database connections
- **Async processing** for non-critical operations
- **CDN integration** for static content
- **Kafka partitioning** for high-throughput streaming
- **Event sourcing** for delivery tracking replay

## 🚀 Getting Started

### Prerequisites

- Node.js 18+ and Bun
- Docker and Docker Compose
- Kubernetes (minikube for local development)
- PostgreSQL, MongoDB, Redis
- Kafka and Zookeeper

### Local Development Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-org/patty-palace.git
   cd patty-palace
   ```

2. **Start infrastructure services:**
   ```bash
   docker-compose -f docker-compose.dev.yml up -d
   ```

3. **Install dependencies for all services:**
   ```bash
   # API Gateway
   cd services/api-gateway && bun install
   
   # Microservices
   cd ../auth-service && bun install
   cd ../user-service && bun install
   # ... repeat for all services
   ```

4. **Set up environment variables:**
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

5. **Run database migrations:**
   ```bash
   cd scripts && ./run-migrations.sh
   ```

6. **Start all services:**
   ```bash
   # Using the development script
   ./scripts/start-dev.sh
   ```

### Frontend Applications Setup

1. **Customer Mobile App:**
   ```bash
   cd apps/customer-mobile
   npm install
   expo start
   ```

2. **Outlet Management Website:**
   ```bash
   cd apps/outlet-management
   npm install
   npm run dev
   ```

3. **Delivery Partner App:**
   ```bash
   cd apps/delivery-partner
   npm install
   expo start
   ```

## 🚀 Deployment

### Environment Setup

```
├── Development
│   ├── Local development with Docker Compose
│   └── Feature branch deployments
├── Staging
│   ├── Integration testing environment
│   └── Performance testing
└── Production
    ├── Blue-green deployment
    ├── Canary releases
    └── Rollback capabilities
```

### CI/CD Pipeline (Jenkins)

```groovy
pipeline {
  stages {
    ├── Source Code Checkout
    ├── Unit Tests (Jest + Vitest)
    ├── Integration Tests
    ├── Code Quality (SonarQube)
    ├── Security Scanning
    ├── Docker Build
    ├── Push to Registry
    ├── Deploy to Staging
    ├── E2E Tests
    └── Deploy to Production
  }
}
```

### Testing Strategy

```
Testing Pyramid:
├── Unit Tests (70%)
│   ├── Service logic tests
│   ├── Utility function tests
│   └── Component tests
├── Integration Tests (20%)
│   ├── API endpoint tests
│   ├── Database integration tests
│   └── Message queue tests
└── E2E Tests (10%)
    ├── User journey tests
    ├── Cross-service tests
    └── UI automation tests
```

### Production Deployment

1. **Build and push Docker images:**
   ```bash
   ./scripts/build-and-push.sh
   ```

2. **Deploy to Kubernetes:**
   ```bash
   kubectl apply -f k8s/production/
   ```

3. **Monitor deployment:**
   ```bash
   kubectl rollout status deployment/api-gateway
   ```

## 📝 API Documentation

API documentation is available at:
- **Development:** http://localhost:3000/docs
- **Staging:** https://api-staging.pattypalace.com/docs
- **Production:** https://api.pattypalace.com/docs

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Development Guidelines

- Follow the established coding standards
- Write comprehensive tests for new features
- Update documentation for any API changes
- Ensure all CI/CD checks pass

## 📞 Support

For support and questions:
- Create an issue in this repository
- Contact the development team at dev@pattypalace.com
- Check our documentation wiki

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**Built with ❤️ by the Patty Palace Team**
