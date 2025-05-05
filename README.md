
# Website Monitoring System: Event-based Gmail Alerts

In this project, we will implement a comprehensive monitoring solution using Prometheus and various exporters to ensure the reliability and performance of a web application hosted on an AWS EC2 instances. 

This setup will include:
- Node Exporter for hardware and OS metrics
- Blackbox Exporter for probing endpoints
- Alertmanager for handling alerts
- Gmail integration will also configured to receive notifications for critical alerts


## Project Components and Best Practices

### Components
1. EC2 Instances:
- Instance 1: Hosts the web application, Node Exporter, and Nginx.
- Instance 2: Hosts Prometheus, Blackbox Exporter, and Alertmanager.

2. Prometheus: Centralized monitoring tool for collecting and querying metrics.

3. Node Exporter: Collects hardware and OS-level metrics from the web server.

4. Blackbox Exporter: Probes endpoints to monitor uptime and response time.

5. Alertmanager: Manages alerts sent by Prometheus based on defined
rules.

6. Gmail Integration: Sends email notifications for critical alerts.

### Alerts
1. InstanceDown: Triggers when an instance is down for more than 1
minute.

2. WebsiteDown: Triggers when the website is down for more than 1
minute.

3. HostOutOfMemory: Triggers when available memory is less than 25%
for more than 5 minutes.

4. HostOutOfDiskSpace: Triggers when the root filesystem has less than 50% available space.

5. HostHighCpuLoad: Triggers when CPU load is above 80% for more than 5 minutes.

6. ServiceUnavailable: Triggers when the node exporter service is
unavailable for more than 2 minutes.

7. HighMemoryUsage: Triggers when memory usage exceeds 90% for
more than 10 minutes.

8. FileSystemFull: Triggers when any filesystem has less than 10% available space for more than 5 minutes.

### Best Practices
1. Define Clear Objectives and Metrics
- Identify Key Metrics: Determine which metrics are critical for the health and performance of your application and infrastructure (e.g., CPU usage, memory usage, response times, error rates).
- Set Baselines and Thresholds: Establish baseline performance levels and set thresholds for alerts to distinguish between normal and abnormal behavior.

2. Use Multiple Data Sources
- Combine Metrics and Logs: Use both metrics and logs to get a comprehensive view of the system's health.
- Integrate Various Exporters: Use relevant exporters (Node Exporter, Blackbox Exporter, etc.) to collect metrics from different parts of your infrastructure.

3. Implement Robust Alerting
- Define Relevant Alerting Rules: Create alerting rules that cover various failure scenarios and performance degradation.
- Avoid Alert Fatigue: Ensure alerts are actionable and avoid too many false positives. Group related alerts to reduce noise.
- Use Multiple Notification Channels: Configure alerts to be sent via multiple channels (email, SMS, chat tools) to ensure they are noticed.

4. Ensure High Availability and Redundancy
- Deploy Across Multiple Regions: Set up monitoring components in multiple regions to avoid single points of failure.
- Backup and Replicate Data: Regularly back up Prometheus data and configuration files. Use replication to ensure data availability.

5. Optimize Performance and Resource Usage
- Tune Scrape Intervals and Retention Policies: Set appropriate scrape intervals and data retention policies to balance between data granularity and resource usage.
## Documentation

### Phase 1: Initial Setup
- Provision EC2 Instances
    - Launch two EC2 instances in any preferred region
        - Name those instances as MonitoringVM & WebsiteVM
    - Configure security groups to allow necessary traffic;
        - SMTP @ port 25
        - Custom TCP @ port 3000-10000
        - HTTP @ port 80
        - HTTPS @ port 443
        - Custom TCP @ port 587
        - SSH @ port 22
        - Custom TCP @ port 6443
        - Custom TCP @ port 30000 - 32767
        - SMTPS @ port 465
        - Custom TCP @ port 27017
    - Ensure the following requirements;
        - Instance Type = t2.medium
        - vCPUs = 2
        - Memory = 8GB
        - Network Performance = Moderate

### Phase 2: Installation and Configuration

SSH into MonitoringVM and update it
#### Prometheus Installation
- Download Prometheus into the server
- Extract the Downloaded .tar file
- Rename the extracted file as "prometheus" 
#### BlackBox Exporter Installation
- Download BlackBox Exporter into the server
- Extract the Downloaded .tar file
- Rename the extracted file as "blackBox_Exporter"
#### Alert Manager Installation
- Download Alert Manager into the server
- Extract the Downloaded .tar file
- Rename the extracted file as "alert_Manager"

SSH into WebsiteVM and update it
#### Node Exporter Installation
- Clone your website into the server
- Download Node Exporter into the server
- Extract the downloaded .tar file
- Rename the extracted file as "nodeExporter"
- Execute the node file and run it in background
- Install OpenJDK
- Install Maven and "mvn package"

#### Configure tools
- Return to MonitoringVM
- Execute the prometheus file, and ensure that the service is running on port 9090
- Create a alert_rules.yml file and write/paste your alert_rules
- Configure prometheus.yml and map the IP to our WebsiteVM @ port 9093
- Write/paste configuration targets
- Restart prometheus

