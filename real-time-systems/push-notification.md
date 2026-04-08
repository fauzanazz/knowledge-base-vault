---
title: "Push Notification"
category: system-design
summary: "Push notifications are messages sent from servers to client devices to alert users about new content or events, even when the application is not actively running. They are essential for user engagement and real-time communication in mobile and web applications."
sources:
  - raw/articles/chat-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:30:06.896Z
---

# Push Notification

> Push notifications are messages sent from servers to client devices to alert users about new content or events, even when the application is not actively running. They are essential for user engagement and real-time communication in mobile and web applications.

# Push Notification

Push notifications are server-initiated messages sent to client devices to inform users about new content, events, or updates. They enable applications to engage users even when the app is not actively running, making them crucial for user retention and real-time communication.

## Core Components

**Notification Service:**
- Centralized system that manages notification delivery
- Handles message formatting, targeting, and scheduling
- Integrates with platform-specific push services

**Platform Services:**
- **Apple Push Notification Service (APNs)**: iOS devices
- **Firebase Cloud Messaging (FCM)**: Android devices and web browsers
- **Web Push Protocol**: Web browsers across platforms

**Device Registration:**
- Apps register with platform services to receive unique device tokens
- Tokens identify specific app installations on devices
- Tokens must be refreshed periodically

## Notification Flow

1. **Device Registration**: App registers with platform push service and receives device token
2. **Token Storage**: App sends device token to application server
3. **Event Trigger**: Server-side event occurs (new message, update, etc.)
4. **Notification Creation**: Server creates notification payload
5. **Platform Delivery**: Server sends notification to platform service (APNs/FCM)
6. **Device Delivery**: Platform service delivers to target device
7. **User Interaction**: User sees notification and optionally opens app

## Use Cases in Chat Systems

In [[Chat System]] design, push notifications are critical for:

**Offline Messaging:**
- Notify users of new messages when app is closed
- Display message preview and sender information
- Enable quick reply functionality

**Group Chat Updates:**
- Alert users when added to new groups
- Notify about important group announcements
- Show typing indicators for active conversations

**Presence Updates:**
- Inform users when friends come online
- Notify about status changes
- Alert for missed calls or video chat requests

## Message Types

**Alert Notifications:**
- Display text message to user
- Include title, body, and optional media
- Can trigger sound and vibration

**Badge Notifications:**
- Update app icon badge count
- Show number of unread messages
- Clear when user opens app

**Silent Notifications:**
- Update app data in background
- Sync message history
- Refresh user presence status

## Delivery Considerations

**Reliability:**
- Platform services provide best-effort delivery
- Implement retry mechanisms for critical notifications
- Handle delivery failures gracefully

**Timing:**
- Respect user time zones and quiet hours
- Batch non-urgent notifications
- Prioritize real-time messages

**Privacy:**
- Encrypt sensitive notification content
- Allow users to control notification preferences
- Comply with privacy regulations

## Scalability Challenges

**High Volume:**
- Handle millions of notifications per second
- Use [[Message Queue]] for notification processing
- Implement rate limiting to prevent spam

**Targeting:**
- Efficiently query user devices and preferences
- Support complex targeting criteria
- Handle device token updates and cleanup

**Analytics:**
- Track delivery rates and user engagement
- Monitor notification performance metrics
- A/B test notification content and timing

Push notifications are essential for maintaining user engagement and enabling real-time communication in modern applications, requiring careful design for reliability, privacy, and scale.

---
*Related: [[Chat System]], [[Message Queue]], [[Mobile Development]], [[User Engagement]], [[Real-time Systems]], [[Firebase]], [[Apple Push Notification Service]]*
