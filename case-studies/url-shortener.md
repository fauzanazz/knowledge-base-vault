---
title: "URL Shortener"
category: system-design
summary: "A URL shortener is a service that converts long URLs into shorter, more manageable links while maintaining the ability to redirect users to the original destination. It requires careful design for scalability, unique ID generation, and efficient data storage."
sources:
  - raw/articles/url-shortener-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:49:28.864Z
---

# URL Shortener

> A URL shortener is a service that converts long URLs into shorter, more manageable links while maintaining the ability to redirect users to the original destination. It requires careful design for scalability, unique ID generation, and efficient data storage.

# URL Shortener

A URL shortener is a web service that takes long URLs and creates shorter, more manageable versions that redirect to the original destination. Popular examples include TinyURL and bit.ly.

## Core Requirements

- Generate **unique** and **short** URLs
- Handle high traffic volumes (100 million URL generations per day)
- Support efficient read operations with 10:1 read-to-write ratio
- Store massive amounts of data (365 billion records over 10 years)

## API Design

**URL Shortening:**
- Endpoint: `POST api/v1/data/shorten`
- Parameters: `{longUrl: longURLString}`
- Returns: `shortURL`

**URL Redirecting:**
- Endpoint: `GET api/v1/shortUrl`
- Returns: `longURL` for redirection

## Redirection Types

- **301 Redirect:** Permanent redirect cached by browsers, reducing server load but limiting analytics
- **302 Redirect:** Temporary redirect useful for tracking clicks and analytics

## Hash Function Approaches

### Base 62 Conversion
Encodes unique IDs using characters `[0-9, a-z, A-Z]` providing 62 possible characters. A 7-character hash supports up to 3.5 trillion unique URLs.

**Example:** ID `2009215674938` converts to `zn9edcu`

### Hash + Collision Resolution
Uses hash functions like CRC32, MD5, or SHA-1 with collision resolution strategies including Bloom Filters for efficient lookup.

## System Flow

1. Check if `longURL` exists in database
2. If found, return existing `shortURL`
3. Otherwise, generate unique ID using distributed ID generator
4. Convert ID to `shortURL` using Base 62
5. Store mapping in database

For redirection, the system first checks [[Caching]] for faster access before querying the database.

## Scalability Considerations

- **Web Tier:** Stateless design with [[Load Balancer]] for traffic distribution
- **Database Tier:** Uses [[Database Replication]] and [[Database Sharding]]
- **Rate Limiting:** Prevents abuse through request limits per IP
- **Analytics:** Collects click rates, sources, and timestamps for insights

The system requires careful consideration of [[System Scaling]] principles to handle massive traffic volumes efficiently.

---
*Related: [[System Scaling]], [[Load Balancer]], [[Database Replication]], [[Caching]], [[Database Sharding]]*
