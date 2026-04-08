---
title: "Actor Model"
category: backend-architecture
summary: "A concurrency model where independent 'actors' communicate exclusively through asynchronous message passing, enabling highly concurrent, fault-tolerant, and distributed systems without shared mutable state."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Actor Model

> A concurrency model where independent 'actors' communicate exclusively through asynchronous message passing, enabling highly concurrent, fault-tolerant, and distributed systems without shared mutable state.

## Overview

The Actor Model was first proposed by Carl Hewitt in 1973. An **actor** is the fundamental unit of computation: it has private state, a mailbox (message queue), and a behavior that processes one message at a time. Actors never share memory — they communicate solely through messages, eliminating entire classes of concurrency bugs (race conditions, deadlocks from shared locks).

This model powers **WhatsApp** (2 million connections per server via Erlang), **Ericsson** telephone switches (99.9999999% uptime), and **Discord** (handling millions of concurrent users via Elixir/Erlang).

## Actor Fundamentals

When an actor receives a message, it can:
1. **Create** new actors
2. **Send** messages to known actors (by address)
3. **Change** its own behavior for the next message (update internal state)

```
Actor A ──[Message: "Compute X"]──► Actor B (mailbox)
                                        ↓ processes sequentially
                                    Actor B ──[Message: "Result X"]──► Actor C
                                    Actor B creates Actor D (sub-task)
```

No locks, no shared memory, no thread contention.

## Erlang/OTP: The Original Implementation

Erlang was designed at Ericsson in the 1980s for telecom switches — requirements: extreme concurrency, fault tolerance, hot code deployment (no downtime for upgrades). Its OTP (Open Telecom Platform) framework defines actor patterns in production.

```erlang
% Erlang actor (gen_server)
-module(counter).
-behaviour(gen_server).

% Initial state
init([]) -> {ok, 0}.

% Handle messages
handle_call(increment, _From, Count) ->
    NewCount = Count + 1,
    {reply, NewCount, NewCount};

handle_call(get, _From, Count) ->
    {reply, Count, Count}.

% Start the actor
start() -> gen_server:start_link({local, ?MODULE}, ?MODULE, [], []).
```

### Supervision Trees
OTP's killer feature: **Supervisor actors** monitor child actors and restart them on failure. This is "let it crash" philosophy — don't write defensive error handling, let the supervisor recover:

```erlang
% Supervisor tree
-module(app_supervisor).
-behaviour(supervisor).

init([]) ->
    Children = [
        {user_session_sup,   % name
         {user_session, start_link, []},
         permanent,           % restart strategy
         5000,                % shutdown timeout
         supervisor,
         [user_session]},
        {payment_worker,
         {payment_worker, start_link, []},
         transient,           % only restart on error
         5000, worker, [payment_worker]}
    ],
    {ok, {{one_for_one, 10, 60}, Children}}.
```

Strategies:
- `one_for_one`: Restart only failed child
- `one_for_all`: Restart all children when one fails
- `rest_for_one`: Restart failed child + all started after it

## Akka (JVM Actor System)

Akka brings the actor model to Java/Scala with typed actors:

```scala
// Akka Typed Actor (Scala)
object Counter {
  sealed trait Command
  case object Increment extends Command
  case class Get(replyTo: ActorRef[Int]) extends Command

  def apply(): Behavior[Command] = counting(0)

  private def counting(count: Int): Behavior[Command] =
    Behaviors.receiveMessage {
      case Increment => counting(count + 1)
      case Get(replyTo) =>
        replyTo ! count
        Behaviors.same
    }
}

// Create and use
val system = ActorSystem(Guardian(), "app")
val counter = system.spawn(Counter(), "counter")

counter ! Counter.Increment
counter ! Counter.Increment
counter.ask(Counter.Get).foreach(println) // prints 2
```

### Akka Cluster
Actors distributed across multiple JVM nodes — actor refs are location transparent:

```scala
// Actor lives on Node A, but addressable from Node B
val remoteActor: ActorRef[Command] =
  context.actorSelection("akka://system@node-b:2552/user/payment-processor")
  // Send messages the same as local actors
remoteActor ! ProcessPayment(orderId, amount)
```

