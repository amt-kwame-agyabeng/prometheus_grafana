## Setup Instructions

This project can be implemented on any Linux-based Server or local machine. However, this instruction assumes that the server is an AWS EC2 instance running on ubuntu 24.04

### Prerequisite

- AWS EC2 instance running on ubuntu with git and docker installed.
- Basic knowledge of docker, linux cli, prometheus, and grafana

### Sever Setup (Containerized Prometheus, Grafana, and Node Exporter)

- ssh into your EC2 instance
- run ```git clone https://github.com/amt-kwame-agyabeng/prometheus_grafana```
- run ```sudo docker compose up -d```
- Verify that your containers are running by ```sudo docker compose ps``` You will see a results similar to this:

```bash
NAME            IMAGE                        COMMAND                  SERVICE         CREATED       STATUS       PORTS
grafana         grafana/grafana-oss:latest   "/run.sh"                grafana         2 hours ago   Up 2 hours   0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp
node-exporter   prom/node-exporter:latest    "/bin/node_exporter …"   node-exporter   2 hours ago   Up 2 hours   0.0.0.0:9100->9100/tcp, [::]:9100->9100/tcp
prometheus      prom/prometheus:latest       "/bin/prometheus --c…"   prometheus      2 hours ago   Up 2 hours   0.0.0.0:9090->9090/tcp, [::]:9090->9090/tcp
```

- Open the Prometheus Client in your browser ```http://your-instance-ip:9090/targets``` 

- Open the Grafana Dashboard in your browser ```http://your-instance-ip:3000``` 

  - Enter ```admin``` as both the username and password.
  - Update your password when prompted.






## Question 1: Container Orchestration Analysis
Examine the Docker Compose configuration used in this lab. Explain why the Node Exporter container requires mounting host directories (/proc, /sys, /) and what would happen if these mounts were removed or configured incorrectly. How does this design decision reflect the principle of containerized monitoring?


Answer - Node Exporter container requires mount host directory if mounts were removed or misconfigured node exporter would run but only see the container metrics.


## Question 2: Network Security Implications
The lab creates a custom Docker network named "prometheus-grafana". Analyze the security implications of this setup versus using the default Docker network. What potential vulnerabilities could arise from the current port exposure configuration (9090, 9100, 3000), and how would you modify the setup for a production environment?

Answer -  We can modify the setup for a production environment by using reverse proxy and limit access to our only trusted IP address

## Question 3: Data Persistence Strategy
Compare the volume mounting strategies used for Prometheus data versus Grafana data in the Docker Compose file. Explain why different approaches might be needed for different components and what would happen to your dashboards and historical metrics if you removed these volume configurations. 

Answer - If volume configurations are remove then all metrics will be lost and dashboards will not be available they become ephemeral thus data or metrics are lost when shutdown or stopped.

## Section 2: Metrics and Query Understanding 

## Question 4: PromQL Logic Breakdown
The tutorial uses this query for calculating uptime: node_time_seconds - node_boot_time_seconds
Explain step-by-step what each metric represents, why subtraction gives us uptime, and what potential issues could arise with this calculation method. Propose an alternative approach and justify when you might use it instead.

Answer -  The node_time_seconds shows current time in second on the monitor node , node_boot_time_seconds is the timestamp where the system last boot time, subtraction could lead to inconsistent uptime values when node exporter is restarted use time () + 



## Question 5: Memory Metrics Deep Dive
The lab uses node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes to calculate memory usage. Research and explain why this approach is preferred over using node_memory_MemFree_bytes. What's the fundamental difference between "free" and "available" memory in Linux systems, and how does this impact monitoring accuracy?


Answer -  Using node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes gives a more realistic picture of memory usage than subtracting MemFree.
Why?

MemFree reflects only completely unused memory.

MemAvailable includes reclaimable caches and buffers, showing memory that can actually be repurposed.
Linux optimizes performance by caching aggressively, so MemFree often appears low—even when ample memory is available.
Impact: Using MemFree can trigger false low-memory alerts. In practice, monitoring based on MemAvailable avoids unnecessary concerns and better reflects real resource availability.


      


## Question 6: Filesystem Query Analysis
1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})
Break down the mathematical logic, explain why the result needs to be subtracted from 1, and discuss what could go wrong if you monitoring multiple mount points with this approach. How would you modify this query to exclude temporary filesystems?



