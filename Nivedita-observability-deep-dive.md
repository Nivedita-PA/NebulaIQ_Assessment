# Observability Fundamentals

## Metrics:

### Definition:

 Metrics give a specific number that tells you something important at a specific time, eg, how fast a system is responding, how many requests are happening right now, and how much memory usage is normal. 
 
### Data it contains:

Examples are given below:

* http_requests_total{method="POST", status="200"} 1027 @ 1700391645

* memory_usage_bytes{host="server1"} 4294967296 @ 1700391645
  
### Types:

There are four types of Metrics:

* Counters: It tracks the values that keep increasing/growing with time.

  Use case: Total HTTP requests, error counts
  
  Characteristics: Always increases, resets on restart

  Example:

  http_requests_total{method="post",code="200"} 1027


* Guages: It tracks the values that can increase as well as decrease at a specific point in time.

  Use case: Memory usage, active connections, CPU temperature

  Characteristics: Can increase or decrease, represents the current state

  Example:
  
  memory_usage_bytes{host="server1"} 4294967296

* Histograms: It is cumulative in nature. Let's take an example, when you require pages to load, it may take a different amount of time for different users, and putting it in an average load count as a Counter won't give enough perspective to the required information.
  As the load time varies for different users, their experiences with load time will also differ.

  Histogram works on a bucket system: Decide on a range before collecting the data -> put each measured data in a specific bucket -> count what's in each bucket.

  How the histograms look in code:
 
  http_request_duration_seconds_bucket{le="0.01"} 100
  http_request_duration_seconds_bucket{le="0.05"} 340
  http_request_duration_seconds_bucket{le="0.1"} 900
  http_request_duration_seconds_bucket{le="0.5"} 2100
  http_request_duration_seconds_bucket{le="1.0"} 2880
  http_request_duration_seconds_bucket{le="5.0"} 2950
  http_request_duration_seconds_bucket{le="+Inf"} 2950

  http_request_duration_seconds_sum 825.5 (total of all measurements)
  http_request_duration_seconds_count 2950 (total count)

 * Summary: It is similar to a histogram, but instead of giving you all the buckets, it gives you key numbers. Both tell the story, but summaries give you the easy to read overview summary version.

   Example:
   50th percentile (p50): 95ms     ← Half of pages load in 95ms or less
   
   90th percentile (p90): 250ms    ← 90% of pages load in 250ms or less
   
   95th percentile (p95): 550ms    ← 95% of pages load in 550ms or less
   
   99th percentile (p99): 1500ms   ← 99% of pages load in 1.5s or less

### Usage

* Real-time monitoring: Quick detection of anomalies and trends

* Alerting: Threshold-based alerts (e.g., CPU > 80%)

* Dashboards: Visual representation of system health

* Long-term trends: Historical analysis over months or years

## Logs:

### Definition:

Logs are immutable. They are timestamped records which contain detailed story of why did this happen ? Logs contain the full history of system events- errors, warnings, debug information, user actions, and state changes.​

### Types and type of data it contains:

* Unstructured(Plain text): Text that humans can read but machines are unable to parse. This kind of data is having flexible format but difficult to query at scale. Example is given below:

  2025-11-19 09:40:45 ERROR [payment-service] Payment processing failed for user_12345: TimeoutException - Connection to payment gateway timed out after 30s (transaction: txn_987654, amount: 49.99 USD)

* Structured(JSON/XML): Machine can read and is present as key value pair. This kind of data is easy to parse and query. Supports filtering and aggregation. Example is given below:

  {
  
  "timestamp": "2025-11-19T09:40:45.123Z",
  
  "level": "ERROR",
  
  "service": "payment-service",
  
  "trace_id": "7f8a9b2c3d4e5f6a",
  
  "span_id": "1a2b3c4d5e6f7g8h",
  
  "message": "Payment processing failed",
  
  "error": {
  
    "type": "TimeoutException",
    
    "message": "Connection to payment gateway timed out after 30s"
  
  },
  
  "context": {
  
    "user_id": "user_12345",
  
    "transaction_id": "txn_987654",
  
    "amount": 49.99,
  
    "currency": "USD"
  
  }
  
}

* Semi-structured: Not fully standardized.
  
### Log Levels

Standard severity levels help filter and prioritize log data:​

* DEBUG: Detailed diagnostic information

* INFO: General informational messages

* WARN: Warning messages for potentially harmful situations

* ERROR: Error events that might still allow the application to continue

* FATAL/CRITICAL: Severe errors causing application termination

