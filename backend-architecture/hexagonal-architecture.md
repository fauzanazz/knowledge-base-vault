---
title: "Hexagonal Architecture"
category: backend-architecture
summary: "An architectural pattern (Ports & Adapters) by Alistair Cockburn that isolates the application core from external systems through explicitly defined ports and interchangeable adapters."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Hexagonal Architecture

> An architectural pattern (Ports & Adapters) by Alistair Cockburn that isolates the application core from external systems through explicitly defined ports and interchangeable adapters.

## Overview

Hexagonal Architecture — formally called **Ports & Adapters** — was coined by Alistair Cockburn in 2005. The central idea: the application has a hexagonal "core" surrounded by **ports** (abstract interfaces) and **adapters** (concrete implementations). Any external system — HTTP, databases, message queues, CLIs — interacts with the core only through these ports.

The hexagon shape has no geometric significance; it symbolizes *"there are many sides to plug into."*

## Core Concepts

### Application Core
The domain logic and use cases. Has zero knowledge of HTTP, SQL, Kafka, or any infrastructure technology. Contains:
- **Domain entities**: Business objects with behavior
- **Application services**: Orchestration of domain logic
- **Domain services**: Cross-entity business rules

### Ports
**Interfaces** declared by the core that define how it communicates with the outside world. Two types:

- **Primary (Driving) Ports**: How external actors *drive* the application. Example: `OrderService` interface called by HTTP controllers or CLI tools.
- **Secondary (Driven) Ports**: How the application *drives* external systems. Example: `OrderRepository` interface that the database adapter implements.

### Adapters
**Concrete implementations** of ports. Adapters translate between the outside world's format and the core's format.

- **Primary Adapters**: REST controller, GraphQL resolver, gRPC handler, CLI command
- **Secondary Adapters**: PostgreSQL repository, Redis cache, Kafka producer, SMTP email sender

```
        [HTTP Adapter] ──► [Primary Port] ──► CORE ──► [Secondary Port] ──► [DB Adapter]
        [CLI Adapter]  ──►                           ──►                ──► [Kafka Adapter]
        [gRPC Adapter] ──►                           ──►                ──► [Mock Adapter (test)]
```

## Real-World Example: Spotify

Spotify's backend services use hexagonal principles so that the same playlist management core runs identically whether driven by the mobile REST API, internal gRPC calls, or during integration tests with in-memory adapters. When they migrated from Cassandra to a newer storage layer, only the secondary adapter changed — not a line of domain logic.

## Implementation Example (Java/Spring)

```java
// Secondary Port (defined in core)
public interface MusicRepository {
    Optional<Track> findById(TrackId id);
    void save(Track track);
}

// Application Service (core)
public class PlaylistService {
    private final MusicRepository musicRepo;
    private final PlaylistRepository playlistRepo;

    public void addTrackToPlaylist(PlaylistId playlistId, TrackId trackId) {
        Track track = musicRepo.findById(trackId)
            .orElseThrow(() -> new TrackNotFoundException(trackId));
        Playlist playlist = playlistRepo.findById(playlistId)
            .orElseThrow(() -> new PlaylistNotFoundException(playlistId));
        playlist.addTrack(track);
        playlistRepo.save(playlist);
    }
}

// Primary Adapter (REST — lives OUTSIDE core)
@RestController
public class PlaylistController {
    private final PlaylistService service;

    @PostMapping("/playlists/{id}/tracks")
    public ResponseEntity<Void> addTrack(@PathVariable String id, @RequestBody AddTrackRequest req) {
        service.addTrackToPlaylist(new PlaylistId(id), new TrackId(req.trackId()));
        return ResponseEntity.noContent().build();
    }
}

// Secondary Adapter (JPA — lives OUTSIDE core)
@Repository
public class JpaMusicRepository implements MusicRepository {
    @Override
    public Optional<Track> findById(TrackId id) { /* JPA query */ }
    @Override
    public void save(Track track) { /* JPA save */ }
}
```

## Testing Strategy

The isolation enables three testing tiers:
1. **Unit tests**: Test core logic with no adapters — use in-memory fakes that implement secondary ports
2. **Integration tests**: Test individual adapters against real infrastructure (Testcontainers)
3. **E2E tests**: Drive through primary adapters end-to-end

```java
// Fast unit test — no Spring context, no DB
class PlaylistServiceTest {
    MusicRepository musicRepo = new InMemoryMusicRepository(); // fake adapter
    PlaylistService service = new PlaylistService(musicRepo, new InMemoryPlaylistRepository());

    @Test
    void addTrack_succeeds_whenTrackExists() { /* ... */ }
}
```

## Hexagonal vs. Clean Architecture

| Aspect | Hexagonal | Clean Architecture |
|--------|-----------|-------------------|
| Origin | Alistair Cockburn, 2005 | Robert C. Martin, 2012 |
| Layers | No strict layering — just core + adapters | Four explicit concentric layers |
| Primary/Secondary distinction | Explicit | Implicit via direction |
| Detail level | High-level concept | Prescriptive structure |

They share the same core principle (dependency inversion) and are often used together.

## Trade-offs

| Pros | Cons |
|------|------|
| Easily swap infrastructure (DB, queue, framework) | Boilerplate: interfaces + adapters + mappers |
| Excellent testability with fake adapters | Can feel over-engineered for simple services |
| Clear boundaries prevent infrastructure bleed | Requires discipline to not shortcut |
| Supports multiple primary adapters simultaneously | Port/adapter proliferation in large systems |

## Anti-Patterns

- **Leaking JPA annotations into domain entities** — coupling core to a secondary adapter
- **Business logic in adapters** — adapters must only translate, never decide
- **Skipping primary ports** — calling application services directly from controllers without an interface removes flexibility

## When to Use

✅ **Use when**: Services with multiple delivery mechanisms (REST + gRPC + CLI); teams that want fast unit test suites; systems that may swap databases or brokers.

❌ **Avoid when**: Tiny single-purpose services, prototypes, or scripts where abstraction overhead exceeds the benefit.

---
*Related: [[Clean Architecture]], [[Domain-Driven Design]], [[Modular Monolith]], [[API Gateway Pattern]]*
