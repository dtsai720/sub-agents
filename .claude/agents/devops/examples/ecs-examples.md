# AWS ECS/Fargate Deployment Examples

本文件提供 AWS ECS 與 Fargate 的完整配置範例。

---

## Backend Task Definition

`ecs/backend-task-definition.json`:

```json
{
  "family": "backend",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::<ACCOUNT_ID>:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::<ACCOUNT_ID>:role/backendTaskRole",
  "containerDefinitions": [
    {
      "name": "backend",
      "image": "<ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/backend:latest",
      "cpu": 512,
      "memory": 1024,
      "essential": true,
      "portMappings": [
        {
          "containerPort": 8080,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "PORT",
          "value": "8080"
        },
        {
          "name": "LOG_LEVEL",
          "value": "info"
        },
        {
          "name": "ENVIRONMENT",
          "value": "production"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:<REGION>:<ACCOUNT_ID>:secret:backend/database-url"
        },
        {
          "name": "REDIS_URL",
          "valueFrom": "arn:aws:secretsmanager:<REGION>:<ACCOUNT_ID>:secret:backend/redis-url"
        },
        {
          "name": "JWT_SECRET",
          "valueFrom": "arn:aws:secretsmanager:<REGION>:<ACCOUNT_ID>:secret:backend/jwt-secret"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/backend",
          "awslogs-region": "<REGION>",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": [
          "CMD-SHELL",
          "wget --quiet --tries=1 --spider http://localhost:8080/health || exit 1"
        ],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 10
      }
    }
  ]
}
```

---

## Backend Service Definition

`ecs/backend-service.json`:

```json
{
  "serviceName": "backend",
  "cluster": "production-cluster",
  "taskDefinition": "backend",
  "desiredCount": 3,
  "launchType": "FARGATE",
  "platformVersion": "LATEST",
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": [
        "subnet-xxxxxxxx",
        "subnet-yyyyyyyy"
      ],
      "securityGroups": [
        "sg-xxxxxxxxx"
      ],
      "assignPublicIp": "DISABLED"
    }
  },
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:<REGION>:<ACCOUNT_ID>:targetgroup/backend-tg/xxxxxxxxx",
      "containerName": "backend",
      "containerPort": 8080
    }
  ],
  "deploymentConfiguration": {
    "deploymentCircuitBreaker": {
      "enable": true,
      "rollback": true
    },
    "maximumPercent": 200,
    "minimumHealthyPercent": 100
  },
  "healthCheckGracePeriodSeconds": 60,
  "enableECSManagedTags": true,
  "propagateTags": "SERVICE",
  "tags": [
    {
      "key": "Environment",
      "value": "production"
    },
    {
      "key": "Service",
      "value": "backend"
    }
  ]
}
```

---

## Auto Scaling Configuration

### Target Tracking Scaling Policy

```json
{
  "ServiceNamespace": "ecs",
  "ResourceId": "service/production-cluster/backend",
  "ScalableDimension": "ecs:service:DesiredCount",
  "PolicyName": "backend-cpu-scaling",
  "PolicyType": "TargetTrackingScaling",
  "TargetTrackingScalingPolicyConfiguration": {
    "TargetValue": 70.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
    },
    "ScaleOutCooldown": 60,
    "ScaleInCooldown": 300
  }
}
```

### Register Scalable Target

```bash
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --resource-id service/production-cluster/backend \
  --scalable-dimension ecs:service:DesiredCount \
  --min-capacity 3 \
  --max-capacity 10
```

---

## IAM Roles

### ECS Task Execution Role

`iam/ecs-task-execution-role.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "*"
    }
  ]
}
```

### ECS Task Role (Application Permissions)

`iam/backend-task-role.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::my-app-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "sqs:SendMessage",
        "sqs:ReceiveMessage",
        "sqs:DeleteMessage"
      ],
      "Resource": "arn:aws:sqs:<REGION>:<ACCOUNT_ID>:my-queue"
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:<REGION>:<ACCOUNT_ID>:table/MyTable"
    }
  ]
}
```

---

## Deployment Commands

### Register Task Definition

```bash
aws ecs register-task-definition \
  --cli-input-json file://ecs/backend-task-definition.json
```

### Create or Update Service

