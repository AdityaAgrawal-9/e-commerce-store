# 🛒 E-Commerce Store — Microservices Architecture

A scalable, production-grade e-commerce backend built with a **microservices architecture** using Java & Spring Boot. Each service is independently deployable, owns its own database, and communicates via REST APIs and asynchronous Kafka messaging.

---

## 📐 Architecture Overview

```
                    ┌──────────────────────────────┐
                    │    Load Balancer (AWS ELB)   │
                    └───────────────┬──────────────┘
                                    │
                    ┌───────────────▼──────────────┐
                    │      API Gateway             │
                    │(Routing, Auth, Rate Limiting)│
                    └──┬────────┬────────────┬─────┘
                       │        │            │
          ┌────────────▼─┐  ┌───▼──────┐ ┌───▼─────────┐
          │ Auth Service │  │ Product  │ │    Cart     │
          │   (MySQL)    │  │ Service  │ │  Service    │
          └──────┬───────┘  │(MySQL+ES)│ │(MongoDB +   │
                 │          └──────────┘ │  Redis)     │
                 │                       └──────┬──────┘
                 │                              │
          ┌──────▼──────────────────────────────▼──────┐
          │               Apache Kafka                 │
          │          (Async Event Bus)                 │
          └──────┬─────────────────────┬───────────────┘
                 │                     │
        ┌────────▼───────┐   ┌─────────▼──────┐   ┌──────────────────┐
        │  Order Service │   │Payment Service │   │Notification Svc  │
        │    (MySQL)     │   │   (MySQL)      │   │  (KAFKA)         │
        └────────────────┘   └───────────────-┘   └──────────────────┘
```

---

## 🗂️ Services At a Glance

