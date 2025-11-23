# Distributed Tracing Fundamentals:

## Why Distributed Tracing?
## Monolith (Single Server) - Easy to Debug
When everything is in one server, you get a stack trace:

eg.

User clicks → Main()

 → handleRequest()
    
  → validateUser()
        
  → queryDatabase()
        
  → formatResponse()
        
 → Return response
    
If something breaks, the stack trace shows the exact path.

## Microservices - Complicated to Debug
With 100 services, a single user request travels like this:

eg.
API Gateway 

→ Auth Service 

→ Payment Service 

→ Database

→ Notification Service

→ Cache

→ Another Service

...
Problem: Each service logs independently. Finding the request path is like finding a needle in a haystack.

Solution: Distributed tracing - a trace ID that follows the request through all services, stitching together the entire journey.

## Core Concepts

### Trace: The Entire Journey

A trace shows one complete request from start to finish, passing through multiple services.

User Request → [travels through 5 services] → Response

All of this is one trace. It has ONE unique Trace ID that connects everything.

Trace ID: 5b8efff798038103d269b633813fc60c

This trace ID appears in:
- Service A logs
- Service B logs
- Service C logs
- Database logs
- Every service that touched this request

### Span: A Single Operation
A span is ONE piece of work within a trace. Every operation is a span.

Examples of spans:

- HTTP request to another service

- Database query

- Cache lookup

- Function call

- External API call

One trace = many spans.

eg.

User Request (1 trace)

Span 1: Handle HTTP request (API Gateway)

Span 2: Check user auth (Auth Service)

Span 3: Query database (DB Service)

Span 4: Send notification (Notification Service)

All 4 spans belong to the SAME trace (same Trace ID)

### Trace ID: The Glue
Trace ID = unique identifier that joins all spans together.

Example:

Trace ID: 5b8efff798038103d269b633813fc60c

Service A creates span with this Trace ID

Service A sends request to Service B, includes Trace ID

Service B creates span with same Trace ID

Service B sends request to Service C, includes Trace ID

Service C creates span with same Trace ID

Result: All spans connected by same Trace ID

### Span ID: Individual Identity
Span ID = unique identifier for a specific span.

Example:
Trace ID: 5b8efff798038103d269b633813fc60c (same for all)

Service A Span ID: eee19b7ec3c1b173 (unique to this span)

Service B Span ID: eee19b7ec3c1b174 (unique to this span)

Service C Span ID: eee19b7ec3c1b175 (unique to this span)

Each span has different ID, but same Trace ID

### Parent-Child Relationships
Spans form a tree structure. One span can be the parent of other spans.

Example:

Root Span (API Gateway)

├─ Child Span (Call Auth Service)

├─ Child Span (Call Payment Service)

│  ├─ Grandchild Span (DB Query)

│  └─ Grandchild Span (External API Call)

└─ Child Span (Send Notification)

This tree shows the call hierarchy

How it works:

Example:

Service A (Root/Parent):

- Trace ID: abc123
- Span ID: span-001 (parent)

Service A calls Service B:

- Trace ID: abc123 (same!)
- Span ID: span-002
- Parent Span ID: span-001 (links to Service A)

Service B calls Service C:

- Trace ID: abc123 (same!)
- Span ID: span-003
- Parent Span ID: span-002 (links to Service B)

Result: The tree shows: A → B → C

### Attributes/Tags: Metadata on Spans
Attributes = extra information attached to a span.

Examples:

HTTP Request Span:

- http.method: "GET"
- http.status_code: 200
- http.url: "/api/users/123"

Database Query Span:

- db.system: "postgresql"
- db.statement: "SELECT * FROM users WHERE id = ?"
- db.rows_affected: 1

Error Span:

- error: true
- error.message: "Connection timeout"
- error.stacktrace: "[...]"

Attributes help you understand what happened without reading logs.  

### Events: Point-in-Time Markers
Events = "something happened at this exact moment" within a span.

Examples:

Database Query Span:

├─ 0ms: Event "Query started"

├─ 50ms: Event "Results received" 

├─ 51ms: Event "Processing started"

└─ 100ms: Span ends

User sees: Specific moments in time where things happened

