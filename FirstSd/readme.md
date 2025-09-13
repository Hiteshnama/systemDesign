# Notification System – System Design

## 1) One-line Summary
A system to send notifications (Email, SMS, Push). It must be extensible, concurrent, and handle multiple channels.

---

## 2) Goals & Non-goals
**Goals**
- Support multiple notification channels (Email, SMS, Push).
- Allow concurrent processing of notifications.
- Easy to extend with new channels (e.g., WhatsApp, Slack).

**Non-goals**
- Real integration with SMS/email providers (mocked for demo).
- Rich UI / scheduling system.
- Delivery guarantees like retry/DLQ (not part of MVP).

---

## 3) High-Level Design (HLD)
**Components**
1. **Client (User Input)** – enters notification type & message.  
2. **Notification Service** – receives request, validates, submits tasks.  
3. **Notifier Factory** – creates correct notifier instance.  
4. **Notifiers** – strategies (Email, SMS, Push).  
5. **ExecutorService** – thread pool for concurrent dispatch.  

**Flow**

---

## 4) API + Interactions
For a real system (beyond console input), you could expose:  

- `POST /notifications`  
  **Body:** `{ "type": "email", "message": "Hello User" }`  
  **Response:** `202 Accepted`  

---

## 5) Data Model / Schema & DB choice
- MVP: No DB, everything runs in memory.  
- Real-world:  
  - **Table: notifications**  
    ```
    id (PK), type, message, status, created_at, updated_at
    ```  
  - **Choice:** Relational DB (Postgres/MySQL) for transactions.  
  - **Index:** `(status, created_at)` for querying pending notifications.  

---

## 6) Scalability & Capacity Planning
- Assume **1M notifications/day** (~12 QPS).  
- Each notification ~500B → ~500MB/day storage.  
- Horizontal scaling with stateless Notification Service.  
- Thread pool per instance (size = 2×CPU cores).  
- Cache recent templates in Redis.  
- For massive scale: push tasks to **Kafka / RabbitMQ**, workers consume and send.

---

## 7) Concurrency, Locking & Consistency
- **In-memory MVP:** No shared resource contention.  
- **At scale:**  
  - Use **DB transactions** to mark notification as `in_progress → sent`.  
  - Prevent duplicate delivery via **idempotency key**.  
  - Locking not required in this simple system, but would be in others (e.g., ticket booking).

---

## 8) Reliability / Observability
- Log each send attempt.  
- Retry failed sends with exponential backoff.  
- Dead-letter queue for permanent failures.  
- Metrics: notification count, failure rate, avg latency.

---

## 9) Security & Privacy
- Encrypt sensitive user data (phone/email) at rest.  
- TLS for data in transit.  
- Authentication/authorization around API.  
- Rate limiting to avoid abuse.

---

## 10) Low-Level Design (LLD) – Java Code
See [NotificationSystem.java](./NotificationSystem.java) for runnable implementation.

---

## 11) Class Relationships (Code Structure)
- `Notifier` is an **interface** with the method `send(String message)`.  
- `EmailNotifier`, `SMSNotifier`, and `PushNotifier` are **concrete implementations** of `Notifier`.  
- `NotifierFactory` follows the **Factory Pattern**, returning the appropriate `Notifier` based on type.  
- `NotificationSystem` (main class) accepts user input, uses the factory, and delegates to the selected `Notifier`.  
- `ExecutorService` ensures that multiple notifications are processed concurrently.

See [class-diagram.md](./class-diagram.md) for UML diagram.
