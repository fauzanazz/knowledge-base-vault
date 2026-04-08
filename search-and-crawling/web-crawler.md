---
title: "Web Crawler"
category: system-design
summary: "A web crawler is a system that systematically discovers and collects web content for applications like search engine indexing. It must balance scalability, robustness, politeness, and extensibility to handle billions of pages effectively."
sources:
  - raw/articles/_done/web-crawler-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:53:13.398Z
---

# Web Crawler

> A web crawler is a system that systematically discovers and collects web content for applications like search engine indexing. It must balance scalability, robustness, politeness, and extensibility to handle billions of pages effectively.

# Web Crawler

A **web crawler** (also known as a spider or robot) is a system that systematically discovers and collects web content such as web pages, images, and videos. Web crawlers are essential components of search engines, web archiving systems, and data mining applications.

## Applications

- **Search Engine Indexing**: Collect web pages to create searchable indexes (e.g., Googlebot)
- **Web Archiving**: Preserve web data for future use (e.g., US Library of Congress)
- **Web Mining**: Extract knowledge from web data for analysis
- **Web Monitoring**: Detect copyright or trademark infringements

## Design Requirements

A scalable web crawler must handle:
- **1 billion pages per month** (400 pages/second, peak 800 QPS)
- **HTML-only content** collection
- Duplicate content detection and avoidance
- **5-year storage** requirement (~30 PB)

## Core Components

### URL Management
- **Seed URLs**: Starting points categorized by locality or topic
- **[[URL Frontier]]**: FIFO queue storing URLs to be downloaded
- **URL Filter**: Excludes blacklisted or erroneous URLs
- **URL Seen**: Tracks visited URLs to avoid duplication

### Content Processing
- **HTML Downloader**: Downloads web pages with DNS resolution
- **Content Parser**: Validates and parses web pages
- **Content Seen**: Detects duplicate content using hash comparisons
- **URL Extractor**: Extracts new links from parsed pages

## Key Design Principles

### Politeness
Implemented through the [[URL Frontier]] to ensure only one request per host at a time, preventing server overload. Uses hostname-to-queue mapping with worker threads and configurable delays.

### Priority Management
Assigns higher priority to important pages based on PageRank or update frequency. Uses front queues for prioritization and back queues for politeness.

### Robustness
- **[[Consistent Hashing]]** for load distribution
- Error handling to prevent system crashes
- Data validation for content integrity
- Spider trap avoidance using URL length limits

### Performance Optimizations
- **Distributed crawling** using [[Horizontal Scaling]]
- **DNS [[Caching]]** to avoid repeated lookups
- Geographic distribution of crawl servers
- Short timeouts for unresponsive servers

Web crawlers use BFS traversal rather than DFS due to the potentially infinite depth of web graphs, ensuring comprehensive coverage while maintaining system stability.

---
*Related: [[URL Frontier]], [[Consistent Hashing]], [[Horizontal Scaling]], [[Caching]], [[System Scaling]]*