### Span Status: Success or Failure
Every span has a status:

text
- OK: Operation succeeded
- ERROR: Operation failed
- UNSET: Not specified (default)
  
Example:

Span 1: Database Query → Status: OK 

Span 2: Auth Check → Status: ERROR 

Span 3: Notification → Status: OK 

User can immediately see: Auth check failed

Example of span structure:

```
{
  "traceId": "5b8efff798038103d269b633813fc60c",
  "spanId": "eee19b7ec3c1b174",
  "parentSpanId": "eee19b7ec3c1b173",
  "name": "GET /api/users/123",
  "kind": "SERVER",
  "startTime": "2025-11-22T10:30:00.000Z",
  "endTime": "2025-11-22T10:30:00.250Z",
  "duration": "250ms",
  "status": "OK",
  "attributes": {
    "http.method": "GET",
    "http.url": "/api/users/123",
    "http.status_code": 200,
    "service.name": "api-gateway",
    "service.version": "1.2.0"
  },
  "events": [
    {
      "timestamp": "2025-11-22T10:30:00.050Z",
      "name": "Auth check completed"
    },
    {
      "timestamp": "2025-11-22T10:30:00.150Z",
      "name": "Database query completed"
    }
  ]
}
```
## Trace Context Propagation: The Magic

### How does Service A tell Service B "this request belongs to trace abc123"?
Through HTTP headers (or gRPC metadata, or message queue headers).

### W3C Trace Context Standard
The standard way to transfer traces uses two HTTP headers:

Header 1: traceparent

Example: 

traceparent: 00-0af7651916cd43dd8448eb211c80319c-b7ad6b7169203331-01

Breaking this down:

Example:

Format: version-traceid-parentid-traceflags

00                          = Version (0 = current)

0af7651916cd43dd8448eb211c80319c = Trace ID (unique for entire request)

b7ad6b7169203331               = Parent Span ID (which span in parent service)

01                              = Trace Flags (01 = sample this trace)

Plain English:

"Hello Service B, this request is part of trace 0af7651916..., and the parent span is b7ad6b7169.... Please keep tracing it (flag = 01 means yes)."

Header 2: tracestate (Optional)
Example:

tracestate: congo=t61rcWkgMzE

Purpose: Let vendors add custom data.

Example:

text
tracestate: datadog=t61rcWkgMzE, my-vendor=custom-value

### B3 propogation:
B3 propagation is another tracing format used mostly by Zipkin, Spring Cloud Sleuth, Istio, and older microservice systems.

It serves the same purpose as W3C Trace Context:
to pass tracing information between services
but the format is different.

### How Context Propagation Works (Step by Step)
Scenario: User Request Through 3 Services

Step 1: User → Service A

Example:
```
User sends: GET /api/users/123

Service A receives request (no headers, so create new trace):
- Creates Trace ID: abc123xyz
- Creates Span ID: span-001
- Span status: SERVER

Service A adds to HTTP response headers:
traceparent: 00-abc123xyz-span-001-01
tracestate: (empty)
```

Step 2: Service A → Service B
```
Service A needs to call Service B
It adds its context to the outgoing request:

HTTP Request headers:
traceparent: 00-abc123xyz-span-001-01
tracestate: (vendor data)

Service B receives request with headers:

Service B extracts headers:
- Trace ID: abc123xyz (same as parent!)
- Parent Span ID: span-001 (knows who called it)

Service B creates new span:
- Trace ID: abc123xyz (same trace)
- Span ID: span-002 (new ID for this span)
- Parent Span ID: span-001 (links to Service A)
- Span status: CLIENT (it's calling another service)
```

Step 3: Service B → Service C
```
Service B calls Service C:

HTTP Request headers:
traceparent: 00-abc123xyz-span-002-01
tracestate: (vendor data)

Service C extracts and creates:
- Trace ID: abc123xyz (same entire trace!)
- Span ID: span-003
- Parent Span ID: span-002 (knows who called it)
```

Result:

