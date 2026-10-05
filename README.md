# dbloadlab
a self-hosted experimentation platform for learning and testing database, cache, and message-broker performance under controlled workloads.

# DataLab — Technical Specification

## 1. Product

**DataLab — Database Performance Experimentation Lab**

DataLab is a self-hosted platform for experimenting with **PostgreSQL, Redis, and Kafka** under controlled workloads.

The user manually performs tuning and architectural changes. DataLab provides:

* infrastructure/environment creation;
* configuration management;
* workload generation;
* live monitoring;
* experiment management;
* result comparison;
* configuration snapshots;
* experiment history.

**DataLab never automatically tunes the systems.**

---

# 2. Architecture

```text
                         DataLab Web UI
                              │
                              ▼
                         DataLab API
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
   Environment Manager   Workload Manager   Experiment Manager
          │                   │                   │
          └───────────────────┼───────────────────┘
                              │
                    Docker / Compose
                              │
              ┌───────────────┼───────────────┐
              │               │               │
         PostgreSQL         Redis           Kafka
              │               │               │
              └───────────────┼───────────────┘
                              │
                         Metrics Agent
                              │
                         Prometheus
                              │
                           Grafana
```

---

# 3. Web UI

Recommended stack:

```text
Frontend:
React
TypeScript
Vite

UI:
Tailwind CSS
shadcn/ui

Charts:
Apache ECharts

Backend:
Go

API:
REST + WebSocket
```

WebSocket is used for live experiment status and metrics.

---

# 4. Dashboard

Main dashboard:

```text
┌─────────────────────────────────────────────────────┐
│ DataLab                              [+ Experiment] │
├─────────────────────────────────────────────────────┤
│                                                     │
│  PostgreSQL       Redis             Kafka            │
│  ● Running        ● Running         ● Running        │
│  3 Nodes          6 Nodes           3 Brokers       │
│                                                     │
├─────────────────────────────────────────────────────┤
│ Active Experiments                                 │
│                                                     │
│ PostgreSQL Index Test              RUNNING          │
│ Redis Cluster Benchmark            COMPLETED        │
│ Kafka Partition Test               COMPLETED        │
│                                                     │
├─────────────────────────────────────────────────────┤
│ Current System Load                                │
│                                                     │
│ CPU      ████████████████  72%                     │
│ Memory   ███████████       54%                     │
│ TPS      18,421                                     │
│ P95      43 ms                                      │
└─────────────────────────────────────────────────────┘
```

---

# 5. Systems Page

```text
Systems

┌──────────────────────────────────────────┐
│ PostgreSQL                               │
│ Version: 17.6                            │
│ Nodes: 3                                 │
│ Status: ● Running                        │
│                                          │
│ [Open] [Start] [Stop] [Destroy]          │
└──────────────────────────────────────────┘

┌──────────────────────────────────────────┐
│ Redis                                    │
│ Version: 8.x                             │
│ Mode: Cluster                            │
│ Nodes: 6                                 │
│ Status: ● Running                        │
│                                          │
│ [Open] [Start] [Stop] [Destroy]          │
└──────────────────────────────────────────┘

┌──────────────────────────────────────────┐
│ Kafka                                    │
│ Brokers: 3                               │
│ Status: ● Running                        │
│                                          │
│ [Open] [Start] [Stop] [Destroy]          │
└──────────────────────────────────────────┘
```

---

# 6. Create Environment UI

User selects:

```text
System

○ PostgreSQL
○ Redis
○ Kafka
```

Then:

```text
Version:
[ PostgreSQL 17.6 ]

Nodes:
[ 3 ]

CPU per node:
[ 4 ]

RAM per node:
[ 8 GB ]

Storage:
[ 100 GB ]

[Create Environment]
```

For Redis:

```text
Mode:

○ Standalone
○ Replication
○ Sentinel
○ Cluster
```

For Kafka:

```text
Brokers:
[ 3 ]

Partitions:
[ 12 ]

Replication Factor:
[ 3 ]
```

---

# 7. PostgreSQL UI

PostgreSQL environment page:

```text
PostgreSQL

Topology
────────────────────────────

Primary
  │
  ├── Replica 1
  └── Replica 2


Performance
────────────────────────────

TPS       18,421
P50       12ms
P95       43ms
P99       91ms
CPU       72%
Memory    61%
IOPS      8,421
```

Tabs:

```text
Overview
Topology
Configuration
Queries
Indexes
Connections
Replication
Partitions
Metrics
Experiments
```

---

# 8. Configuration Editor

This is important because **you want to manually tune the database**.

UI:

```text
PostgreSQL Configuration

shared_buffers          8GB
work_mem                64MB
maintenance_work_mem   512MB
max_connections         500
random_page_cost        1.1
effective_cache_size     24GB
checkpoint_timeout       15min
max_wal_size             8GB

                         [Apply]
```

DataLab should show:

```text
Current value
Changed value
Unsaved changes
```

But it should **not recommend values**.

Example:

```text
shared_buffers

Current:
4GB

New:
8GB

[Apply Change]
```

After applying:

```text
Configuration changed.

Create experiment snapshot?

[Yes] [No]
```

---

# 9. Redis UI

Tabs:

```text
Overview
Topology
Configuration
Memory
Commands
Replication
Cluster
Metrics
Experiments
```

Cluster visualization:

