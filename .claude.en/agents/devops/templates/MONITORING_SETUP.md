# Monitoring Setup Guide

## Metrics Collection

### Application Metrics
- Request rate, error rate, latency
- Database query performance
- Cache effectiveness

### Infrastructure Metrics
- CPU, memory, network, disk
- Container/pod health
- Auto-scaling events

## Alerting Rules

### Critical (P0)
- Service down → Page on-call
- Error rate > 5% → Page on-call
- DB connection failed → Page on-call

### Warning (P1)
- CPU > 80% → Slack notification
- Memory > 85% → Slack notification
- Latency P95 > 1s → Slack notification

## Dashboards

1. Application Dashboard
2. Infrastructure Dashboard
3. Business Metrics Dashboard

## Log Aggregation

- Kubernetes: Fluent Bit → CloudWatch
- ECS: awslogs driver → CloudWatch
- Retention: 30 days (production)
