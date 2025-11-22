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

## Protocols

### OTLP(Open Telemetry Protocol):

* OTLP supports formats like binary, JSON.
* It can deliver traces, metrics, or logs to /v1/traces, /v1/metrics, /v1/logs.
* It works with any language, it’s fast, can batch lots of data, and handles re-sending if something goes wrong.

### Prometheus Exposition Format

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

### StatsD Protocol

*  Simple messages like user.login.attempts:1|c = “someone logged in, add 1 to the counter” Or api.response_time:142|ms = “this action took 142 milliseconds.”
  
*  Needs a stats collector to put everything togeather.
  
*  Metric types: Counter, Gauge, Timer/Histogram.

### Syslog Protocol

* Used for decades for sending computer and system event logs.
  
* Can send simple messages or structured messages.
  
* Key info includes: priority, timestamp, server name, app name, process ID, message type, extra info, and your message.

### Jaeger Thrift Protocol 

* Think of Jaeger as an efficient, specialized courier for your trace journeys.
  
* Data (trace spans) is sent on specific ports. Different languages might use different ports.

* Collectors receive traces through various endpoints.

* It packs up all the trace spans (steps in a request journey) efficiently, sends them, and gathers them so you can view how a request moved through your app.

## Cradinality

### Definition:

 Cardinality is the number of groups or categories. Example: If you’re only tracking 2 things, like red and blue balls, cardinality is low (just 2). If you give each ball a color and, serial number, you could have thousands or millions of unique groups — that’s high cardinality.

 It matters because every unique group needs its own record (its own time series). Each one takes up memory, disk space, and affects speed.

 Adding labels like user ID, IP address, request ID, etc., where there are lots of possible values:

 Dangerous labels: Anything with millions of options: user IDs, individual session IDs, emails, IPs.

 Safe labels: Things with just a few options: GET/POST, error codes, server names, datacenter location — limited variety.

 Only use safe labels for metrics, so you don’t overload your system. Dangerous labels cause cardinality explosion.

 # Data Collection Architecture

 ## Collector:
 
 A collector is the middleman that receives the data from applications, filters it(filters bad data and combines related data), and then sends it back to the backend (database, observability platforms, eg, Grafana, Datadog).

 Problems with sending back the data to the backend are that every app should know how to filter, batch, and send data, get all data, including garbage data, want to use a new backend, then have to change every app, and if the backend is down, data is lost.

 ## Collection models:

 ### Agent-Based Collection:
 
 Install an agent(small program) on app/host.

 The agent has direct access to the app's code

 Data is extracted early, before wasting resources

 Can make smart decisions about what to keep

 Example: Datadog agent, OpenTelemetry Collector running as an agent

### Agentless Collection:
 
 A collector that pulls data from the app/host without installing anything on it.

 App exposes the matrics endpoint for the collectors to read the data periodically without caring who else is reading this data.

### eBPF-based collection: 

 Monitoring happens at the kernel level. It works like an eBPF program runs inside the kernel and automatically sees all network calls, system calls, and events without touching the code.
 
 Complete visibility: Sees everything

 Secure: Kernel verifies programs before running them

 Built-in: Already in modern Linux kernels, nothing to install

 ### Advantages:
 
* Minimal resource overhead

* No app modifications

* Complete invisibility (app doesn't know it's monitored)

* Catches all events without missing

* Great for security monitoring too

### Push model
The app sends the data when it wants, and the collector receives it.

Flexible timing, no need for fixed endpoints, good for frequent updates

### Pull model
The collector asks for the data, and the app responds by exposing the related endpoint.

Easy to verify authenticity

Predictable (know exactly when the collection happens)

