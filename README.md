## Cluster Summary
- running nodes with count and highlighted in green color
- pending nodes with count and highlighted in yellow color
- dead nodes with count and highlighted in red color
- Allocated CPU 
- Allocated Memory
- individual Node status with ip address and healthy or unhealthy status in highlighted color
## Node status
- uptime with node name, number of days & hours
- free memory with node name, percentage
- free disk space with node name, percentage
- unalocated mem
- unalocated cpu
- allocation status

------
1. Cluster Summary Dashboard
Node Status Overview:

#### Running Nodes (Ready)
```bash
count(kube_node_status_condition{condition="true", type="Ready"})
```
Pending Nodes
```bash
count(kube_node_status_condition{condition="false", type="Ready"})
```

Dead Nodes (NotReady)

```bash
count(kube_node_status_condition{condition="false", type="Ready"})
```

CPU Allocation (Total Requested CPU)

```bash
sum(kube_pod_container_resource_requests_cpu_cores)
```

Memory Allocation (Total Requested Memory)
```bash
sum(kube_pod_container_resource_requests_memory_bytes)
```

Pod Status Overview (Running Pods)
```bash
count(kube_pod_status_phase{phase="Running"})
```

Pending Pods
```bash
count(kube_pod_status_phase{phase="Pending"})
```

Failed Pods
```bash
count(kube_pod_status_phase{phase="Failed"})
```

2. Node Status Dashboard
Node Uptime:
```bash
Node Uptime (Last Reboot Time)
```

```bash
(time() - node_time_seconds) / 60 / 60 / 24
```
Free Memory:
```bash
node_memory_MemFree_bytes / node_memory_MemTotal_bytes * 100
```

Free Disk Space:
```bash
node_filesystem_avail_bytes / node_filesystem_size_bytes * 100
```
Unallocated Memory:
Unallocated Memory (Memory not used by containers)
```bash
(node_memory_MemFree_bytes + node_memory_Buffers_bytes + node_memory_Cached_bytes) / node_memory_MemTotal_bytes * 100
```
Unallocated CPU:
Unallocated CPU (CPU not in use)
```bash
sum(node_cpu_seconds_total{mode="idle"}) by (instance) / sum(node_cpu_seconds_total{mode="user",mode="system"}) by (instance) * 100
```
Node Allocation Status:
CPU Allocation vs Usage
```bash
sum(kube_pod_container_resource_requests_cpu_cores) by (node) / sum(node_cpu_seconds_total{mode="user",mode="system"}) by (instance)
```


3. Resource Usage and Efficiency
CPU Utilization vs Allocation:
```bash
(sum(rate(container_cpu_usage_seconds_total{job="kubelet", cluster="your_cluster", container!="", image!="", instance=~".+"}[5m])) by (node)) / sum(kube_pod_container_resource_requests_cpu_cores) by (node) * 100
```
Memory Utilization vs Allocation:
```bash
(sum(container_memory_usage_bytes{job="kubelet", cluster="your_cluster", container!="", image!="", instance=~".+"}) by (node)) / sum(kube_pod_container_resource_requests_memory_bytes) by (node) * 100
```

Node Resource Pressure (CPU and Memory Usage):
CPU Resource Pressure
```bash
sum(rate(container_cpu_usage_seconds_total{job="kubelet", mode="user", cluster="your_cluster"}[5m])) by (node) / sum(node_cpu_seconds_total{mode="user",mode="system"}) by (instance) * 100
```
Memory Resource Pressure
```bash
sum(container_memory_usage_bytes{job="kubelet", cluster="your_cluster"}) by (node) / sum(node_memory_MemTotal_bytes) by (instance) * 100
```


4. Pod-Level Metrics Dashboard
Pod Count by Namespace:
Pods by Namespace
```bash
count(kube_pod_status_phase{phase="Running"}) by (namespace)
```

Pod Restarts:
Pod Restarts (Over time)
```bash
rate(kube_pod_container_status_restarts_total[5m])
```

Pod Resource Requests vs Usage:
Pod CPU and Memory Requests vs Usage
CPU Usage vs Requests:
```bash
sum(rate(container_cpu_usage_seconds_total{job="kubelet", cluster="your_cluster", container!="", image!="", instance=~".+"}[5m])) by (pod) / sum(kube_pod_container_resource_requests_cpu_cores) by (pod)
```

Memory Usage vs Requests:
```bash
sum(container_memory_usage_bytes{job="kubelet", cluster="your_cluster", container!="", image!="", instance=~".+"}) by (pod) / sum(kube_pod_container_resource_requests_memory_bytes) by (pod)
```

Pod Scheduling Issues:
Pods in Pending State for More than 5 Minutes
```bash
count(kube_pod_status_phase{phase="Pending"}) by (pod)
```

5. Alerts & Notifications
Node Not Ready Alert:

Trigger an alert if a node enters "NotReady" state.
```bash
kube_node_status_condition{condition="false", type="Ready"} == 1
```

CPU/Memory Usage Threshold Alert (e.g., CPU > 90%):

Trigger alert if CPU usage exceeds 90%.
```bash
sum(rate(container_cpu_usage_seconds_total{job="kubelet", cluster="your_cluster", container!="", image!="", instance=~".+"}[5m])) by (node) / sum(node_cpu_seconds_total{mode="user",mode="system"}) by (instance) * 100 > 90
```

Memory Usage Alert (e.g., Memory > 85%):
```bash
sum(container_memory_usage_bytes{job="kubelet", cluster="your_cluster"}) by (node) / sum(node_memory_MemTotal_bytes) by (instance) * 100 > 85
```

Pod CrashLoopBackOff Alert:
```bash
kube_pod_container_status_waiting{reason="CrashLoopBackOff"} == 1
````
Final Notes:
Adjust Cluster/Instance Names: The queries above reference generic cluster or instance names like your_cluster. Ensure to replace these with the actual names or labels used in your Prometheus setup.
Unit Adjustments: Depending on your setup, you may need to adjust the units (e.g., for memory and CPU). Prometheus stores these metrics in raw formats, so sometimes conversion is necessary (e.g., bytes to megabytes).
Alert Thresholds: Customize alert thresholds as per your environment's needs, based on resource capacity, scaling behavior, and fault tolerance.
Grafana Visualizations: Use appropriate Grafana visualization types (e.g., heatmaps, time series, gauges) based on the nature of the metrics for better clarity.
These Prometheus queries will provide the foundation for your Grafana dashboards, helping to monitor the health and performance of your Kubernetes cluster and its nodes effectively