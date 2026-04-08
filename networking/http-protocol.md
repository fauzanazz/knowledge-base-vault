---
title: "HTTP Protocol"
category: networking
summary: "HTTP is a stateless request-response protocol for client-server communication that has evolved from HTTP/1.1 through HTTP/2 to HTTP/3, each addressing performance and reliability limitations."
sources:
  - raw/articles/communication-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:17:50.259Z
---

# HTTP Protocol

> HTTP is a stateless request-response protocol for client-server communication that has evolved from HTTP/1.1 through HTTP/2 to HTTP/3, each addressing performance and reliability limitations.

# HTTP Protocol

HTTP (Hypertext Transfer Protocol) is a stateless request-response protocol designed for client-server communication. It operates over TCP with optional TLS encryption (HTTPS).

## Message Structure

HTTP messages consist of three parts:
- **Start line**: Indicates request purpose or response status
- **Headers**: Key-value pairs containing message metadata
- **Body**: Optional container for data payload

## Protocol Evolution

**HTTP/1.1**: Text-based protocol that keeps connections open between requests but requires sequential request processing, creating head-of-line blocking.

**HTTP/2**: Binary protocol enabling multiplexed concurrent request-response transactions on single connections, eliminating the need for multiple connections to fetch independent resources.

**HTTP/3**: UDP-based protocol with custom reliability mechanisms. Packet loss only blocks affected streams rather than all streams, addressing TCP's head-of-line blocking limitation.

## Request Methods

Common HTTP methods with safety and idempotency properties:
- **GET**: Safe and idempotent - retrieves resources without side effects
- **POST**: Neither safe nor idempotent - creates resources
- **PUT**: Idempotent but not safe - updates resources
- **DELETE**: Idempotent but not safe - removes resources

## Response Status Codes

- **2xx**: Success (200 OK)
- **3xx**: Redirection (301 Moved Permanently)
- **4xx**: Client errors (400 Bad Request, 404 Not Found)
- **5xx**: Server errors (500 Internal Server Error, 503 Service Unavailable)

## Stateless Design

HTTP's stateless nature requires all client information to be included in requests. Servers treat requests independently unless additional data indicates client association.

HTTP serves as the foundation for [[REST APIs]] and modern web communication in [[Distributed Systems]].

---
*Related: [[Network Communication]], [[REST APIs]], [[TCP Protocol]], [[TLS Security]], [[Web Protocols]]*