```
Full trace visible:
Trace ID: abc123xyz

Service A
├─ Span: span-001 (SERVER, 250ms)
   └─ Parent: none (root span)

Service B  
├─ Span: span-002 (CLIENT, 100ms)
   └─ Parent: span-001 (Service A called us)

Service C
├─ Span: span-003 (CLIENT, 50ms)
   └─ Parent: span-002 (Service B called us)

Timeline shows:
[Service A]
    └─ [Service B]
        └─ [Service C]
```

### Context Propagation in Different Protocols
#### HTTP Headers (REST APIs)
```
GET /api/users/123 HTTP/1.1
Host: api.example.com
traceparent: 00-0af7651916cd43dd8448eb211c80319c-b7ad6b7169203331-01
tracestate: vendor=value

(Request body...)
```
#### gRPC Metadata
gRPC uses metadata (works like HTTP headers):

```
Service A → Service B (gRPC call)

Metadata:
- traceparent: 00-abc123xyz-span-001-01
- tracestate: vendor-value

(gRPC proto message...)
```

#### Message Queues (Kafka, RabbitMQ)

For async messaging, context goes in message headers:

```
Kafka Message:
Headers:
{
  "traceparent": "00-abc123xyz-span-001-01",
  "tracestate": "vendor=value"
}

Body:
{
  "userId": 123,
  "action": "user_created"
}
```
#### Baggage: Carrying Extra Context

Baggage = application-specific data that travels with the request through all services.

When You'd Use Baggage

```
Instead of: Each service manually passes user_id, tenant_id, feature_flag

You can: Put them in baggage header, all services can read

Baggage header (W3C standard):
baggage: userId=12345, tenantId=acme, feature_flag=new_checkout

All services in the trace can access:
- User ID: 12345
- Tenant ID: acme
- Feature flag: new_checkout

Without explicitly passing these as parameters
```

Careful with Baggage
Warning: Baggage increases payload size.

```
Small baggage: OK
  baggage: userId=123, tenantId=456

Huge baggage: Not OK (wastes bandwidth)
  baggage: userId=123, full_user_object={"name":..., "email":..., "history":[...]}

Rule of thumb: Only put small, essential data in baggage
```
## Span Structure 

### Span name

A short, human‑readable label for **what this span is doing**.

Examples:
- `"GET /api/users/{id}"` – handling an HTTP request
- `"SELECT user by id"` – database query
- `"call-payment-service"` – outgoing HTTP call

Think of it like a function name or log title that tells you: **“this span represents THIS action.”**

---

### Span kind (client, server, internal)

This tells you **what role** the span is playing:

- **SERVER** – handling an incoming request  
  - Example: Your API service receiving an HTTP request from a client.
- **CLIENT** – making an outgoing request  
  - Example: Your service calling another service or a database.
- **INTERNAL** – work happening inside a service  
  - Example: A chunk of business logic that doesn’t call another service.

This helps you see the **direction**: who is the **caller** (client) and who is the **callee** (server).

---

### Start time, end time, duration

These tell you **when** the span ran and **how long it took**.

- **Start time:** When the work started (e.g., `10:30:00.100Z`)
- **End time:** When the work finished (e.g., `10:30:00.350Z`)
- **Duration:** `End − Start` (e.g., `250ms`)

Why it matters:
- Helps you see **which part of a request is slow**.
- Lets trace UIs draw those **Gantt/flame charts** with bars over time.

---

### Status (ok, error)

This says **whether the operation succeeded or failed**.

Common values:
- **OK / UNSET** – no error
- **ERROR** – something went wrong (exception, timeout, HTTP 500, etc.)

If a span is **ERROR**:
- Trace UIs usually color it **red**.
- You can quickly scan a trace and **spot where the problem happened**.

---

### Attributes (tags)

Key–value data attached to the span that gives **extra context**.

Examples for an HTTP span:
- `http.method = "GET"`
- `http.route = "/api/users/{id}"`
- `http.status_code = 200`
- `user.id = "12345"`
- `service.name = "user-service"`

Examples for a DB span:
- `db.system = "postgresql"`
- `db.statement = "SELECT * FROM users WHERE id = ?"`
- `db.rows_affected = 1`

Use attributes to answer: **“what exactly was this span doing?”**

---

### Events

Small **timestamped notes inside a span** – like mini log entries attached to that span.

Examples:
- `"cache_miss"` at `20ms`
- `"retrying_request"` at `100ms`
- `"payment_approved"` at `150ms`

