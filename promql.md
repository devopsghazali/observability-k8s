# PromQL Queries for Kubernetes Monitoring

This document contains useful PromQL queries for monitoring Kubernetes clusters using Prometheus and Grafana.
These queries help detect resource pressure, workload instability, and infrastructure issues.

---

# 1. Node CPU Usage

```promql
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

**Explanation**

This query calculates the percentage of CPU usage on each node.
Prometheus tracks CPU time spent in different modes such as idle, system, and user.
By measuring the idle CPU rate and subtracting it from 100%, we get the actual CPU utilization.

**Use Case**

Detect nodes that are experiencing high CPU load.

---

# 2. Node Memory Usage

```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) 
/ node_memory_MemTotal_bytes * 100
```

**Explanation**

This query calculates memory usage percentage.

* `MemTotal` represents total memory on the node.
* `MemAvailable` represents memory that can still be used.

The difference represents used memory.

**Use Case**

Identify nodes that are approaching memory exhaustion.

---

# 3. Pod Restart Count

```promql
sum by(pod) (kube_pod_container_status_restarts_total)
```

**Explanation**

This query shows the total number of restarts for each pod.

Frequent restarts may indicate:

* application crashes
* misconfiguration
* resource exhaustion

**Use Case**

Detect unstable or failing workloads.

---

# 4. Pod CPU Usage

```promql
sum(rate(container_cpu_usage_seconds_total{container!=""}[5m])) by (pod)
```

**Explanation**

This query calculates CPU usage per pod.

`container_cpu_usage_seconds_total` tracks total CPU time consumed by containers.
The `rate()` function calculates how quickly CPU usage increases over time.

**Use Case**

Identify CPU-intensive pods.

---

# 5. Pod Memory Usage

```promql
sum(container_memory_usage_bytes{container!=""}) by (pod)
```

**Explanation**

This query returns memory usage per pod.

It helps detect workloads consuming excessive memory.

**Use Case**

Troubleshooting memory-heavy applications.

---

# 6. Kubernetes Node Ready Status

```promql
kube_node_status_condition{condition="Ready",status="true"}
```

**Explanation**

This query checks if nodes are in a healthy `Ready` state.

If a node becomes `NotReady`, workloads may stop scheduling or existing pods may fail.

**Use Case**

Cluster health monitoring.

---

# 7. API Server Request Rate

```promql
rate(apiserver_request_total[5m])
```

**Explanation**

This query measures how frequently requests are being sent to the Kubernetes API server.

A sudden spike in requests may indicate:

* heavy cluster automation
* misconfigured controllers
* system stress

**Use Case**

Cluster performance monitoring.

---

# 8. Network Traffic per Node

```promql
rate(node_network_receive_bytes_total[5m])
```

**Explanation**

This query calculates incoming network traffic on each node.

Since the metric is cumulative, `rate()` converts it into traffic per second.

**Use Case**

Detect network spikes or abnormal traffic patterns.

---

# 9. Filesystem Usage Percentage

```promql
(node_filesystem_size_bytes - node_filesystem_free_bytes) 
/ node_filesystem_size_bytes * 100
```

**Explanation**

This query calculates disk usage percentage for node filesystems.

Disk pressure can lead to:

* pod scheduling failures
* node instability

**Use Case**

Disk utilization monitoring.

---

# 10. HTTP Request Rate (Application Metric)

```promql
rate(http_requests_total[5m])
```

**Explanation**

This query measures the rate of HTTP requests received by an application.

This metric helps analyze:

* traffic spikes
* service load
* request patterns

**Use Case**

Application performance monitoring.

---

# Cluster Level CPU Usage

```promql
avg(rate(node_cpu_seconds_total{mode!="idle"}[5m])) * 100
```

**Explanation**

This query calculates the average CPU usage across the entire cluster.

**Use Case**

Understand overall cluster load and infrastructure health.

---

# Example Alert Query

```promql
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
```

**Explanation**

This condition triggers an alert when CPU usage exceeds 80%.

**Use Case**

Early detection of node overload conditions.