```text
             Redis Cluster

        ┌──────────┐
        │ Node 1   │
        │ Master   │
        └────┬─────┘
             │
        ┌────▼─────┐
        │ Node 4   │
        │ Replica  │
        └──────────┘

        ┌──────────┐
        │ Node 2   │
        │ Master   │
        └────┬─────┘
             │
        ┌────▼─────┐
        │ Node 5   │
        │ Replica  │
        └──────────┘
```

---

# 10. Kafka UI

Tabs:

```text
Overview
Brokers
Topics
Partitions
Consumer Groups
Configuration
Metrics
Experiments
```

Topic view:

```text
orders

Partitions: 12
Replication Factor: 3

Partition    Leader    Replicas    Messages
0            1         1,2,3       8.2M
1            2         2,3,1       8.1M
2            3         3,1,2       8.4M
...
```

Consumer group:

```text
orders-consumer

Consumers: 5
Lag: 12,421

Partition    Consumer    Lag
0            consumer-1  120
1            consumer-2  1,820
...
```

---

# 11. Workload UI

Instead of only CLI, provide a visual workload builder.

### PostgreSQL

```text
Create Workload

Clients:
[500]

Duration:
[10 min]

Ramp-up:
[60 sec]

Dataset:
[100M rows]

Read / Write:
[80] / [20]

                    [Start]
```

### Redis

```text
Operations

GET       70%
SET       20%
INCR      10%

Clients:
1000

Pipeline:
10

Duration:
10 min

[Start]
```

### Kafka

```text
Producers:
[10]

Consumers:
[5]

Messages:
[10,000,000]

Message size:
[1 KB]

Compression:
[snappy]

[Start]
```

---

# 12. Live Experiment Screen

When an experiment is running:

```text
PostgreSQL Index Experiment
RUNNING ●

Elapsed: 04:32

TPS
20k ┤               ╭──────
15k ┤         ╭─────╯
10k ┤─────────╯
 5k ┤
    └──────────────────────

P95 Latency
200ms ┤──────╮
150ms ┤      ╰────╮
100ms ┤           ╰──────
 50ms ┤
      └───────────────────

CPU
██████████████████ 72%

IOPS
████████████ 48%

[Stop Experiment]
```

---

# 13. Experiment Comparison UI

This is one of the most important pages.

```text
Compare Experiments

          Baseline       Experiment #42

TPS        8,421             15,782
P50          42ms                18ms
P95         180ms                61ms
P99         420ms               130ms
CPU           91%                67%
IOPS        8,200              4,100
```

Charts:

```text
        TPS

20k ┤                     █
15k ┤                     █
10k ┤        █            █
 5k ┤        █            █
    └────────────────────────
       Before           After
```

The UI can calculate differences, but it should not say:

> "You should use this configuration."

The engineer decides.

---

# 14. Experiment Notes

Each experiment has a notes section:

```text
Experiment #42

Title:
PostgreSQL index test

Hypothesis:
Index should reduce random reads for user_id lookup.

Changes:
- Added idx_orders_user_id

Observation:
- P95 decreased significantly.
- CPU decreased.
- Index increased storage usage.

Conclusion:
Index is beneficial for this workload.
```

This makes DataLab useful as a **personal performance engineering notebook**.

---

# 15. Experiment History

```text
Experiments

#42 PostgreSQL Index       5 min ago
#41 PostgreSQL Partition   1 hour ago
#40 Redis Cluster          3 hours ago
#39 Kafka Partitions       yesterday
#38 PostgreSQL Replica     yesterday
```

Filters:

```text
System
Date
Status
Tags
Experiment type
```

---

# 16. Configuration Snapshots

Before/after each experiment:

```text
Snapshot #18

PostgreSQL 17.6

shared_buffers = 8GB
work_mem = 64MB
max_connections = 500
...

[View Diff]
```

Diff:

```diff
- shared_buffers = 4GB
+ shared_buffers = 8GB

- work_mem = 16MB
+ work_mem = 64MB
```

This is especially useful when you perform many tuning experiments.

---

# 17. UI Pages

Final UI structure:

```text
DataLab
│
├── Dashboard
│
├── Systems
│   ├── PostgreSQL
│   ├── Redis
│   └── Kafka
│
├── Workloads
│
├── Experiments
│   ├── Running
│   ├── History
│   └── Compare
│
├── Datasets
│
└── Settings
```

---

# 18. Backend

```text
Go
├── REST API
├── WebSocket
├── Environment Manager
├── Workload Manager
├── Experiment Manager
├── Dataset Manager
└── Metrics Manager
```

Database:

```text
PostgreSQL
```

Frontend:

```text
React
TypeScript
Vite
Tailwind
shadcn/ui
ECharts
```

Infrastructure:

```text
Docker
Docker Compose
```

Monitoring:

```text
Prometheus
Grafana
Exporters
```

---

# 19. Most Important Product Rule

```text
             ┌───────────────────────┐
             │       DataLab         │
             │                       │
             │ Deploy                │
             │ Configure             │
             │ Generate Load         │
             │ Monitor               │
             │ Record                │
             │ Compare               │
             └───────────┬───────────┘
                         │
                         ▼
                  YOU make decisions
```

**DataLab is your laboratory, not your tuning assistant.**

You decide:

* which index to create;
* which PostgreSQL parameter to change;
* how to partition;
* how to replicate;
* how to shard;
* how many Redis nodes to use;
* how to configure Kafka;
* what workload to run.

DataLab simply gives you a **repeatable environment and hard performance data** to test whether your decision actually worked.

Refine the DataLab UI

* Design the frontend architecture
* Define the MVP screens