They help you see **important moments** within that one operation **without scanning all global logs**.

---

### Parent span ID

This tells you **which span started or caused this span**.

- If a span has **no parent**, it’s usually the **root span** of the trace (the first thing that happened).
- If a span has a **parent span ID**, it’s a **child span**.

Example tree:

Span A (ID = 1, no parent) ← root

├─ Span B (ID = 2, parent = 1)

│ └─ Span C (ID = 3, parent = 2)

└─ Span D (ID = 4, parent = 1)

This is how the tracing system can draw the call tree:

- A called **B** and **D**.  
- **B** called **C**.

# 2. OpenTelemetry (OTel) Deep Dive


## 1. What Is OpenTelemetry (OTel)?

OpenTelemetry is like a **standard toolbox** for observability.

It gives you:
- A **common way to collect**:
  - Metrics (numbers)
  - Logs (text messages)
  - Traces (request journeys)
- **APIs + SDKs** for many languages (Java, Go, Node, Python, etc.).
- A **Collector** that receives data and sends it to any backend (Jaeger, Tempo, Datadog, New Relic, etc.).

Key idea: **Vendor-neutral** – you instrument your app once, and you can switch between backends later without changing your code.

It effectively **replaces** older projects:
- OpenTracing
- OpenCensus

Those are now merged into OpenTelemetry.

---

## 2. OpenTelemetry Architecture (Big Picture)

Think of OTel in **three main parts**:

1. **OTel API** – The *interface* your code uses
   - Defines what a span, trace, metric, and log look like.
   - Functions like `startSpan()`, `setAttribute()`, etc.

2. **OTel SDK** – The *implementation* behind the API
   - Decides how data is processed.
   - Handles sampling, batching, exporting.
   - Includes exporters to send data to backends.

3. **OTel Collector** – The *middleman service*
   - Runs as a separate process/pod.
   - Receives data from apps (via OTLP, Jaeger, Zipkin, Prometheus, etc.).
   - Processes it (filter, sample, add tags).
   - Exports it to multiple backends.

### Simple Architecture Diagram (Text)

```text
Your App (instrumented with OTel API + SDK)
   ↓  (OTLP / HTTP / gRPC)
OpenTelemetry Collector (Receivers → Processors → Exporters)
   ↓
Observability Backends (Jaeger, Tempo, Prometheus, Datadog, New Relic, ...)
```

You can:
- Export **directly from SDK to backend**, or
- Go **via Collector**, which is recommended for production.

---

## 3. OTLP (OpenTelemetry Protocol)

**OTLP** is the standard format and protocol that OTel uses to send telemetry between SDKs, Collectors, and backends.

- **Transport:** HTTP or gRPC
- **Encoding:** Protobuf (binary, efficient) or JSON
- **Data types:** Traces, Metrics, Logs

### Trace Data Structure (Simplified)

In OTLP, traces are usually organized like:

```text
ResourceSpans
  ├─ Resource (service.name, env, region, etc.)
  └─ ScopeSpans (previously InstrumentationLibrarySpans)
       ├─ Scope (which library created spans)
       └─ Spans[] (actual span objects)
```

A **Span** in OTLP contains:
- `trace_id`
- `span_id`
- `parent_span_id`
- `name`
- `kind`
- `start_time`, `end_time`
- `attributes[]`
- `events[]`
- `status`

You normally don’t build this manually; the SDK does that for you.

How is trace data serialized ?

OpenTelemetry serializes trace data using Protocol Buffers (Protobuf) — a compact, efficient, binary serialization format.

Why Protobuf?

- Small size

- Fast encoding/decoding

- Schema-defined (strong typing)

- Cross-language support
---

## 4. Instrumentation – How to Add OTel to an App

There are **two main ways** to instrument an app:

1. **Auto-instrumentation (zero/low code)**
2. **Manual instrumentation (explicit code)**

### 4.1 Auto-Instrumentation (Magic)

**Goal:** Get traces/metrics/logs **without changing your app code**.