Can control capacity (don't accept random streams)

Can distribute collectors easily

### Process happening at collection endpoint:

#### Filtering unwanted data:
It is the process of removing the data that is not needed. It is required not to send garbage data to the expensive backend.

#### Sampling strategy:

Head-based sampling: The Decision to keep or throw the data is being made very early at the start of a request.
It’s very fast and efficient—no waiting for more info. It saves CPU and memory because only selected requests create full trace data.
The system can’t know in advance if a request will end up being important—maybe it fails, is extra slow, or hits a bug. But you already decided whether to keep or discard it. If the request fails and you already said No you lose valuable debugging info.

Tail-based sampling: The decision to keep or drop a trace is made after the whole request finishes—when all the data is collected.
It’s great for debugging. You must collect all requests and spans first, which uses more CPU and memory, before filtering. It’s a bit slower and more complex.

#### Probabilistic:

#### Batching and buffering:
Batching: Instead of sending the data separately one by one, you collect the data and send it as a group/batch.

Buffering: Storing the data temporarily so that it can be processed in more efficient chunks.

#### Metadata enrichment
It is adding useful information to your data so that later it can be searched, used, and analyzed easily.
eg.

request_duration: 250ms

service: payment-api

region: us-east-1

environment: production

datacenter: aws-1a

team: payments

#### Protocol translation:
The conversion happening between the data formats is called protocol translation. Apps use different protocols, and the backend might require a different one. That's when protocol translation is needed.

### Collection at different layers:

#### Application layer: 

Apps keep track of what they do using the SDK (Software development kit)

#### System layer:

Collect metrics about the system —CPU, memory, disk usage, and running processes. This helps monitor infrastructure health.

#### Network layer:

Monitors network traffic between services without changing apps. Used for debugging network issues.

#### Kernel layer:

Runs monitoring code directly inside the OS Kernel. No overhead or app changes are required, eg. eBPF.

### Auto instrumentation:
Observability code is automatically added when the app starts—no coding required. Works with many popular frameworks and is quick to set up.

Pros: Fast setup, no code changes, broad coverage

Cons: Less control, can’t add custom business info, may miss some parts

### Manual instrumentation:
Observability code is added using the custom code, which can gather the chosen/important info.

Pros: Full control, better for critical paths

Cons: More coding, must understand tracing

# Backend Pipeline Architecture

## INGESTION LAYER: The Front Door
What it is: The entry point where data arrives at the backend​

### How Data Arrives
* HTTP: Like browsing a website (most common, flexible)

* gRPC: Like a high-speed direct phone line (faster, efficient)

* TCP: Raw network connection (very low-level, fast)

Like the post office front desk receiving packages through different doors

### Load Balancing & High Availability​
Load balancer: Distributes incoming data across multiple servers (like having 5 cashiers instead of 1)

Why: Prevents any single server from being overwhelmed

High availability: If one server crashes, others keep working​

### Initial Validation & Routing​
Validation: Check if data is properly formatted (Is this a valid package?)

Routing: Send metrics to metrics DB, logs to log storage, traces to trace storage

Drop bad data: Reject malformed or spam data early

## PROCESSING STAGES: The Smart Sorter
What it does: Cleans, organizes, and prepares data before storage​

### Parsing & Normalization​
What: Convert data into a standard format

text
Raw:  "ERROR: Payment failed user=123"

Parsed: {level: "ERROR", message: "Payment failed", user_id: 123}

Why: Makes data searchable and queryable

### Aggregation​

What: Combine raw data points into useful summaries

text

Raw: 1000 individual requests per second

Aggregated: rate(requests) = 1000/sec, p95 latency = 

Why: Saves storage, faster queries

### Rollups for Long-Term Storage​
What: Reduce precision for old data

text

Recent: Store every 10-second data point

1 week old: Store 1-minute averages

1 month old: Store 1-hour averages

Why: Huge storage savings while keeping trends

### Indexing​
What: Create searchable indexes (like a book's index)

text

Logs indexed by: timestamp, service name, error level, trace_id

Metrics indexed by: labels (method, status, region)

Why: Makes searches instant instead of scanning everything​

### Data Transformation & Enrichment​
What: Add context, convert formats

text

Before: request_duration: 250ms

After: request_duration: 250ms, service: payment-api, region: us-east-1

Why: Easier to filter and debug later

## STORAGE LAYER: The Warehouse
Why Different Storage for Different Data Types?​

### Time-Series Databases (for Metrics):​

Optimized for: Numbers changing over time

Structure: Timestamp + value + labels

Compression: 10-100x compression (delta encoding, gorilla algorithm)

Example: Prometheus, VictoriaMetrics, InfluxDB

Why special? Metrics are numeric, predictable patterns, can be heavily compressed

### Log Storage (for Text):​

Optimized for: Searching text, JSON

Structure: Full-text indexes, inverted indexes (word → documents)

Size: Large (raw text hard to compress)

Example: Elasticsearch, Loki

Why special? Logs contain rich text, need full-text search, unpredictable content

### Trace Storage (for Request Flows):​

Optimised for: Hierarchical spans, relationships

Structure: Trace ID → many spans with parent-child links

Size: Very large (many spans per request)

Example: Jaeger (Cassandra/Elasticsearch backend), Tempo (S3)

Why special? Traces are complex graphs, and need to reconstruct request flows

### Hot vs Warm vs Cold Storage​
Think of your kitchen:

Hot: Fresh food on the counter (use today)

Warm: Food in the fridge (use this week)

Cold: Food in the freezer (long-term storage)

* HOT Storage (Fast, Expensive):​

Hardware: SSD, NVMe, in-memory caches

Speed: Milliseconds

Cost: $$$

Retention: Last 24-72 hours

Use: Live dashboards, real-time queries, recent alerts

* WARM Storage (Balanced):​

Hardware: Standard HDD or mid-tier SSD

Speed: Seconds

Cost: $$

Retention: 1-4 weeks

Use: Weekly reports, occasional debugging

* COLD Storage (Cheap, Slow):​

Hardware: Archive storage (AWS S3 Glacier, Azure Archive)

Speed: Minutes to hours (must "thaw" data first)

Cost: $

Retention: Months to years

Use: Compliance, audits, forensics, rarely accessed

Why this matters: Storing everything hot costs 100x more than cold. Most data is rarely accessed after a week​

### Compression Techniques​

Compression is a way to make data take up less storage space by squeezing it down, like how a ZIP file makes files smaller.​

In observability, compression is crucial because you're constantly collecting huge amounts of data.

* Delta Encoding (for metrics):​ Store differences instead of actual values.

text

Raw: 100, 102, 105, 103, 107

Delta: 100, +2, +3, -2, +4

Result: Saves 50-70% space

* Run-Length Encoding: If the same value repeats, store count instead.

text

Raw: 0, 0, 0, 0, 1, 1, 5, 5, 5

Encoded: 0x4, 1x2, 5x3

* Gorilla Compression (for time-series):​ A special algorithm designed just for time-series data.

Exploits predictable patterns in metrics

Can achieve 10-100x compression

* gzip/LZ4 (for logs): General-purpose compression for any text.

General text compression

3-10x compression

### Retention Policies​
* Metrics: Keep forever (compressed to nothing)

Hot: 24 hours

Warm: 1 week

Cold: Years (heavily downsampled)

* Logs: Expensive, delete aggressively

Hot: 3-7 days

Warm: 1-2 weeks

Cold: 30-90 days

* Traces: Super expensive, shortest retention

Hot: 6-24 hours

Warm: 3-7 days

Sampled traces only (1-10%)  

## Query Languages

### PromQL (Prometheus Query Language) - for Metrics

Purpose: Query numerical metrics over time.

Style: Functional, like math formulas.

Examples:

rate(http_requests_total[5m]) — Get request rate over the last 5 minutes.

histogram_quantile(0.95, http_request_duration_seconds_bucket) — Calculate 95th percentile latency.

sum by (service) (http_requests_total) — Aggregate requests by service.

Best for: Alerting, dashboards showing trends, and statistical analysis of metrics.

### LogQL - for Logs

Purpose: Query and filter log entries (text events).

Style: Similar to PromQL but focused on text parsing and filtering.

Examples:

{service="payment"} |= "error" — Find error logs from the payment service.

rate({service="payment"} |= "error" [5m]) — Rate of error logs in last 5 minutes.

{service="api"} | json | status_code > 400 — Parse JSON logs and filter by status code.

Best for: Debugging by searching log content or filtering specific events.

### TraceQL - for Traces

Purpose: Query distributed trace data.

Style: Similar to PromQL but with span-specific filters.

Examples:

{duration > 1s} — Find traces longer than 1 second.

{span.service="payment" && span.status="error"} — Errors in payment service traces.

{resource.service.name="api" && span.http.status_code >= 500} — Complex filters on traces.

Best for: Finding slow requests, errors, and analyzing request flows.

## Query Optimization

Observability systems make queries fast and efficient by:

Time-based pruning: Only scanning data relevant to the requested time range.

Index usage: Jumping directly to relevant data using indexes (like a book's index).

Parallel execution: Querying multiple shards or servers in parallel.

Query planning: Rewriting queries behind the scenes to reduce work.

## Aggregation at Query Time vs Storage Time

* Storage-time aggregation (Rollups):
Data is pre-aggregated and summarized before storing.

Pros: Very fast query responses, low CPU during query.

Cons: You can’t change the aggregation type later; less flexible.

* Query-time aggregation:
Raw data is stored, and aggregation happens when you query.

Pros: Flexible, can create any custom aggregation on demand.

Cons: Queries take longer, more CPU-intensive.

## Caching Strategies

* Query result caching: If the same query is repeated, return cached results instantly instead of running the query again.

Example: A Query for “errors in the last 24 hours” may be cached to respond in milliseconds.

* Data caching: Frequently accessed data, especially recent “hot” data, is kept in memory or fast storage for quick access.

These strategies drastically reduce latency and backend load, making dashboards and alerting responsive.

## Intelligence Layer: Insights, Anomalies, and Root Cause Analysis3

### INSIGHTS: Smart Patterns from Raw Data

Insights are useful conclusions or patterns that observability systems automatically checks out from raw metrics, logs, and traces. They answer questions like: What changed recently? or Is this performance normal?

Instead of just showing raw numbers, insights explain what the numbers mean.

Example:

text
Raw data:

- API response time: 250ms, 245ms, 248ms, 252ms, 249ms, 251ms
  
- Trend: Fairly stable around 250ms

Insight: API latency has been stable at 250ms for the past hour

Observability platforms follow a pipeline:

1. Collect Raw Telemetry
   
They gather:

* Metrics
* Logs 
* Traces
  
This is just raw data—numbers and text.

2. Normalize & Aggregate the Data

Systems group the data into meaningful units:

Average over time windows (e.g., average latency per minute)

Count occurrences (e.g., number of errors per hour)

Group by labels (service, endpoint, region)

This step makes the data easier to analyze.

3. Apply Statistical Analysis

Insights come from detecting patterns, such as:

* Averages & Baselines

What is normal ?

e.g., latency usually = 200–250ms
→ This becomes the baseline.

* Trends

Is something slowly increasing/decreasing?

CPU rising from 40% → 65% over 30 minutes

* Anomalies

Is something suddenly different?

Latency jumped from 250ms → 800ms

* Seasonality

Patterns that repeat at similar times:

High traffic every day between 6–7 PM.

4. Apply Rules or Algorithms

Platforms use:

Threshold rules (error rate > 5%)

Change detection algorithms (CUSUM, EWMA)

Machine learning anomaly detection

Correlation analysis (error spike + deployment)

Pattern recognition

5. Convert Findings Into Human-Friendly Insights

Finally, the system converts the analytical results into a readable statement.

Example:

Raw Data:

Latency: 245, 248, 250, 252, 249…

Analysis:

Variation small

No upward trend

Matches baseline

Insight: API latency has been stable at around 250ms for the past hour.

## Pattern Recognition Techniques

* Statistical Analysis: Understanding Your Data
  
What it is: The middle value that represents what's typical.

Why it matters:

Gives you a baseline to compare against

If mean jumps from 200ms to 500ms = something broke

Track mean over time to spot trends

Example:

Your Reddit clone's page load times:

Times: 150ms, 155ms, 148ms, 152ms, 160ms

Mean: 153ms

You'd tell stakeholders: "Our average page load is 153ms"

* Trend Analysis: Spotting Patterns Over Time
  
   What it is: Looking at how metrics change over hours, days, weeks to find patterns or problems.

   Why trend analysis matters:

   Detect problems early (before they become critical)

   Spot seasonal patterns (plan capacity accordingly)

   Confirm fixes worked (latency went down after deploying fix)
   
* Correlation Analysis: Finding Relationships
  
    What it is: When two metrics change together, they might be related. Correlation analysis finds these relationships.
  
    Why correlation matters:

    Helps identify root causes ("What changed when the problem happened?")

    Detect cascading failures ("Service A fails → Service B times out")

    Validate fixes ("We fixed X, and now error rate is stable")

## Machine Learning Approaches: Advanced Pattern Recognition

- Clustering: Grouping Similar Patterns

What it is: Automatically grouping data into categories based on similarities.

Example:

Your Reddit clone traffic over a week:

Group 1: Quiet Times

- Weekdays 2-3 AM, weekends 3-4 AM
- Very low traffic, high availability
- Few errors (if any)
- Low resource usage

Group 2: Normal Business Hours

- Weekdays 9 AM - 6 PM
- Medium-high traffic
- Some errors (0.1-0.5%)
- Medium resource usage

Group 3: Peak Hours

- Weekdays 12-1 PM, 5-6 PM
- Very high traffic (traffic spike)
- Higher error rate (0.5-2%)
- High resource usage

Group 4: Anomalous/Unusual

- Traffic spikes at 3 AM (unusual!)
- Error rate 10% (abnormally high)
- Everything else is weird too
- ALERT: Something unusual happened


Why clustering helps:

Automatically categorizes data without manual rules

Detects "Group 4" = anomalies that don't fit normal patterns

Can adapt as patterns change (learning system)
  
- Forecasting: Predicting Future Behavior
  
What it is: Using past data to predict what will happen next.  

Why forecasting helps:

Plan infrastructure before hitting limits

Budget for growth

Schedule maintenance during quiet times

- Classification: Categorizing Issues

  What it is: Automatically categorizing errors or issues into types.

Why classification helps:

- Prioritize which issues to fix first (80/20 rule)

- Route alerts to right teams (database team gets database errors)

- Track trends ("Are we getting more timeouts this month?")

- Automate response ("If timeout error → trigger auto-scaling")

Example:

Your Reddit clone has high latency alert:

Statistical Analysis: 
→ p95 latency is 5 seconds (was 200ms yesterday)

Trend Analysis:
→ Latency has been increasing steadily for 2 hours

Correlation Analysis:
→ Latency spike correlates with code deployment at 9 AM
→ Latency correlates with high CPU usage

Clustering:
→ Current traffic pattern is different from normal (anomalous)

Forecasting:
→ If trend continues, system will be down in 1 hour

Classification:
→ Most errors are "database timeouts" (80%)
→ All started after the deployment

Conclusion: "Deployment introduced a database query bug"
Action: Rollback deployment
Result: Latency back to normal, no outage

## Anomalies and Root Cause Analysis (RCA) - Complete Guide

### ANOMALIES: Detecting the Unusual

An anomaly is when something behaves differently than expected. It's data that stands out from normal patterns.
If your website usually gets 1,000 requests per second, suddenly getting 10,000 requests per second is an anomaly. Or if your API response time is usually 200ms but jumps to 5,000ms, that's an anomaly.

Real-world examples

- Error rate spikes from 0.1% to 10% (sudden increase)
- Database connections drop to zero (sudden decrease)
- Memory usage keeps growing without stopping (trend anomaly)
- API latency is wildly variable instead of stable (consistency anomaly)
- Request traffic at 3 AM when it's always quiet (timing anomaly)

### Anomaly Detection Methods

#### 1. Threshold-Based Detection

**Static Threshold: Simple but Rigid**

Rule: "Alert if CPU > 80%"

Example:
- Server A (heavy workload): CPU at 85% = ALERT (but this is normal for this server)
- Server B (light workload): CPU at 85% = ALERT (genuinely unusual for this server)

Problem: Same rule doesn't work for different situations

When to use static thresholds:

- Simple systems with consistent behavior
- Clear, obvious limits (disk full at 100%)
- Quick and dirty alerting

**Dynamic Threshold: Smarter Adaptation**

Rule: "Alert if CPU > (average CPU + 30%)"

How it learns:
- Monday typical CPU: 40-50%
- Weekend typical CPU: 20-30%
- Monday at 60% CPU = Alert (20% above normal for Monday)
- Weekend at 40% CPU = Alert (10% above normal for weekend)

Same server, different rules for different times
Much more intelligent!

When to use dynamic thresholds:

- Systems with varying load patterns
- Want to avoid false alarms
- Don't know exact healthy threshold upfront
  
#### 2. Statistical Methods

**Z-Score Method: How Far from Normal?**

Imagine your API response time has:
- Average: 250ms
- Standard Deviation: 50ms (how spread out the values are)

Today's response time: 500ms

Z-score = (500 - 250) / 50 = 5

What this means:
- 500ms is 5 standard deviations away from average
- This happens naturally <0.00001% of the time
- This is DEFINITELY an anomaly

Rule: If Z-score > 3, it's anomalous
(3 standard deviations = happens naturally <0.3% of the time)

**Real example:**

Your Reddit clone response times:
Normal: 100-200ms (average 150ms, std dev 25ms)

Day 1: Response time 200ms
  Z-score = (200-150)/25 = 2 (normal, expected variation)

Day 2: Response time 250ms
  Z-score = (250-150)/25 = 4 (ANOMALY, alert!)

Day 3: Response time 1000ms
  Z-score = (1000-150)/25 = 34 (HUGE ANOMALY, critical alert!)

**Pros:** Works well for normally distributed data
**Cons:** Fails if data is skewed or has outliers

**IQR (Interquartile Range) Method: Robust Outlier Detection**

Concept: Divide data into 4 quarters

Example: Response times: 50, 100, 150, 200, 250, 300, 5000ms

Q1 (25th percentile): 125ms
Q2 (50th percentile/median): 175ms
Q3 (75th percentile): 275ms

IQR = Q3 - Q1 = 275 - 125 = 150ms

Outlier = anything beyond (Q3 + 1.5 × IQR)
        = 275 + 1.5×150
        = 275 + 225
        = 500ms

So: Any response time > 500ms is an outlier

In our data: 5000ms > 500ms → ANOMALY!

**Why IQR is better than Z-score:**
- Works with skewed data
- Less affected by extreme outliers
- More robust for real-world messy data

**Moving Average Method: Adapts to Changing Baseline**

Calculate average of last N measurements
Compare current value to this moving average

Example: Track CPU over 10 minutes using 5-minute moving average

Time    Actual CPU    5-min Average    Difference
0:00    40%           40%              0%
0:05    42%           41%              +1%
0:10    45%           42.3%            +2.7%
0:15    48%           43.8%            +4.2%
0:20    52%           45.4%            +6.6%
0:25    70%           47.2%            +22.8% ← ANOMALY! (spike)
0:30    75%           51.2%            +23.8% ← Still anomaly
0:35    55%           54.4%            +0.6%  ← Back to normal
0:40    50%           54.3%            -4.3%

Alert: CPU spiked at 0:25 (deviation > 20%)


**Why moving average is useful:**
- Adapts to changing normal patterns
- Catches trends (like gradual increase)
- Less sensitive to single outliers

**Isolation Forest: Finding Isolated Anomalies**

Concept: Normal points cluster together. Anomalies are isolated.

How it works (simplified):
1. Create random "decision trees" that split data into groups
2. Normal data needs many splits to isolate (takes longer)
3. Anomalies get isolated quickly (fewer splits needed)
4. Calculate anomaly score based on isolation speed

Example:
- High CPU + High Memory + High Error Rate + Slow Queries
  → Very unusual combination
  → Gets isolated quickly
  → Anomaly Score: HIGH

- High CPU (but normal memory, normal errors, normal queries)
  → Not unusual overall (CPU might spike during reports)
  → Needs more splits to isolate
  → Anomaly Score: LOW


**Why Isolation Forest works well:**
- Doesn't need to know "normal" upfront
- Works with multi-dimensional data
- Catches complex anomalies (unusual combinations)
- Fast to compute

**Autoencoders: Neural Network Pattern Learning**

Concept: Neural network learns to "compress and reconstruct" normal data

How it works:
1. Train on months of normal, healthy data
2. Network learns compression pattern (what normal looks like)
3. Given new data, try to reconstruct it
4. If reconstruction error is high = anomaly

Example:
Normal data: CPU 45%, Memory 60%, Requests 1000/sec, Errors 5/hr
Network compresses: "typical daytime pattern"
Can reconstruct perfectly: CPU ~45%, Memory ~60%, etc.

Anomaly data: CPU 95%, Memory 95%, Requests 5000/sec, Errors 500/hr
Network tries to reconstruct: Should be "CPU 45%, Memory 60%"
But actual is "CPU 95%, Memory 95%"
Reconstruction error is huge → ANOMALY

**When to use autoencoders:**
- Complex, non-linear patterns
- Need to learn from historical data
- Have lots of training data available

**LSTM: Time-Aware Anomaly Detection**

Concept: Special neural network that remembers patterns over time

How it works:
1. Train on time-series of normal behavior
2. Network learns "what comes next" in the sequence
3. Given new data, predict what should happen
4. If actual differs from prediction = anomaly

Example: Daily traffic pattern
Normal pattern:
  Midnight: 100 requests/sec
  3 AM: 50 requests/sec
  6 AM: 100 requests/sec
  9 AM: 1000 requests/sec ← PEAK
  6 PM: 2000 requests/sec ← PEAK
  Midnight: 100 requests/sec

Network learns this pattern.

New situation:
  It's 3 AM, network predicts: 50 requests/sec
  Actual: 5000 requests/sec (unexpected spike!)
  Prediction error huge → ANOMALY


**When to use LSTM:**
- Time-dependent patterns matter
- Seasonal or cyclical behavior
- Need to predict based on sequence
  
**Seasonality: Predictable Patterns That Repeat**

Examples:
- Daily: Traffic high during business hours, low at night
- Weekly: Traffic high Mon-Fri, lower on weekends
- Yearly: Traffic high during holidays, school terms
- Hourly: Traffic spikes at specific times (lunch rush 12-1 PM)

Problem: If you don't account for seasonality, you'll get false alarms

Example without seasonality awareness:
  Rule: "Alert if traffic > 5000 requests/sec"
  
  Monday 10 AM: 8000 requests/sec → ALERT (but this is normal for Monday morning)
  Sunday 3 AM: 3000 requests/sec → No alert (but this is high for Sunday 3 AM)

Solution: Learn seasonal patterns
  "During business hours Mon-Fri, 5000 req/sec is normal"
  "During weekends, 1000 req/sec is normal"
  "Late night, anything >500 req/sec is anomaly"

**How to handle seasonality:**
1. Learn baseline for each "season" (time of day, day of week, etc.)
2. Compare current to season-specific baseline
3. Adjust thresholds per season


**Trends: Gradual Changes Over Time**

Example: Memory leak in application
  Day 1: Memory 40%
  Day 2: Memory 42%
  Day 3: Memory 44%
  Day 4: Memory 46%
  Day 5: Memory 48%
  Day 6: Memory 50%
  Day 7: Memory 52%
  Day 8: Memory 54% → Server might crash soon!

Individual data points look normal (54% isn't high)
But the TREND is alarming (increasing 2% per day)

Solution: Detect trend, not just individual values

**How to detect trends:**
1. Calculate linear regression (best-fit line through data)
2. Check slope of line (how fast is it increasing/decreasing?)
3. Forecast where it will be in N days
4. Alert if trending toward bad outcome

Forecast: "At current rate, memory will reach 100% in 5 days"
Action: Restart service, fix memory leak, before system crashes

#### 5. False Positive Reduction

**Problem: System Cries Wolf Too Often**

You get 100 alerts, but 95 are false alarms (not real problems)
Result: Team ignores alerts (alert fatigue)
When real problem comes, team misses it

False positive = alert for non-problem
False negative = no alert for real problem

We want: Few false positives, few false negatives (hard!)


**Techniques to reduce false positives:**

1. Require Confirmation (multiple signals)
   Rule: "Alert only if (CPU > 80% AND Memory > 80% AND Errors > 1%)"
   
   Prevents: CPU spike alone won't trigger alert
   Result: Only alerts when multiple things go wrong simultaneously
   
2. Require Duration (anomaly must last a while)
   Rule: "Alert if CPU > 80% for MORE THAN 5 MINUTES"
   
   Prevents: CPU spike for 30 seconds won't trigger
   Result: Filters out temporary blips, catches real problems  

3. Context Awareness (know what's supposed to happen)
   Rule: "Ignore CPU spikes during known batch jobs (every Sunday 2-4 AM)"
   
   Prevents: Alerting on expected behavior
   Result: Only alerts on unexpected anomalies
   
4. Confidence Threshold (be really sure before alerting)
   Rule: "Alert only if anomaly score > 0.95 (95% confidence)"
   
   Prevents: Marginal anomalies (might be noise)
   Result: Only high-confidence anomalies trigger alerts
   
5. Whitelist Known Events
   Rule: "Don't alert for 30 minutes after deployment (normal to have spikes)"
   
   Prevents: Expected post-deployment behavior triggering alerts
   Result: Cleaner, more relevant alerts

#### 6. Severity Classification

**Not All Anomalies Are Equal**

Severity levels:

 GREEN: Minor deviation (1-5% change)
  - Probably just noise
  - No action needed
  - Don't alert

 YELLOW: Moderate deviation (5-20% change)
  - Something unusual happening
  - Alert the team, but not urgent
  - Team can investigate when they have time
  - Example: "CPU 75%" when normal is 50%

ORANGE: Major deviation (20-50% change)
  - Significant problem
  - Alert immediately
  - Team should start investigating
  - Example: "Error rate 5%" when normal is 0.1%

`RED: Critical deviation (>50% change or system down)
  - System in crisis
  - Page on-call engineer immediately
  - Emergency response
  - Example: "API response 10 seconds" when normal is 200ms

Alternative: Confidence-based severity
  - Confidence < 60%: GREEN (probably false alarm)
  - Confidence 60-80%: YELLOW (probably real, maybe not urgent)
  - Confidence 80-95%: ORANGE (likely real problem)
  - Confidence > 95%: RED (definitely a problem)
    
## Root Cause Analysis (RCA) - Complete Guide

### What is RCA?

**RCA = Finding the Real Problem, Not Just the Symptoms**

**In observability:**

Symptom: "API returning 500 errors to users"
Shallow cause: "Web server crashed"
Root cause: "Memory leak in new code released 2 hours ago"

Fix symptom: Restart web server (temporary, leak happens again)
Fix root cause: Patch the code, redeploy (permanent solution)

## Why is RCA Hard in Distributed Systems?

**Simple Systems (Easy):**
Monolithic app on one server:
Problem occurs → Check that server's logs → Found it! → Fix it

Example:
Server A has memory leak
Check Server A logs
See: "Memory increasing, can't allocate"
Fix: Patch code
Done.

**Distributed Systems (Hard):**

100 microservices, multiple databases, service mesh, caches, message queues
Problem: Users getting timeouts from API
Which service is slow? Why is it slow?

The blame chain can be long:
Deployment at 9 AM
→ New code has unoptimized query
→ Database query takes 5 seconds instead of 50ms
→ Payment Service times out waiting for database
→ API Gateway times out waiting for Payment Service
→ Users get 500 errors

To find root cause, must trace through entire chain.

## Root Cause Analysis (RCA) Techniques

rca-guide

### 1. Correlation Analysis — Find What Changed Together

If multiple metrics spike or drop at the same time, they are likely connected.

Example:

Errors spike at 9:05 AM

CPU, memory, DB connections also spike at 9:05 AM

Deployment happened at 9:04 AM

→ Likely the new release caused resource issues.

How to use:

Compare metrics on a timeline

Look for patterns that start at the same moment

Connect them to a deployment or config change

### 2. Dependency Mapping — Know Who Depends on What

Understanding service relationships helps identify where a failure started.

Example:
If Auth DB fails → Auth Service fails → API Gateway rejects all requests.

How to use:

Map your service architecture

Identify whether the failure is full or partial

Trace the failing path to the true failing component

### 3. Trace Analysis — Find the Slowest Step in a Request

Traces show how a request flows through services and where time is spent.

Example:
A request takes 460ms.
Database query takes 370ms → this is the bottleneck.

How to use:

Look for the longest spans

Identify time-heavy operations

Optimize the slowest section

### 4. Log Aggregation & Pattern Detection — Find Common Errors

Systems generate millions of logs—pattern matching quickly narrows down the problem.

Example:
Searching logs shows:

80% errors: “Connection refused”
All from Payment Service → to Database
→ Database connection pool exhausted.

How to use:

Filter logs by time, service, and severity

Group errors by message

Identify dominant error patterns

### 5. Change Event Correlation — Most Incidents Happen After a Change

Always ask: “What changed right before the problem?”

Example:

8:50 — New version deployed

9:05 — Error spike

9:30 — Rollback

9:35 — Errors cleared
→ Release introduced a bug.

Correlate with:

Deployments

Config updates

DB migrations

Infra changes

Feature flag toggles

# Observability User Experience - Complete Guide
## Dashboard Patterns: How to Show Information
### Common Visualization Types
1. Time-Series Graphs (Line, Area, Stacked)
What it shows: Data changing over time

Best for:
- API latency over 24 hours
- Error rate trends
- CPU usage over time
- User requests per minute

Why use it:
- Natural way to see trends
- Easy to spot sudden changes
- Can compare multiple lines

2. Gauges and Single-Stat Panels
What it shows: One important number right now

Best for:
- Current system health
- Important KPIs (CPU, Memory, Disk)
- Active user count
- System status (green = good, red = bad)

Why use it:
- Quick at-a-glance information
- No need to read graphs
- Perfect for alerts

3. Heatmaps (Latency Distribution)
What it shows: How data is distributed (many small values on left, few on right?)

Best for:
- Request latency over time
- Showing slow requests vs normal
- Identifying patterns (always slow at noon?)

Why use it:
- Shows distribution, not just average
- Can spot patterns quickly
- Visualize tail latency (the slow requests)

4. Histograms
What it shows: How many requests fall into each time bucket?

Best for:
- Request duration distribution
- Error frequency breakdown
- Disk usage categories

Why use it:
- See distribution at a glance
- Understand typical vs worst-case

5. Tables and Logs View
What it shows: Structured list of data

Best for:
- Comparing multiple services
- Detailed error logs
- Recent events/incidents
- Service inventory

Why use it:
- Easy to scan and compare
- Good for detailed information

### Dashboard Organization
#### Infrastructure Dashboards

Shows: Server health, resources

Purpose: Monitor server health  
Audience: DevOps, SREs  
Time range: Last 24 hours  

#### Application Dashboards (RED Metrics)

Shows: Application performance

RED = Requests, Errors, Duration

Purpose: Monitor application health  
Audience: Backend engineers, product team  
Time range: Last 1 hour  

#### Business KPI Dashboards

Shows: Business metrics

Purpose: Show business impact  
Audience: Product, Management  
Time range: Last 30 days  

#### Variables and Templating for Filtering
Dropdown filters:

All panels update based on filters.  

#### Information Density

Too Much Information (Overwhelming):

 50 different metrics            
 100 panels per screen           
 Colors everywhere               
 No clear hierarchy              
 User: "Where do I look first?"
 
Just Right (Scannable):

 TOP: 4 most important metrics    
 MIDDLE: Related trends           
 BOTTOM: Details (if needed)      
 User: "I know what matters"      

## User Workflows: How People Actually Use Dashboards
### Debugging an Incident - Typical Flow

STEP 1: Alert Triggers
   Alert: "Error rate > 5%"
 Severity: HIGH
User thinks: "Something broke!"

STEP 2: Open Dashboard
 Click dashboard link from alert
 Dashboard loads
 Time range automatically set to last 1 hour
 User sees: "Error rate is red, other metrics look okay"

STEP 3: Identify Time Range and Components
 User thinks: "When did it start?"
Drag time range on graph: Error started at 2:30 PM
 Look at which services affected:
   API: Error 
   Database: Normal 
    Cache: Normal 
 Conclusion: API is the problem

STEP 4: Drill Down into Specific Service
 Click on "API Service" panel
 New dashboard opens: API-specific metrics
 See: CPU 95%, Memory 85%, Connections: 1000/1000
 User thinks: "API is out of resources!"

STEP 5: Correlate Metrics & Check Logs
 Click "Logs" tab on dashboard
 Filter: service="api" AND level="ERROR"
 See: Millions of "Connection refused" errors
 All errors started at 2:30 PM

STEP 6: Find Traces for Slow Requests
 Click trace link in error log
 See: Full request timeline
 Database query took 30 seconds
 User thinks: "Database is timing out!"

STEP 7: Analyze Span Breakdown
Trace shows:
  API Gateway: 50ms 
   Application logic: 100ms 
  Database query: 30,000ms
 Root cause: Database query is VERY slow

STEP 8: Identify Root Cause
 Check database dashboard
See: Disk I/O 99% (maxed out)
 Check: Was there a deployment at 2:30 PM?
 Answer: YES! New report feature deployed
 Check: New feature added unoptimized query
 ROOT CAUSE FOUND: New query doing full table scan

STEP 9: Verify Fix
 DBA adds database index
 Check: Database query time drops to 50ms
 Check: Error rate clears
 Check: Alerts turn green
 User thinks: "Fixed! Incident closed."

Timeline: 15 minutes to root cause

### How Users Navigate Between Metrics, Logs, Traces
Metrics → Logs → Traces

Three-Step Navigation:

STEP 1: Metrics Dashboard (High Level)
 See error spike
 Drill down into service

STEP 2: Logs (Find Error Context)
 See error messages
 Click trace_id link

STEP 3: Traces (See Request Flow)
  See full request timeline
  Identify slow span
  Look at logs for that time
  Circle back to metrics

Time Range Selection and Time-Sync
Single time range → All panels update together
Metrics, logs, and traces use same time window

Search and Filtering Patterns
Simple search box  
Advanced filters (service, level, time, status)  
Instant results  

### Tool Analysis: Comparing Datadog, Grafana, New Relic
#### What Makes a Dashboard Intuitive:
- Clear title
- Key metric at top
- Visible time range
- Clear color meaning
- Easy drill-down
- Fast loading
- 
 Bad Dashboard: 
- No title
- Too many panels
- Hidden time range
- Confusing colors
- Not clickable
- Slow

#### Handling Large Datasets:
Tool Approach:

Grafana: 
- Aggregates data before showing
- Shows last 1000 points
- User can zoom in for detail
- Responsive even with 1 year of data

Datadog:
- Pre-calculates rollups
- Shows summary graphs
- Click to explore details
- Very fast response

New Relic:
- Downsamples based on time range
- Last hour = every second
- Last month = every hour
- Keeps performance good

Pattern: Never show raw millions of data points
Always aggregate/summarize first

#### Real-Time Updates vs Static
Real-Time (Auto-refresh every 10 seconds):
Pros: See problems immediately
Cons: Data moves, hard to read while scrolling

Static Snapshot (Manual refresh):
Pros: Can read carefully, data doesn't move
Cons: Might miss fast-changing issues

Best Practice: 
- Default: Auto-refresh every 30 seconds
- User can toggle to manual
- Can pause while investigating

#### How Insights/Anomalies Are Presented
GOOD:
Clear alert, context, actions

BAD:
Raw numbers without meaning

### Insight Presentation: How Insights Reach Users
METHOD 1: Cards on Dashboard
```
┌─────────────────────────────┐
│ Insight Card                │
│ ┌────────────────────────┐  │
│ │ 💡 API Latency Spike   │  │
│ │                        │  │
│ │ p95 increased 50% in   │  │
│ │ last 30 minutes        │  │
│ │                        │  │
│ │ [Investigate]          │  │
│ └────────────────────────┘  │
└─────────────────────────────┘
```

METHOD 2: Notifications
- Pop-up message
- Email alert
- Slack message
- SMS for critical

METHOD 3: Dedicated Insights Page
- List of all recent insights
- Filter by type, severity
- See which you've already seen  

### Anomaly Visualization
Without visualization:
"CPU increased 45% at 2:30 PM"
(Hard to understand severity)
```
With visualization:
CPU graph with anomaly marked:
│     ┌─ ANOMALY ALERT
│     │ ┌──────┐
│     │ │      │
│ ────┼─┘      └────── (Average line)
│
(Immediately clear: unusual spike) 
```
### RCA Presentation
How tools guide users to root cause:

1. Problem shown (red alert)
2. Related metrics highlighted (what changed?)
3. Timeline shown (when did it happen?)
4. Root cause suggested (likely cause: deployment)
5. Action buttons (View deployment, rollback)

Example flow:
```
Error alert
├─ [View related metrics] ← Helps understand scope
├─ [Check recent changes] ← Shows deployments
├─ [View slow traces] ← Shows what was slow
└─ [View deployment logs] ← Explains root cause
```

#### Alert Management UX
Alert Management Interface:
```
┌──────────────────────────────┐
│ Alerts: 5 Critical, 12 Warning│
├──────────────────────────────┤
│ ☐ API Error Rate > 5%       │
│   2:30 PM - 2:45 PM         │
│   Status: Triggered          │
│   [Acknowledge] [Snooze]     │
├──────────────────────────────┤
│ ☐ Database CPU > 90%         │
│   2:35 PM - ongoing          │
│   [Acknowledge] [Silence]    │
└──────────────────────────────┘
```
Features:
- Acknowledge: I'm working on it
- Snooze: Alert me again in 1 hour
- Silence: Stop alerting (maintenance mode) 

#### User's Mental Model When Investigating:
```
START: "Something is broken!"
│
├─ Question 1: "How bad is it?"
│  └─ Action: Open dashboard, check severity
│
├─ Question 2: "When did it start?"
│  └─ Action: Look at time on the graphs
│
├─ Question 3: "What broke?"
│  └─ Action: Check which service/component affected
│
├─ Question 4: "Where exactly?"
│  └─ Action: Drill into affected service
│
├─ Question 5: "Why did it break?"
│  └─ Action: Check logs and traces
│
├─ Question 6: "What changed?"
│  └─ Action: Check deployment, config changes
│
└─ CONCLUSION: "Found root cause!"
   └─ Action: Apply fix, verify it worked
```

Dashboard design should follow this journey, not require hopping around.

### What Could Be Improved:

### Problem 1: Too Much Information
Current: 50 panels per dashboard

Better: 5 key panels + link to details

Most users: "I don't need all 50 metrics"

### Problem 2: Slow Load Times

Current: Dashboard takes 10 seconds to load

Better: Shows in 2 seconds with zoom for details

Users: "I left to get coffee while the dashboard loaded."

### Problem 3: Unclear Drill-Down

Current: Click metric → goes to random new dashboard

Better: Context preserved, time range synced, related data shown

Users: "Where am I now? What was my time range?"

### Problem 4: Alert Fatigue

Current: 100 alerts per day (99 false positives)

Better: Smart filtering, group-related alerts

Users: "I ignore all alerts because most are noise."

### Problem 5: Hard to Navigate Between Tools

Current: Open Grafana → then open Datadog → then log into Jaeger

Better: Single pane of glass with all data

Users: "I forget which tool shows what."

# 6. Technical Challenges & Trade-offs in Observability (Simple Version)

Observability is powerful, but it’s not free. Every design choice has trade-offs (good and bad sides). Below is a simple, high-level overview in layman terms.

---

## 1. Data Volume & Cost

**How much data do large apps generate?**  
Big systems can easily generate **hundreds of GB to many TB of data per day** (metrics, logs, traces).

Example:
- A medium Kubernetes cluster can already produce **hundreds of GB per month** just in metrics for a small app.
- Real production systems at scale can hit **tens or hundreds of TB per day**.

**Why this is a problem:**
- Storage costs go up fast (disk, object storage, cloud bills).
- Network costs go up (sending data across regions / to SaaS vendors).
- Tools like Splunk/Datadog often charge **per GB ingested**, so bills explode if not controlled.

**What tools do:**
- Offer cheaper storage tiers (hot/warm/cold).  
- Compress data and downsample old data.  
- Charge different rates per data type (logs more expensive than metrics).

---

## 2. Cardinality Explosion

**What is it?**  
Cardinality = how many unique label combinations a metric has.  
You get **cardinality explosion** when you add labels like `user_id`, `session_id`, `ip`, `trace_id` to metrics and end up with **millions or billions** of unique time series.

**What causes it?**
- Labels with many possible values:
  - `user_id`
  - `request_id`
  - `email`
  - `ip_address`
  - `container_id`

**Impact:**
- Huge memory usage in metric stores.
- Slow queries.
- High cost (each time series consumes CPU+RAM+disk).
- In extreme cases, monitoring system collapses.

**Mitigation Strategies:**
- **Don't put user IDs / request IDs** into metrics. Use logs/traces instead.
- Aggregate by groups ("premium vs free users") instead of per-user.
- Limit which labels are allowed (enforce rules in collectors).
- Regularly review and delete useless metrics.
- Use platforms with good cardinality management features.

**What tools do:**
- Some enforce label limits.
- Some have features to collapse/aggregate series.
- Most recommend *strict label discipline*.  

---

## 3. Sampling Trade-offs

**Why sample?**  
Because you **can’t afford to store every trace** in a high-traffic system.

Example:  
1 million requests per second × 50 spans each = 50 million spans per second.  
Storing all of that is insane.

**What you lose when sampling:**
- You do **not see every request**.
- You might miss some rare issues (if sampling is naive).

### Head-based vs Tail-based Sampling

**Head-based (decide at the start):**
- Decide at the first span whether to keep the trace.
- Cheap: You avoid work for dropped traces.
- Predictable cost (e.g., "keep 1% of all traces").
- But: You may **miss errors** or slow traces, because you decide before you know the result.

**Tail-based (decide at the end):**
- Collect whole trace first, then decide.
- Can always keep:
  - All error traces.
  - All slow traces.
  - Some normal traces.
- Better for debugging.
- But: More CPU + RAM + complexity (must buffer all spans until decision)

**What tools do:**
- Many SaaS (Datadog, New Relic) do clever mixed sampling internally.
- OpenTelemetry Collector supports tail-based sampling.
- Common strategy:
  - Head sample to reduce volume.
  - Tail-sample within that to always keep errors/slow traces.

---

## 4. Real-time vs Batch Processing

**Real-time (streaming) observability:**
- Data is processed as soon as it arrives.
- You see metrics and alerts within seconds.
- Needed for:
  - On-call alerts.
  - SLO monitoring.
  - User-facing SLAs.

**Batch processing:**
- Data is collected, then processed every X minutes/hours.
- Slower, but cheaper.
- Good for:
  - Daily/weekly reports.
  - Cost analysis.
  - Non-urgent analytics.

**Trade-off:**
- Real-time = **faster but more expensive and complex**
- Batch = **cheaper but delayed**.

**What tools do:**
- Core observability (metrics/alerts) is typically real-time.
- Heavy analytics, billing reports, long-term trends often done in batch.

---

## 5. Storage Optimization

### Compression vs Query Speed

- **More compression**:
  - Pros: Cheaper storage.
  - Cons: Slower queries (need to decompress).

- **Less compression**:
  - Pros: Faster queries.
  - Cons: Higher storage cost.

Most systems pick a **balanced default** and let you tune it.

### Downsampling Older Data

- New data: keep high resolution (e.g., 10s points).
- Old data: store summaries (e.g., 1min / 5min averages).

Example:
- Last 7 days: every 10 seconds.
- Last 30 days: every 1 minute.
- Last 1 year: every 10 minutes.

Saves lots of space while keeping trend visibility.

### Retention Policies

- Metrics: keep longer (cheap and compressed).
- Logs: keep shorter (expensive, noisy).
- Traces: keep shortest (very heavy).

Typical:
- Metrics: 6–24 months.
- Logs: 7–30 days (longer in cold storage).
- Traces: 1–7 days, sampled.

**What tools do:**
- Let you configure per-data-type retention.
- Offer archive tiers (S3/Glacier) for long-term cold storage.

---

## 6. Query Performance at Scale

**Goal:** Query TBs of data in a few seconds.

### Techniques

- **Indexing:**
  - Build indexes on time, service, status, etc.
  - Lets queries jump directly to relevant data.

- **Partitioning:**
  - Split data by time, tenant, region.
  - Avoid scanning everything for each query.

- **Pre-aggregation:**
  - Precompute common stats (e.g., hourly averages).
  - Fast queries but less flexible.

- **On-demand aggregation:**
  - Compute stats when you run the query.
  - More flexible but heavier.

- **File compaction:**
  - Combine many tiny files into fewer big files.
  - Greatly speeds up scans and reduces overhead.

**What tools do:**
- ClickHouse-based systems scan millions of rows in milliseconds.
- Elastic TSDS and others optimize for time-series + downsampling.
- S3-based systems compact small files into larger ones for faster queries.

---

## 7. Alert Fatigue

**Problem:** Too many alerts → people start ignoring ALL alerts.

Causes:
- Every small blip triggers an alert.
- Too many duplicate alerts.
- Same incident spams 10 different rules.
- No severity separation (minor = critical).

**Effects:**
- On-call burnout.
- Missed real incidents.
- People mute channels or ignore tools.

**Solutions (Smart Alerting):**
- Add **time tolerance** (only alert if bad for X minutes).
- Group related alerts into **one incident**.
- Use sensible **severities** (info / warn / critical).
- Regularly clean up noisy rules.
- Rotate on-call fairly.

**What tools do:**
- Deduplicate / group alerts into incidents.
- Allow "for" clauses (Prometheus) to avoid flapping.
- Provide alert analytics (who gets pinged how often).[258]

---

## 8. Context Correlation (Metrics + Logs + Traces)

**Goal:** Quickly answer:
"Metric says something is wrong. Logs and traces tell me **why**."

**Why hard?**
- Data comes from many systems.
- Different formats (metrics vs logs vs traces).
- Timestamps may not be perfectly aligned.
- Missing or inconsistent IDs (trace_id, span_id, user_id).

**What’s needed:**
- Shared identifiers:
  - `trace_id` in logs and traces.
  - `service.name`, `host`, `env` everywhere.
- Good time sync (NTP, etc.).
- Centralized places to search.

**What tools do:**
- Auto-inject trace IDs into logs.
- Provide unified views (one screen showing metrics + logs + traces).
- Allow clicking from metric → logs → trace → deployment event.

---

## Big Picture: Everything Is a Trade-off

There is **no perfect observability setup**. Every design is a balance:

- **More data** → better visibility, but higher cost + slower queries.
- **More labels** → more detail, but cardinality explosion risk.
- **More real-time** → faster detection, but higher infra + complexity.
- **More sampling** → cheaper, but you might miss rare issues.
- **More alerts** → more coverage, but more alert fatigue.

**What existing tools choose (roughly):**
- Datadog / New Relic:
  - Strong real-time + UX.
  - Store a lot (but expensive).
  - Heavy sampling + smart backend logic.
- Grafana + Prometheus / Loki / Tempo:
  - More DIY, more control.
  - You decide retention, sampling, storage.
  - Cheaper but needs engineering effort.
- ClickHouse-/S3-based stacks:
  - Store tons of raw data, push aggregation to query time.
  - Rely on very fast columnar engines + compression.

In practice, good teams:
- Start simple.
- Measure cost vs value.
- Iterate on:
  - What data to keep.
  - How long to keep it.
  - How often to sample.
  - How to alert without burning people out.

That balance **is** the core engineering challenge of observability.
