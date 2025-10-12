# Monitoring & Observability Setup Guide

## Metrics to Monitor

### Application Metrics
- Request rate (requests/sec)
- Error rate (errors/sec, %)
- Latency (P50, P95, P99)
- Active connections
- Database query performance
- Cache hit/miss rate

### Infrastructure Metrics
- CPU utilization (%)
- Memory utilization (%)
- Network I/O (bytes/sec)
- Disk I/O (IOPS, bytes/sec)
- Container/Pod restarts

### Business Metrics
- User registrations
- Active users
- Revenue metrics
- Conversion rates

## Alert Levels

### P0 (Critical) - Immediate Action
- Service down (all health checks failing)
- Error rate > 5%
- Database connection failure
- **Action**: Page on-call engineer

### P1 (Warning) - Monitor Closely
- CPU > 80% for 10 minutes
- Memory > 85% for 10 minutes
- Latency P95 > 1 second
- **Action**: Notify team channel

### P2 (Info) - Awareness
- Deployment notifications
- Scaling events
- **Action**: Log only

## Prometheus + Grafana Setup

### Prometheus Configuration
```yaml
scrape_configs:
  - job_name: 'backend'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

### Grafana Dashboards
1. Application Dashboard - Request rate, errors, latency
2. Infrastructure Dashboard - CPU, memory, network
3. Business Dashboard - User metrics, revenue

## CloudWatch Alarms (AWS)

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu \
  --metric-name CPUUtilization \
  --namespace AWS/ECS \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold
```

## Log Aggregation

### Structured Logging Format
```json
{
  "timestamp": "2024-01-15T10:30:45Z",
  "level": "info",
  "message": "User login",
  "user_id": "12345",
  "trace_id": "abc123"
}
```

### Log Retention
- Production: 30 days
- Staging: 14 days
- Development: 7 days
