# Class Diagram – Notification System

```mermaid
classDiagram
    class Notifier {
        <<interface>>
        +send(message: String)
    }

    class EmailNotifier {
        +send(message: String)
    }

    class SMSNotifier {
        +send(message: String)
    }

    class PushNotifier {
        +send(message: String)
    }

    class NotifierFactory {
        +createNotifier(type: String) Notifier
    }

    class NotificationSystem {
        +main(args: String[])
    }

    Notifier <|.. EmailNotifier
    Notifier <|.. SMSNotifier
    Notifier <|.. PushNotifier
    NotifierFactory --> Notifier
    NotificationSystem --> NotifierFactory
