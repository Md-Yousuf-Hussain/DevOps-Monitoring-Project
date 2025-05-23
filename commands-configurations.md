
# Commands & Configurations

## VM-1 (Node Exporter)
1. Download Node Exporter
```
wget
https://github.com/prometheus/node_exporter/releases/download/v1.8.1/
node_exporter-1.8.1.linux-amd64.tar.gz
```
2. Extract Node Exporter
```
tar xvfz node_exporter-1.8.1.linux-amd64.tar.gz
```
3. Start Node Exporter
```
cd node_exporter-1.8.1.linux-amd64
./node_exporter &
```
## VM-2 (Prometheus, Alertmanager, Blackbox Exporter)
### Prometheus
1. Download Prometheus
```
wget
https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
```
2. Extract Prometheus
```
tar xvfz prometheus-2.52.0.linux-amd64.tar.gz
```
3. Start Prometheus
```
cd prometheus-2.52.0.linux-amd64
./prometheus --config.file=prometheus.yml &
```
### Alertmanager
1. Download Alertmanager
```
wget
https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
```
2. Extract Alertmanager
```
tar xvfz alertmanager-0.27.0.linux-amd64.tar.gz
```
3. Start Alertmanager
```
cd alertmanager-0.27.0.linux-amd64
./alertmanager --config.file=alertmanager.yml &
```
### Blackbox Exporter
1. Download Blackbox Exporter
```
wget
https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
```
2. Extract Blackbox Exporter
```
tar xvfz blackbox_exporter-0.25.0.linux-amd64.tar.gz
```
3. Start Blackbox Exporter
```
cd blackbox_exporter-0.25.0.linux-amd64
./blackbox_exporter 
```
## Prometheus and Alertmanager Configuration
### Prometheus Configuration (prometheus.yml)
### Global Configuration
```
global:
  scrape_interval: 15s        # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s    # Evaluate rules every 15 seconds. Default is every 1 minute.
  # scrape_timeout is set to the global default (10s).
``` 
### Alertmanager Configuration
```
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - "localhost:9093"   # Alertmanager endpoint

rule_files:
  - "alert_rules.yml"          # Path to alert rules file
  # - "second_rules.yml"      # Additional rule files can be added here
```
### Scrape Configuration
```
Prometheus Itself
scrape_configs:
  - job_name: "prometheus"       # Job name for Prometheus
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'
    static_configs:
      - targets:
          - "localhost:9090"      # Target to scrape (Prometheus itself)
```
### Node Exporter
```
scrape_configs:
  - job_name: "node_exporter"       # Job name for Node Exporter
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'
    static_configs:
      - targets:
          - "3.110.195.114:9100"     # Target Node Exporter endpoint
```
### BlackBox Exporter
```
scrape_configs:
  - job_name: "blackbox"            # Job name for Blackbox Exporter
    metrics_path: /probe            # Path for Blackbox probe
    params:
      module: [http_2xx]            # Module to look for HTTP 200 responses
    static_configs:
      - targets:
          - "http://prometheus.io"            # HTTP target
          - "https://prometheus.io"           # HTTPS target
          - "http://3.110.195.114:8080/"      # HTTP target with port 8080
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: "13.235.248.225:9115"    # Blackbox exporter address
```
## Alert Rules Configuration (alert_rules.yml)
### Alert Rules Group
```
groups:
  - name: alert_rules           # Name of the alert rules group
    rules:
      - alert: InstanceDown
        expr: up == 0           # Expression to detect instance down
        for: 1m
        labels:
          severity: "critical"
        annotations:
          summary: "Endpoint {{ $labels.instance }} down"
          description: |
            {{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute.

      - alert: WebsiteDown
        expr: probe_success == 0 # Expression to detect website down
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Website down"
          description: "The website at {{ $labels.instance }} is down."

      - alert: HostOutOfMemory
        expr: (node_memory_MemAvailable / node_memory_MemTotal * 100) < 25
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Host out of memory (instance {{ $labels.instance }})"
          description: |
            Node memory is filling up (< 25% left)
            VALUE = {{ $value }}
            LABELS: {{ $labels }}

      - alert: HostOutOfDiskSpace
        expr: (node_filesystem_avail{mountpoint="/"} * 100) / node_filesystem_size{mountpoint="/"} < 50
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "Host out of disk space (instance {{ $labels.instance }})"
          description: |
            Disk is almost full (< 50% left)
            VALUE = {{ $value }}
            LABELS: {{ $labels }}

      - alert: HostHighCpuLoad
        expr: sum by (instance)(irate(node_cpu{job="node_exporter_metrics",mode="idle"}[5m])) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Host high CPU load (instance {{ $labels.instance }})"
          description: |
            CPU load is > 80%
            VALUE = {{ $value }}
            LABELS: {{ $labels }}

      - alert: ServiceUnavailable
        expr: up{job="node_exporter"} == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Service Unavailable (instance {{ $labels.instance }})"
          description: |
            The service {{ $labels.job }} is not available
            VALUE = {{ $value }}
            LABELS: {{ $labels }}

      - alert: HighMemoryUsage
        expr: (node_memory_Active / node_memory_MemTotal * 100) > 90
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "High Memory Usage (instance {{ $labels.instance }})"
          description: |
            Memory usage is > 90%
            VALUE = {{ $value }}
            LABELS: {{ $labels }}

      - alert: FileSystemFull
        expr: (node_filesystem_avail / node_filesystem_size * 100) < 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "File System Almost Full (instance {{ $labels.instance }})"
          description: |
            File system has < 10% free space
            VALUE = {{ $value }}
            LABELS: {{ $labels }}
```
## Alertmanager Configuration (alertmanager.yml)
### Routing Configuration
```
route:
  group_by: ['alertname']       # Group alerts by alert name
  group_wait: 30s               # Wait time before sending the first notification
  group_interval: 5m            # Interval between notifications for new alerts
  repeat_interval: 1h           # Interval to resend notifications for ongoing alerts
  receiver: 'email-notifications'  # Default receiver

receivers:
  - name: 'email-notifications'   # Receiver name
    email_configs:
      - to: 'jaiswaladi246@gmail.com'     # Email recipient
        from: 'test@gmail.com'            # Email sender
        smarthost: 'smtp.gmail.com:587'   # SMTP server and port
        auth_username: 'your_email'       # SMTP auth username
        auth_identity: 'your_email'       # SMTP auth identity
        auth_password: 'your_pass'  # SMTP auth password
        send_resolved: true               # Send notifications when alerts resolve
```
### Inhibition Rules
```
inhibit_rules:
  - source_match:
      severity: "critical"          # Source alert severity
    target_match:
      severity: "warning"           # Target alert severity
    equal: ["alertname", "dev", "instance"]  # Fields to match for inhibition
```