How it works (high-level):
- You attach an **agent** (Java `-javaagent`, Python `opentelemetry-instrument`, Node hooks, etc.).
- The agent:
  - Hooks into the runtime (JVM, Node, Python, etc.).
  - Uses techniques like **bytecode manipulation** (Java), **monkey patching** (Python/Node), or **runtime hooks**.
  - Automatically instruments common libraries:
    - HTTP servers/clients (Spring, Express, Flask, FastAPI, etc.)
    - Database clients
    - Messaging (Kafka, RabbitMQ)

No code changes; the agent auto-wraps frameworks.

Bytecode manipulation means:

Modifying the compiled Java classes at runtime (while the app is running)
without changing your source code.

Java compiles .java → .class → JVM executes bytecode.
The OpenTelemetry Java Agent modifies this bytecode to insert tracing logic.

Why?

So it can automatically:

- start spans for HTTP requests

- trace JDBC queries

- add context propagation

- measure execution time

without touching your actual code.

How it works:

- JVM loads a class (e.g., HttpClient.class)

- Agent intercepts this step

- It injects extra lines like:

```
startSpan("HTTP request");
callOriginalMethod();
endSpan();
```

Tools used:

- ByteBuddy (used by OpenTelemetry)

- ASM

- Javassist

**Pros:**
- Very fast to enable.
- No or minimal code changes.
- Great for legacy services.

**Cons:**
- Limited control – traces what the agent knows about.
- Harder to add **business context** (user IDs, order IDs, etc.).
- Might miss custom logic or internal functions.

Good for: **initial rollout**, legacy apps, broad coverage.

---

### 4.2 Manual Instrumentation

You **write code** using the OTel SDK to create spans, add attributes, etc.

**Pros:**
- Full control.
- Can add rich business attributes.
- You choose which spans matter.

**Cons:**
- More coding effort.
- Need to understand tracing concepts.
- Easier to forget to instrument some paths.

**Best practice:**
- Use **auto-instrumentation for baseline coverage**.
- Add **manual spans for critical business flows** (checkout, payment, signup, etc.).

---

## 5. How to Instrument an Application (Step-by-Step)

Example: You have a simple HTTP API service and want traces + metrics.

### Step 1 – Decide Architecture

- For local/dev: SDK → backend directly.
- For production: SDK → OTel Collector → backend.

### Step 2 – Install OTel SDK + Instrumentation

### Step 3 – Configure SDK

- Set **service name**, environment, etc.
- Configure **exporter** to send data (OTLP to Collector or vendor).[289][291]

### Step 4 – Enable Auto-Instrumentation (if available)

- For frameworks like Flask, Django, Express, Spring Boot, etc.
- This auto-captures HTTP spans, DB calls, etc.

### Step 5 – Add Manual Spans for Key Business Logic

- Around checkout flows, payments, complex jobs.
- Add attributes like `tenant_id`, `order_id`, etc.

### Step 6 – Run and Verify

- Start OTel Collector.
- Run app.
- Hit endpoints.
- Open tracing backend (Jaeger, Tempo, Datadog, etc.) and verify traces.

---

## 6. Sampling Strategies (Trade-offs)

**Why sample?**
Because collecting **every trace** in a big system is too expensive (CPU, memory, storage, money).

You sample to:
- Reduce cost.
- Reduce noise(not important data).
- Keep important data.

### 6.1 Head-Based Sampling

**Decision made at the START** of the trace (first span).

How it works:
- When a request arrives, sampler decides:
  - "Keep this trace" or
  - "Drop this trace".
- Decision is stored in trace context and propagated.

Types:
- **Probabilistic** – keep X% randomly (e.g., 10%).
- **Rate-limited** – keep at most N traces per second.

**Pros:**
- Simple to implement.
- Saves resources early (don’t even build spans for dropped traces).
- Predictable cost.

**Cons:**
-  Can miss rare bugs (e.g., error traces) if they happen inside dropped traces.
-  Doesn’t know yet if request will be slow or fail.

**Example:**

```text
Traffic: 10,000 requests/sec
Head sampling at 10% → 1,000 traces/sec stored
Cost reduced ~10x
But some error traces may be missed.
```

### 6.2 Tail-Based Sampling

**Decision made at the END** of the trace, once all spans are collected.