## Section 3: Visualization and Dashboard Design
Question 7: Visualization Type Justification
The tutorial uses three different visualization types: Stat, Time Series, and Gauge. For each visualization created in the lab, justify why that specific type was chosen over alternatives. What criteria should guide visualization selection, and when might your choices be suboptimal?

Answer - State is used for showing the current state of the system, time series is used for showing the trend of the system, gauge is used for showing the progress of the system. The criteria for choosing the visualization type is the type of data we are trying to show and the type of information we are trying to convey. 

## Question 8: Threshold Configuration Strategy
Explain the reasoning behind the 80% threshold setting for disk usage in the gauge chart. Research industry standards and propose a more sophisticated alerting strategy that considers different types of systems (database servers, web servers, etc.). How would you implement multi-level thresholds that provide actionable insights?

## Question 9: Dashboard Variable Implementation
The tutorial introduces dashboard variables using the $job variable. Explain how this variable system works internally in Grafana, what happens when you have multiple values for a variable, and design a scenario where poorly implemented variables could break your dashboard. How would you test variable robustness?

## Section 4: Production and Scalability Considerations
Question 10: Resource Planning and Scaling
Based on your lab experience, calculate the approximate resource requirements (CPU, memory, storage) for monitoring 100 servers using this setup. Consider metric ingestion rates, retention periods, and dashboard query load. What bottlenecks would you expect to encounter first, and how would you address them?

## Question 11: High Availability Design
The current lab setup is single-node. Design a high-availability architecture for this monitoring stack that can handle component failures. Explain your approach to data consistency, load balancing, and disaster recovery. What trade-offs would you make between complexity and reliability?

## Question 12: Security Hardening Analysis
Identify at least five security vulnerabilities in the lab setup and propose specific remediation strategies. Consider authentication, authorization, network security, and data protection. How would you implement secrets management and secure communication between components?
      
## Section 5: Troubleshooting and Operations

Question 13: Debugging Methodology
Describe a systematic approach to troubleshooting when Prometheus shows a target as "DOWN" in the targets page. Walk through the diagnostic steps you would take, including command-line tools, log analysis, and configuration verification. What are the most common causes and their solutions?

## Question 14: Performance Optimization

After running the lab, analyze the query performance of your dashboards. Identify which queries might be expensive and explain why. Propose optimization strategies for both PromQL queries and Grafana dashboard design. How would you monitor the monitoring system itself?

Costly queries I've recognized: # This is costly - computes across all time series rate(node_cpu_seconds_total[5m]) 
 Improved - more precise rate(node_cpu_seconds_total{mode="idle"}[5m]) 
 Tactics for enhancing query performance: 
 • Employ recording regulations for intricate computations 
 • Restrict time intervals in searches 
 • Utilize particular label selectors 
 • Steer clear of functions that necessitate a complete series scan. 
 Dashboard enhancement: 
 • Decrease update frequency for unchanging data 
 • Implement query caching 
 • Restrict simultaneous queries for each dashboard 
 • Aggregate data in advance using recording rules
Overseeing the monitoring system: 
Performance metrics of Prometheus 
- prometheus_rule_evaluation_time_seconds 
- prometheus_tsdb_compaction_time_seconds up{job="prometheus"}

## Question 15: Capacity Planning Scenario
You notice that Prometheus is consuming increasing amounts of disk space over time. Analyze the factors that contribute to storage growth, calculate retention policies based on business requirements, and design a data lifecycle management strategy. How would you balance historical data availability with resource constraints?

 Active series count - Additional servers result in more metrics. 
 2 Scrape interval - 15s versus 60s increases storage requirements fourfold. 
 3 Retention duration - 30 days versus 365 days 
 4 Series cardinality - Labels with high cardinality significantly increase storage requirements. 
 Calulation of retention policy: Daily storage = (active_series × samples_per_day × bytes_per_sample) Total storage = daily_storage × retention_days × compression_factor Data lifecycle management approach:
• Tier 1 (0-7 days): Rapid SSD storage with high resolution 
• Tier 2 (7-30 days): Regular storage, downsampled • Tier 3 (30+ days): Significantly compressed, cold storage 
• Archive (1+ years): Transfer to data lake for regulatory adherence Equilibrating historical information against resources: 
• Utilize recording rules to pre-compute essential queries 
• Transfer essential metrics to permanent storage 
• Routine removal of unnecessary metrics