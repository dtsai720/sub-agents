# Deployment Checklist

## Pre-Deployment

### Infrastructure
- [ ] VPC and subnets configured
- [ ] Security groups configured
- [ ] Load balancer provisioned
- [ ] Database created
- [ ] IAM roles configured

### Secrets
- [ ] Database credentials in Secrets Manager
- [ ] API keys stored securely
- [ ] SSL certificates provisioned

### Container Images
- [ ] Images built and pushed to registry
- [ ] Images scanned for vulnerabilities
- [ ] Image tags follow versioning strategy

## Deployment

### Application
- [ ] Task definitions/deployments applied
- [ ] Services created/updated
- [ ] Health checks passing
- [ ] Auto scaling configured

### Network & Security
- [ ] DNS records updated
- [ ] SSL/TLS certificates attached
- [ ] Security groups allow required traffic
- [ ] Network policies applied (K8s)

### Monitoring
- [ ] CloudWatch Logs configured
- [ ] CloudWatch Alarms created
- [ ] Prometheus/Grafana configured
- [ ] Log retention policies set

## Post-Deployment

### Verification
- [ ] Health endpoints responding (200 OK)
- [ ] API endpoints functional
- [ ] Frontend loads correctly
- [ ] Database connectivity verified

### Performance
- [ ] Response times within SLA (< 500ms)
- [ ] Memory usage within limits
- [ ] CPU usage within limits

### Security
- [ ] SSL/TLS valid
- [ ] Authentication working
- [ ] Authorization working
- [ ] No sensitive data in logs

## Rollback Plan

- [ ] Rollback command documented
- [ ] Rollback tested in staging
- [ ] Communication plan ready
