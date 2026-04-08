---
title: "Push Notifications"
category: system-design
summary: "Push notifications are messages sent from servers to client devices to alert users about new content or events. They enable real-time user engagement even when applications are not actively running."
sources:
  - raw/articles/chat-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-08T19:08:40.645Z
---

# Push Notifications

> Push notifications are messages sent from servers to client devices to alert users about new content or events. They enable real-time user engagement even when applications are not actively running.

# Push Notifications

Push notifications are server-initiated messages delivered to client devices to inform users about new content, events, or updates. They provide a critical communication channel for engaging users even when applications are not actively running.

## Core Components

- **Notification Service**: Server-side system that manages and sends notifications
- **Push Provider**: Platform-specific services (APNs for iOS, FCM for Android)
- **Client Registration**: Devices register for notifications and receive unique tokens
- **Message Delivery**: Reliable delivery mechanism with retry logic

## Delivery Flow

1. **Registration**: Client app registers with push provider and receives device token
2. **Token Storage**: App server stores device token associated with user account
3. **Trigger Event**: Application event triggers notification requirement
4. **Message Creation**: Server constructs notification payload with content and metadata
5. **Provider Delivery**: Server sends notification to push provider with device token
6. **Device Delivery**: Push provider delivers notification to target device

## Platform Providers

- **Apple Push Notification Service (APNs)**: iOS and macOS notifications
- **Firebase Cloud Messaging (FCM)**: Android and cross-platform notifications
- **Windows Push Notification Service (WNS)**: Windows platform notifications
- **Web Push Protocol**: Browser-based notifications for web applications

## Use Cases

**[[Chat System]]**: Notify users about new messages when they're offline or using other applications, ensuring timely communication.

**Social Media**: Alert users about likes, comments, friend requests, and content updates.

**E-commerce**: Send order confirmations, shipping updates, and promotional offers.

**News Apps**: Deliver breaking news and personalized content recommendations.

## Message Types

- **Transactional**: Critical updates like payment confirmations or security alerts
- **Promotional**: Marketing messages and feature announcements
- **Informational**: Status updates and non-urgent notifications
- **Interactive**: Notifications with action buttons for quick responses

## Design Considerations

- **Delivery Reliability**: Implement retry mechanisms and delivery confirmation
- **Rate Limiting**: Prevent notification spam and respect platform limits
- **Personalization**: Target notifications based on user preferences and behavior
- **Timing**: Send notifications at optimal times for user engagement
- **Fallback Mechanisms**: Handle device offline scenarios and delivery failures

## Challenges

- **Platform Differences**: Each platform has unique requirements and limitations
- **Delivery Guarantees**: No guarantee of successful delivery or user interaction
- **Battery Impact**: Excessive notifications can drain device battery
- **User Permissions**: Users can disable notifications, reducing reach
- **Spam Prevention**: Balancing engagement with user experience

Push notifications are essential for maintaining user engagement and providing timely information delivery in modern mobile and web applications.

---
*Related: [[Chat System]], [[Mobile Applications]], [[Real-Time Communication]], [[User Engagement]], [[Message Queue]]*