## Elixir and Phoenix: Modern Actor Systems

Elixir runs on the Erlang VM (BEAM), inheriting all OTP primitives with modern syntax:

```elixir
# GenServer (Elixir actor)
defmodule UserSession do
  use GenServer

  # Client API
  def start_link(user_id), do: GenServer.start_link(__MODULE__, user_id)
  def get_cart(pid), do: GenServer.call(pid, :get_cart)
  def add_item(pid, item), do: GenServer.cast(pid, {:add_item, item})

  # Server callbacks
  def init(user_id), do: {:ok, %{user_id: user_id, cart: []}}

  def handle_call(:get_cart, _from, state), do: {:reply, state.cart, state}

  def handle_cast({:add_item, item}, state),
    do: {:noreply, %{state | cart: [item | state.cart]}}
end
```

**Phoenix Channels**: Each WebSocket connection is a lightweight process (actor). Discord serves 11 million concurrent users this way — each guild is an Elixir process with its own state.

## Key Properties

### Location Transparency
Actor addresses work the same whether the actor is local or on a different machine in the cluster. Network distribution becomes a configuration detail.

### Backpressure via Mailbox Bounds
Bounded mailboxes create natural backpressure — if an actor is overloaded, the sender can detect a full mailbox and take action (drop, retry, route elsewhere).

### Hot Code Swapping (Erlang/Elixir)
Replace running actor code without stopping the system:
```elixir
# Load new module version while old actors run
:code.load_file(:my_module)
# Actors pick up new behavior on next message
```

Ericsson's switches run for years without downtime using this feature.

## Real-World Scale

**WhatsApp**: 2 million TCP connections per Erlang server node (2012). 50 engineers served 450 million users at acquisition.

**Discord**: Uses Elixir/Phoenix for real-time messaging. Each Discord guild (server) is a GenServer process. Scaled to millions of concurrent users without sharding at the application level.

**Ericsson AXD301 Switch**: 99.9999999% uptime (31ms downtime/year). Runs entirely on Erlang OTP supervision trees.

**Goldman Sachs**: Uses Akka for high-frequency trading systems — millions of financial messages/second with guaranteed ordering.

## Actor Model vs. Thread-Based Concurrency

| Aspect | Actors | Threads + Locks |
|--------|--------|----------------|
| State sharing | Private — no sharing | Shared — requires synchronization |
| Concurrency bugs | Race conditions impossible | Deadlocks, race conditions common |
| Scalability | Millions of actors per node | Thousands of threads per node |
| Fault tolerance | Supervision trees | Try/catch + restart logic |
| Distribution | Location transparent | Explicit RPC/network calls |
| Learning curve | New mental model | Familiar but error-prone |

## Trade-offs

| Pros | Cons |
|------|------|
| No shared state — no race conditions | Mental model shift required |
| Millions of lightweight actors (vs thousands of threads) | Message serialization overhead |
| Supervision trees provide systematic fault tolerance | Debugging async message flows is harder |
| Location-transparent distribution | Back-pressure and flow control must be explicit |
| Natural fit for real-time, stateful, concurrent systems | Not all problems fit the actor model naturally |

## Anti-Patterns

- **Blocking inside actors**: Never do blocking I/O in an actor — use async messages or a dedicated blocking dispatcher
- **Oversized actors**: One actor per entire domain — defeats isolation; actors should be small and focused
- **Tell instead of ask (overuse of synchronous ask)**: Defeats async benefits; prefer fire-and-forget where possible
- **Mutable shared state**: Passing mutable objects via messages breaks isolation

## When to Use

✅ **Use when**: High concurrency with many independent stateful entities (user sessions, game entities, IoT devices); real-time event processing; systems requiring fault isolation and self-healing; telecom, gaming, chat systems.

❌ **Avoid when**: CPU-bound computation with no concurrency (batch transforms); simple CRUD APIs; teams without Erlang/Akka expertise where the learning curve outweighs benefits.

---
*Related: [[Event-Driven Architecture]], [[Shared Nothing Architecture]], [[Circuit Breaker Pattern]], [[Bulkhead Pattern]]*
