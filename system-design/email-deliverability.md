---
title: "Email Deliverability"
category: system-design
summary: "Email deliverability refers to the ability to successfully deliver emails to recipients' inboxes rather than spam folders. It requires building sender reputation through dedicated IPs, authentication protocols, and careful email classification."
sources:
  - raw/articles/distributed-email-service-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:26:47.963Z
---

# Email Deliverability

> Email deliverability refers to the ability to successfully deliver emails to recipients' inboxes rather than spam folders. It requires building sender reputation through dedicated IPs, authentication protocols, and careful email classification.

# Email Deliverability

Email deliverability is the challenge of ensuring emails reach recipients' inboxes rather than being filtered as spam. Setting up an email server is easy, but achieving good deliverability requires significant domain expertise.

## Core Challenges

New email servers face immediate trust issues with major email providers. Without proper setup, emails from new servers typically end up in spam folders due to aggressive spam-protection algorithms.

## Deliverability Strategies

**IP Management:**
- **Dedicated IPs**: Use dedicated IP addresses for sending emails to build trust
- **IP Warm-up**: Gradually increase sending volume over 2-6 weeks to build reputation
- **Email Classification**: Separate marketing emails from transactional emails using different servers

**Reputation Management:**
- **Quick Spam Banning**: Rapidly identify and ban spam accounts to protect server reputation
- **Feedback Processing**: Establish feedback loops with ISPs to monitor complaint rates
- **Volume Control**: Avoid sudden spikes in email volume that trigger spam filters

**Authentication Protocols:**
- **SPF (Sender Policy Framework)**: Validates sending server authorization
- **DKIM (DomainKeys Identified Mail)**: Cryptographically signs emails
- **DMARC**: Provides policy framework for email authentication

## Monitoring and Maintenance

Successful deliverability requires continuous monitoring of:
- Bounce rates and complaint rates
- Blacklist status across major providers
- Authentication record validity
- Sending reputation scores

## Impact on System Design

Deliverability requirements influence distributed email system architecture by necessitating separate infrastructure for different email types, sophisticated monitoring systems, and gradual scaling approaches rather than rapid deployment.

Building reliable email deliverability requires deep understanding of email ecosystem politics and technical standards.

---
*Related: [[Distributed Email Service]], [[System Scaling]], [[Multi-Data Center Setup]]*