How it works:
- Collector buffers spans for each trace.
- When trace is complete, it decides:
  - Keep if: error happened, OR latency > threshold, OR specific route.
  - Maybe randomly keep some success traces (e.g., 10%).

**Pros:**
- Always keep **error traces**.
- Always keep **slow traces**.
- Can apply powerful rules (by service, user, route, etc.).

**Cons:**
- Needs Collector + more CPU & RAM (to buffer).
- Slight delay before trace is available.

**Example policy (in Collector config):**

```yaml
processors:
  tail_sampling:
    policies:
      - name: error-policy
        type: status_code
        status_code:
          status_codes: [ERROR]

      - name: latency-policy
        type: latency
        latency:
          threshold_ms: 2000

      - name: success-policy
        type: probabilistic
        probabilistic:
          sampling_percentage: 10
```

Meaning:
- 100% of error traces → kept
- 100% of slow traces (>2s) → kept
- 10% of normal traces → kept

### 6.3 Trade-offs

Head-based:
- Best when:
  - Very high traffic.
  - Need predictable cost.
  - Don’t have Collector yet.
- But: Might miss rare, important traces.

Tail-based:
- Best when:
  - Need **all error/slow traces**.
  - Have OTel Collector.
  - Can afford extra resource usage.

**Common real-world approach:**
- Use **head-based** sampling in SDK (e.g., 10%).
- Use **tail-based** in Collector to always keep errors/slow traces inside that 10%.

---

## 7. OpenTelemetry Collector Pipeline

The **Collector** is like a smart router and processor for telemetry.

### Components

1. **Receivers** – How data comes in
   - Examples: `otlp`, `jaeger`, `zipkin`, `prometheus`, `syslog`.

2. **Processors** – What happens to data in the middle
   - Filtering (drop noisy data).
   - Sampling (head/tail sampling for traces).
   - Batching (group data to send efficiently).
   - Attribute manipulation (add env, region tags, etc.).
   - Memory limiting, retrying.

3. **Exporters** – Where data goes out
   - Jaeger, Tempo, Prometheus, Datadog, New Relic, OTLP to another collector, etc.

### Pipeline Diagram (Text)

```text
         ┌───────────┐
         │ Receivers │  (otlp, jaeger, zipkin, prometheus...)
         └─────┬─────┘
               │
         ┌─────▼─────┐
         │ Processors│  (filter, sample, batch, add attributes)
         └─────┬─────┘
               │
         ┌─────▼─────┐
         │ Exporters │  (Jaeger, Tempo, Datadog, New Relic, ...)
         └───────────┘
```

### Example Collector Config (Traces Only)

```yaml
receivers:
  otlp:
    protocols:
      http:
      grpc:

processors:
  batch: {}
  tail_sampling:
    policies:
      - name: errors
        type: status_code
        status_code:
          status_codes: [ERROR]

exporters:
  otlp:
    endpoint: "https://otel-backend.example.com"

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch, tail_sampling]
      exporters: [otlp]
```

Meaning:
- Receive OTLP traces via HTTP/gRPC.
- Batch them.
- Tail-sample (keep certain ones).
- Export via OTLP to backend.

---

## 8. Simple OTel Architecture Diagram (Text)

```text
         ┌───────────────────────────┐
         │        Your Apps         │
         │  (with OTel SDK/API)     │
         └──────────┬────────────────┘
                    │  OTLP / HTTP / gRPC
                    ▼
         ┌───────────────────────────┐
         │    OpenTelemetry          │
         │        Collector          │
         │                           │
         │  ┌──────────┐             │
         │  │Receivers │  ← OTLP, Jaeger, Prometheus
         │  └────┬─────┘             │
         │       │                   │
         │  ┌────▼────┐              │
         │  │Processors│ ← filter, sample, batch, enrich
         │  └────┬────┘              │
         │       │                   │
         │  ┌────▼────┐              │
         │  │Exporters│ → Jaeger, Tempo, Datadog, etc.
         │  └─────────┘              │
         └───────────────────────────┘

                    │
                    ▼
         ┌───────────────────────────┐
         │   Observability Backends  │
         │ (Grafana, Jaeger, Datadog │
         │  New Relic, etc.)        │
         └───────────────────────────┘
```

---

