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