### Usage

* Debugging: Understanding failure scenarios with full context

* Root cause analysis: Detailed investigation of issues

* Audit trails: Compliance and security tracking

* User activity tracking: Understanding user behavior

* Ad-hoc investigation: Exploring unexpected behaviors

## Traces:

### Definition:

Tracking where each request went at multiple levels and how long it spent at each stop. Complete journey can be seen and also exactly where the time is spent.

### Data Structure: Spans and Trace Context

A trace is composed of multiple spans(multiple levels). As your request goes to multiple levels each level needs to know that this request belongs to so and so user this info is called span context. Along with the span context some special info is also forwarded that info is called baggage.
If you add a lot of baggage the request becomes heavy and slow. Use baggae for important info only.

Span Components:

* Trace ID: Globally unique identifier for the entire trace (shared by all spans)

* Span ID: Unique identifier for this specific span

* Parent Span ID: Links this span to its parent, creating hierarchy

* Operation name: What this span represents (e.g., "database_query", "http_request")

* Start/end timestamps: Duration of the operation

* Attributes/Tags: Metadata (HTTP method, status code, user ID)

* Events: Point-in-time occurrences within the span

* Status: Success or error state
  
 Example of trace:

 Trace: 7f8a9b2c3d4e5f6a1b2c3d4e5f6a7b8c
 
├─ Span: API Gateway (150ms)

   ├─ Span: Auth Service (20ms)
   
   ├─ Span: Payment Service (100ms)
   
   │  ├─ Span: Database Query (30ms)
   
   │  └─ Span: External API Call (60ms)
   
   └─ Span: Notification Service (30ms)

Example of baggage:

baggage: user_tier=premium,experiment_id=exp_42,feature_flag=new_checkout

### Usage

* Distributed debugging: Understanding failures across microservices

* Latency analysis: Identifying bottlenecks in request flows

* Dependency mapping: Visualizing service interactions

* Ad-hoc exploration: Investigating specific user requests

* Performance optimization: Finding slow operations

# Protocols

## OTLP(Open Telemetry Protocol):

* OTLP supports formats like binary, JSON.
* It can deliver traces, metrics, or logs to /v1/traces, /v1/metrics, /v1/logs.
* It works with any language, it’s fast, can batch lots of data, and handles re-sending if something goes wrong.

## Prometheus Exposition Format

* Every few seconds Prometheus looks at your app and generates the report as how many errors, how much energy used
  
* The report might look like:
  
 #HELP http_requests_total The total number of HTTP requests
  
 #TYPE http_requests_total counter
 
 http_requests_total{method="post",code="200"} 1027
 
 http_requests_total{method="post",code="400"} 3

 * #HELP is just an explanation.

* #TYPE says if it’s a Counter or something else.

* Labels like {method="post"} help organize the stats.
  
* This info is visible at /metrics so Prometheus can collect it automatically.

## StatsD Protocol

*  Simple messages like user.login.attempts:1|c = “someone logged in, add 1 to the counter” Or api.response_time:142|ms = “this action took 142 milliseconds.”
  
*  Needs a stats collector to put everything togeather.
  
*  Metric types: Counter, Gauge, Timer/Histogram.

## Syslog Protocol

* Used for decades for sending computer and system event logs.
  
* Can send simple messages or structured messages.
  
* Key info includes: priority, timestamp, server name, app name, process ID, message type, extra info, and your message.

## Jaeger Thrift Protocol 

* Think of Jaeger as an efficient, specialized courier for your trace journeys.
  
* Data (trace spans) is sent on specific ports. Different languages might use different ports.

* Collectors receive traces through various endpoints.

* It packs up all the trace spans (steps in a request journey) efficiently, sends them, and gathers them so you can view how a request moved through your app.

# Cradinality

## Definition:

 Cardinality is the number of groups or categories. Example: If you’re only tracking 2 things, like red and blue balls, cardinality is low (just 2). If you give each ball a color, serial number, you could have thousands or millions of unique groups — that’s high cardinality.

 It matters because every unique group needs its own record (its own time series). Each one takes up memory, disk space, and affects speed.

 Adding labels like user ID, IP address, request ID, etc., where there are lots of possible values:

 Dangerous labels: Anything with millions of options: user IDs, individual session IDs, emails, IPs.

 Safe labels: Things with just a few options: GET/POST, error codes, server names, datacenter location — limited variety.

 Only use safe labels for metrics, so you don’t overload your system. Dangerous labels cause cardinality explosion.
 
  

  




  
  









  


  


