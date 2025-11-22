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