```bash
# Create service
aws ecs create-service \
  --cli-input-json file://ecs/backend-service.json

# Update service (force new deployment)
aws ecs update-service \
  --cluster production-cluster \
  --service backend \
  --force-new-deployment

# Update service with new task definition
aws ecs update-service \
  --cluster production-cluster \
  --service backend \
  --task-definition backend:5
```

### Deploy with Blue-Green Deployment

```bash
# Using CodeDeploy for blue-green deployment
aws deploy create-deployment \
  --application-name backend-app \
  --deployment-group-name production \
  --revision revisionType=AppSpecContent,appSpecContent={content='
{
  "version": 0.0,
  "Resources": [
    {
      "TargetService": {
        "Type": "AWS::ECS::Service",
        "Properties": {
          "TaskDefinition": "arn:aws:ecs:<REGION>:<ACCOUNT_ID>:task-definition/backend:5",
          "LoadBalancerInfo": {
            "ContainerName": "backend",
            "ContainerPort": 8080
          }
        }
      }
    }
  ]
}
'}
```

---

## Monitoring & Logging

### CloudWatch Log Group

```bash
aws logs create-log-group --log-group-name /ecs/backend
aws logs put-retention-policy \
  --log-group-name /ecs/backend \
  --retention-in-days 30
```

### CloudWatch Alarms

```bash
# High CPU alarm
aws cloudwatch put-metric-alarm \
  --alarm-name backend-high-cpu \
  --alarm-description "Backend CPU above 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/ECS \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=ServiceName,Value=backend Name=ClusterName,Value=production-cluster \
  --alarm-actions arn:aws:sns:<REGION>:<ACCOUNT_ID>:alerts

# High memory alarm
aws cloudwatch put-metric-alarm \
  --alarm-name backend-high-memory \
  --alarm-description "Backend Memory above 85%" \
  --metric-name MemoryUtilization \
  --namespace AWS/ECS \
  --statistic Average \
  --period 300 \
  --threshold 85 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=ServiceName,Value=backend Name=ClusterName,Value=production-cluster \
  --alarm-actions arn:aws:sns:<REGION>:<ACCOUNT_ID>:alerts
```

---

## Troubleshooting Commands

### Check Service Status

```bash
# Describe service
aws ecs describe-services \
  --cluster production-cluster \
  --services backend

# List tasks
aws ecs list-tasks \
  --cluster production-cluster \
  --service-name backend

# Describe task
aws ecs describe-tasks \
  --cluster production-cluster \
  --tasks <task-id>
```

### View Logs

```bash
# Tail logs
aws logs tail /ecs/backend --follow

# Get logs for specific time range
aws logs filter-log-events \
  --log-group-name /ecs/backend \
  --start-time $(date -u -d '1 hour ago' +%s)000 \
  --end-time $(date -u +%s)000 \
  --filter-pattern "ERROR"
```

### Check Target Health

```bash
aws elbv2 describe-target-health \
  --target-group-arn <target-group-arn>
```

---

## Best Practices

### ✅ DO

1. **Use Fargate** - Serverless containers, no EC2 management
2. **Use Secrets Manager** - Secure secret storage
3. **Enable Circuit Breaker** - Auto-rollback on failures
4. **Use Target Tracking** - Auto-scaling based on metrics
5. **Configure Health Checks** - In container and ALB
6. **Use Task Roles** - Least privilege IAM permissions
7. **Enable Container Insights** - Enhanced monitoring
8. **Use Private Subnets** - Security best practice
9. **CloudWatch Logs** - Centralized logging
10. **Blue-Green Deployment** - Zero-downtime updates

### ❌ DON'T

1. **Don't use EC2 launch type** - Unless you need host access
2. **Don't hardcode secrets** - Use Secrets Manager
3. **Don't skip health checks** - Poor availability
4. **Don't use public subnets** - Security risk
5. **Don't assign public IPs** - Use NAT Gateway
6. **Don't skip log retention** - Set retention policy
7. **Don't use root user** - Non-root in Dockerfile
8. **Don't skip alarms** - No visibility into issues

### Cost Optimization

1. Use Fargate Spot for dev/test environments
2. Right-size CPU and memory allocations
3. Use Savings Plans for predictable workloads
4. Enable auto-scaling to match demand
5. Set log retention policies (7-30 days)
6. Use S3 for long-term log storage
7. Clean up unused task definitions