| Service | Repository | Status | Database | Key Tech |
|---|---|---|---|---|
| 🔐 User & Auth Service | [authservice](https://github.com/AdityaAgrawal-9/authservice) | ✅ Implemented | MySQL | Spring Security, JWT, Kafka |
| 📦 Product Service | [productservice](https://github.com/AdityaAgrawal-9/productservice) | ✅ Implemented | MySQL | Elasticsearch |
| 🛒 Cart Service | [cartservice](https://github.com/AdityaAgrawal-9/cartservice) | 🚧 In Progress | MongoDB | Redis, Kafka |
| 💳 Payment Service | [paymentservice](https://github.com/AdityaAgrawal-9/paymentservice) | 🚧 In Progress | MySQL | Kafka |
| 📋 Order Service | [orderservice](https://github.com/AdityaAgrawal-9/orderservice) | 🚧 In Progress | MySQL | Kafka |
| 🔔 Notification Service | [notificationservice](https://github.com/AdityaAgrawal-9/notificationservice) | 🚧 In Progress | — | Kafka |

---

## 🔐 User & Auth Service
**Repository:** [AdityaAgrawal-9/authservice](https://github.com/AdityaAgrawal-9/authservice)

Handles all user identity, authentication, and session management. Issues JWTs that are validated by the API Gateway and consumed by all downstream services. Publishes user lifecycle events (e.g. new registration) to Kafka to trigger downstream flows like welcome notifications.

**Tech:** Java, Spring Boot, Spring Security, JWT, MySQL, Kafka

### 1. User Registration
- ✅ Register with email and password
- [ ] Register / login via social media (OAuth2 — Google, Facebook)

### 2. User Login & Session Management
- ✅ Login with email and password
- ✅ JWT issuance on successful login
- ✅ Token refresh flow
- ✅ Logout / token invalidation
- [ ] Session expiry after configurable duration

### 3. Profile Management
- ✅ View own profile (`GET /users/me`)
- ✅ Update profile details (name, phone number etc)
<!-- - [ ] Add / update / delete delivery addresses

### 4. Password Reset
- [ ] Request password reset (sends secure link via email)
- [ ] Reset password via token from email link
-->

### 4. Authorization
- ✅ JWT validation (enforced at API Gateway / service level)
- [ ] Role-based access control (USER, ADMIN)

### 5. Kafka Events Published
- ✅ `notification.send.email` → consumed by Notification Service (welcome email)

---

## 📦 Product Service
**Repository:** [AdityaAgrawal-9/productservice](https://github.com/AdityaAgrawal-9/productservice)

Manages the entire product catalog including listings, category hierarchies, and full-text search. MySQL stores structured product data; Elasticsearch powers fast, typo-tolerant search queries.

**Tech:** Java, Spring Boot, MySQL, Elasticsearch

### 1. Product Browsing
- ✅ List products by category (paginated)
- ✅ Detailed product page — images, description, specifications
- ✅ Fast Search using Redis Cache
- [ ] Filter products by category, price range

### 2. Category Management
- ✅ Browse products by category
- [ ] Create / update / delete categories (Admin)

### 3. Search (Elasticsearch)
- [ ] Full-text keyword search across product name and description
<!-- - [ ] Typo correction / fuzzy matching -->
- [ ] Search with filters (price range, category)
<!-- - [ ] Search autocomplete / suggestions -->

### 4. Product Management (Admin)
- ✅ Create new product listing
- ✅ Update product details
- ✅ Delete product

---

## 🛒 Cart Service
**Repository:** [AdityaAgrawal-9/cartservice](https://github.com/AdityaAgrawal-9/cartservice)

Manages the user's shopping cart with flexible document storage in MongoDB and Redis-backed caching for fast retrieval. Produces Kafka events when cart actions occur (e.g. checkout initiated).

**Tech:** Java, Spring Boot, MongoDB, Redis, Kafka

### 1. Cart Operations
- [ ] Add product to cart
- [ ] Remove product from cart
- [ ] Update item quantity
- [ ] View cart — shows items, unit price, quantity, and running total
- [ ] Clear entire cart

### 2. Checkout
- [ ] Specify delivery address at checkout
- [ ] Select payment method at checkout
- [ ] Validate product availability before confirming checkout
- [ ] Trigger order creation on checkout confirmation

### 3. Kafka Events Published
- [ ] `cart.checkout_initiated` → consumed by Order Service

---

## 💳 Payment Service
**Repository:** [AdityaAgrawal-9/paymentservice](https://github.com/AdityaAgrawal-9/paymentservice)

Manages payment gateway integrations, processes transactions, and stores transaction records. Consumes order events from Kafka to trigger payment and publishes payment outcomes back to Kafka for Order and Notification services to act on.

**Tech:** Java, Spring Boot, MySQL, Kafka

### 1. Payment Processing
- [ ] Support credit / debit card payments
- [ ] Support net banking and UPI
- [ ] Initiate payment session with gateway
- [ ] Handle payment success via gateway webhook
- [ ] Handle payment failure via gateway webhook

### 2. Secure Transactions
- [ ] Encrypted payment data handling (PCI-DSS considerations)
- [ ] Idempotent payment requests (prevent duplicate charges)

### 3. Payment Records & Receipts
- [ ] Generate and store payment receipt on success
- [ ] Retrieve payment receipt / transaction details by Order ID
- [ ] View full payment history per user

### 4. Kafka Events
- [ ] Consumes: `order.placed` → initiates payment processing
- [ ] Publishes: `payment.confirmed` → consumed by Order Service
- [ ] Publishes: `notification.send.email` → consumed by Notification Service

---

## 📋 Order Service
**Repository:** [AdityaAgrawal-9/orderservice](https://github.com/AdityaAgrawal-9/orderservice)

Manages the full order lifecycle from creation to delivery. Consumes payment confirmation events from Kafka to confirm orders and publishes order status change events to keep users informed via the Notification Service.

**Tech:** Java, Spring Boot, MySQL, Kafka

### 1. Order Confirmation
- [ ] Create order record on checkout
- [ ] Confirm order upon receiving `payment.confirmed` Kafka event
- [ ] Send order details to user (via Notification Service)

### 2. Order History
- [ ] List all past orders for a user (paginated)
- [ ] Get full details of a specific order by ID

### 3. Order Tracking
- [ ] Track order status: `PENDING → CONFIRMED → PROCESSING → SHIPPED → DELIVERED`
- [ ] Admin: update order status
- [ ] Estimated delivery date

### 4. Kafka Events
- [ ] Consumes: `payment.confirmed` → confirms and activates the order
- [ ] Publishes: `notification.send.email` → consumed by Notification Service on order confirmed.
- [ ] Publishes: `notification.send.email` → consumed by Notification Service on order shipped.
- [ ] Publishes: `notification.send.email` → consumed by Notification Service on order delivered.

---

## 🔔 Notification Service
**Repository:** [AdityaAgrawal-9/notificationservice](https://github.com/AdityaAgrawal-9/notificationservice)

A deliberately thin, single-responsibility service — its only job is to receive a ready-to-send email payload and dispatch it. It subscribes to **one Kafka topic** (`notification.send.email`) and knows nothing about the business events that triggered it.

> **Design Decision — Inverted Responsibility**
> Rather than this service subscribing to domain events like `order.confirmed` or `user.registered` and deciding what email to send, **the responsibility of constructing the email content belongs to the producer**. Each upstream service (Auth, Order, Payment, etc.) builds the full email DTO — `from`, `to`, `subject`, `body` — and publishes it to the shared `notification.send.email` topic. This keeps the Notification Service completely decoupled from business logic and trivially reusable across any service that needs to send an email.

**Tech:** Java, Spring Boot, Kafka Consumer

### 1. Email Dispatch
- ✅ Consume `notification.send.email` event from Kafka
- ✅ Validate the incoming `SendEmailDto` payload
- ✅ Send email using the pre-built `from`, `to`, `subject`, `body` fields
- [ ] Handle delivery failures gracefully (retry / dead-letter)

### 2. Kafka Topic Consumed

| Topic | Payload (DTO) | Published by |
|---|---|---|
| `notification.send.email` | `from`, `to`, `subject`, `body` | Auth, Order, Payment Service (any service) |

### 3. SendEmailDto Contract
```json
{
  "from":    "noreply@ecommerce-store.com",
  "to":      "user@example.com",
  "subject": "Your order #1234 has been confirmed!",
  "body":    "... full email HTML body ..."
}
```

> Each producer service is responsible for populating these fields before publishing. The Notification Service never inspects the content — it just sends what it receives.

---

## 🏗️ Infrastructure & Cross-Cutting Concerns

### Load Balancer — AWS ELB
- [ ] Distribute incoming traffic across service instances
- [ ] Health-check-based instance routing

### API Gateway
- [ ] Single entry point routing requests to appropriate microservices
- [ ] JWT authentication before forwarding requests downstream
- [ ] Rate limiting per client
- [ ] Request / response logging

### Message Broker — Apache Kafka
- [ ] Asynchronous event-driven communication between all services
- [ ] At-least-once delivery guarantee for critical events
- [ ] Event sourcing for order, payment, and user actions

### Caching — Redis
- [ ] Cart data cached in Cart Service for sub-millisecond retrieval
- ✅ Search/Product data cached in Product Service for fast retrieval

### Search — Elasticsearch
- [ ] Full-text product search in Product Service
<!-- - [ ] Fuzzy / typo-tolerant query support -->

<!-- ### Observability
- [ ] Health check endpoints (`/actuator/health`) on all services
- [ ] Centralized structured logging
- [ ] Distributed tracing across services

### DevOps
- [ ] Dockerize all services (`Dockerfile` per service)
- [ ] `docker-compose.yml` for local development stack (MySQL, MongoDB, Redis, Kafka, Elasticsearch)
- [ ] CI/CD pipeline (GitHub Actions)


---

## 🚀 Getting Started (Local Development)

> Prerequisites: Java 17+, Docker, Maven

```bash
# Clone all services
git clone https://github.com/AdityaAgrawal-9/authservice
git clone https://github.com/AdityaAgrawal-9/productservice
git clone https://github.com/AdityaAgrawal-9/cartservice
git clone https://github.com/AdityaAgrawal-9/paymentservice
git clone https://github.com/AdityaAgrawal-9/orderservice
git clone https://github.com/AdityaAgrawal-9/notificationservice

# Spin up infrastructure
docker-compose up -d   # MySQL, MongoDB, Redis, Kafka, Elasticsearch

# Start each service
cd authservice && mvn spring-boot:run
cd productservice && mvn spring-boot:run
# ... repeat for other services
```

---
-->
## 📊 Overall Progress

| Service | Progress |
|---|---|
| 🔐 Auth Service | ████████░░ ~80% |
| 📦 Product Service | █████████░ ~90% |
| 🛒 Cart Service | ░░░░░░░░░░ 0% |
| 💳 Payment Service | ░░░░░░░░░░ 0% |
| 📋 Order Service | ░░░░░░░░░░ 0% |
| 🔔 Notification Service | █████████░ 90% |

---

*Built by [Aditya Agrawal](https://github.com/AdityaAgrawal-9)*
