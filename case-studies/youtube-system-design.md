---
title: "YouTube System Design"
category: system-design
summary: "YouTube is a massive video streaming platform that requires sophisticated system design to handle billions of users, video uploads, and streaming with high availability and low cost."
sources:
  - raw/articles/youtube-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:47:32.752Z
---

# YouTube System Design

> YouTube is a massive video streaming platform that requires sophisticated system design to handle billions of users, video uploads, and streaming with high availability and low cost.

# YouTube System Design

YouTube is one of the world's largest video streaming platforms, serving 2 billion monthly active users and 5 billion videos watched daily. The system must handle massive scale while maintaining fast uploads, smooth streaming, and cost efficiency.

## Core Requirements

- **Fast video uploads** with resumable capability
- **Smooth video streaming** across devices
- **Dynamic video quality** adaptation
- **Low infrastructure cost** optimization
- **High availability** and reliability

## System Architecture

The YouTube architecture consists of several key components:

- **[[Content Delivery Network]]** for global video distribution
- **API Servers** handling user interactions and metadata
- **Metadata Database** storing video information
- **Original Storage** for uploaded raw videos
- **[[Video Transcoding]]** servers for format conversion
- **Transcoded Storage** for processed videos

## Key Workflows

### Video Upload Flow
Videos are uploaded to blob storage while metadata is updated in parallel. The [[Video Transcoding]] process converts videos into multiple formats and resolutions. Transcoded videos are distributed to the [[Content Delivery Network]] for global access.

### Video Streaming Flow
Videos stream directly from CDN edge servers using protocols like MPEG-DASH, Apple HLS, and Adobe HDS to minimize latency and ensure compatibility across devices.

## Optimizations

**Speed**: Parallel uploads, distributed upload centers, and [[Message Queue]] decoupling
**Cost**: Selective CDN usage, on-demand encoding, and regional distribution
**Security**: Pre-signed URLs, DRM systems, and AES encryption

The system handles both recoverable errors through retries and non-recoverable errors by stopping malformed processing.

---
*Related: [[Video Transcoding]], [[Content Delivery Network]], [[Message Queue]], [[System Scaling]]*
