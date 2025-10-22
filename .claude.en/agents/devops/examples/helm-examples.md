# Helm Chart Examples

Complete Helm Chart examples for deploying applications to Kubernetes (EKS/AKS/GKE).

---

## Table of Contents

1. [Backend Service Helm Chart](#backend-service-helm-chart)
2. [Frontend Service Helm Chart](#frontend-service-helm-chart)
3. [Database Helm Chart](#database-helm-chart)
4. [Complete Multi-Environment Setup](#complete-multi-environment-setup)
5. [Helm Best Practices](#helm-best-practices)

---

## Backend Service Helm Chart

### Directory Structure

```
helm/backend/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-staging.yaml
├── values-production.yaml
└── templates/
    ├── _helpers.tpl
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── hpa.yaml
    ├── pdb.yaml
    ├── configmap.yaml
    └── secret.yaml
```

### Chart.yaml

```yaml
apiVersion: v2
name: backend
description: Backend API Service Helm Chart
type: application
version: 1.0.0
appVersion: "1.0.0"
keywords:
  - backend
  - api
  - microservice
maintainers:
  - name: DevOps Team
    email: devops@example.com
dependencies: []
```

### values.yaml (Default Values)

```yaml
# Replica configuration
replicaCount: 3

# Image configuration
image:
  repository: 123456789012.dkr.ecr.us-west-2.amazonaws.com/backend
  pullPolicy: IfNotPresent
  tag: "" # Overridden by CI/CD (e.g., git SHA)

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

# Service Account
serviceAccount:
  create: true
  annotations: {}
  name: ""

# Pod annotations
podAnnotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "8080"
  prometheus.io/path: "/metrics"

# Pod security context
podSecurityContext:
  runAsNonRoot: true
  runAsUser: 65534
  fsGroup: 65534

# Container security context
securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL

# Service configuration
service:
  type: ClusterIP
  port: 80
  targetPort: 8080
  annotations: {}

# Ingress configuration
ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
  hosts:
    - host: api.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: backend-tls
      hosts:
        - api.example.com

# Resource limits and requests
resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi

# Horizontal Pod Autoscaler
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

# Pod Disruption Budget
podDisruptionBudget:
  enabled: true
  minAvailable: 2

# Health checks
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3

# Environment variables
env:
  - name: PORT
    value: "8080"
  - name: LOG_LEVEL
    value: "info"
  - name: ENVIRONMENT
    value: "production"

# Environment variables from ConfigMap
envFrom:
  - configMapRef:
      name: backend-config

# Secrets (use External Secrets Operator in production)
secrets:
  enabled: true
  data:
    # These should be managed by External Secrets Operator
    DATABASE_URL: ""
    REDIS_URL: ""
    JWT_SECRET: ""

# Node selector
nodeSelector: {}

# Tolerations
tolerations: []

# Affinity rules
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app.kubernetes.io/name
                operator: In
                values:
                  - backend
          topologyKey: kubernetes.io/hostname
```

### values-dev.yaml

```yaml
replicaCount: 1

image:
  tag: "dev-latest"

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 200m
    memory: 256Mi

autoscaling:
  enabled: false

podDisruptionBudget:
  enabled: false

ingress:
  hosts:
    - host: api-dev.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: backend-dev-tls
      hosts:
        - api-dev.example.com

env:
  - name: PORT
    value: "8080"
  - name: LOG_LEVEL
    value: "debug"
  - name: ENVIRONMENT
    value: "development"
```

### values-staging.yaml

```yaml
replicaCount: 2

image:
  tag: "staging-latest"

resources:
  requests:
    cpu: 150m
    memory: 192Mi
  limits:
    cpu: 300m
    memory: 384Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 5

podDisruptionBudget:
  enabled: true
  minAvailable: 1

ingress:
  hosts:
    - host: api-staging.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: backend-staging-tls
      hosts:
        - api-staging.example.com

env:
  - name: PORT
    value: "8080"
  - name: LOG_LEVEL
    value: "info"
  - name: ENVIRONMENT
    value: "staging"
```

### values-production.yaml

```yaml
replicaCount: 5

image:
  tag: "" # Must be set by CI/CD (git SHA)

resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 5
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

podDisruptionBudget:
  enabled: true
  minAvailable: 3

ingress:
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "1000"
    nginx.ingress.kubernetes.io/cors-allow-origin: "https://example.com"
  hosts:
    - host: api.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: backend-prod-tls
      hosts:
        - api.example.com

env:
  - name: PORT
    value: "8080"
  - name: LOG_LEVEL
    value: "warn"
  - name: ENVIRONMENT
    value: "production"

# Production node selector (e.g., dedicated node pool)
nodeSelector:
  workload: production
```

### templates/_helpers.tpl

```yaml
{{/*
Expand the name of the chart.
*/}}
{{- define "backend.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "backend.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Create chart name and version as used by the chart label.
*/}}
{{- define "backend.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "backend.labels" -}}
helm.sh/chart: {{ include "backend.chart" . }}
{{ include "backend.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "backend.selectorLabels" -}}
app.kubernetes.io/name: {{ include "backend.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
Create the name of the service account to use
*/}}
{{- define "backend.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "backend.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}
```

### templates/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "backend.fullname" . }}
  labels:
    {{- include "backend.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      {{- include "backend.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      annotations:
        {{- with .Values.podAnnotations }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
      labels:
        {{- include "backend.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "backend.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
      - name: {{ .Chart.Name }}
        securityContext:
          {{- toYaml .Values.securityContext | nindent 12 }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.targetPort }}
          protocol: TCP
        livenessProbe:
          {{- toYaml .Values.livenessProbe | nindent 12 }}
        readinessProbe:
          {{- toYaml .Values.readinessProbe | nindent 12 }}
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
        env:
        {{- toYaml .Values.env | nindent 10 }}
        {{- if .Values.secrets.enabled }}
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: {{ include "backend.fullname" . }}-secret
              key: DATABASE_URL
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: {{ include "backend.fullname" . }}-secret
              key: REDIS_URL
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: {{ include "backend.fullname" . }}-secret
              key: JWT_SECRET
        {{- end }}
        {{- with .Values.envFrom }}
        envFrom:
        {{- toYaml . | nindent 10 }}
        {{- end }}
        volumeMounts:
        - name: tmp
          mountPath: /tmp
      volumes:
      - name: tmp
        emptyDir: {}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

### templates/service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "backend.fullname" . }}
  labels:
    {{- include "backend.labels" . | nindent 4 }}
  {{- with .Values.service.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "backend.selectorLabels" . | nindent 4 }}
```

### templates/ingress.yaml

```yaml
{{- if .Values.ingress.enabled -}}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "backend.fullname" . }}
  labels:
    {{- include "backend.labels" . | nindent 4 }}
  {{- with .Values.ingress.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
  {{- if .Values.ingress.className }}
  ingressClassName: {{ .Values.ingress.className }}
  {{- end }}
  {{- if .Values.ingress.tls }}
  tls:
    {{- range .Values.ingress.tls }}
    - hosts:
        {{- range .hosts }}
        - {{ . | quote }}
        {{- end }}
      secretName: {{ .secretName }}
    {{- end }}
  {{- end }}
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ .path }}
            pathType: {{ .pathType }}
            backend:
              service:
                name: {{ include "backend.fullname" $ }}
                port:
                  number: {{ $.Values.service.port }}
          {{- end }}
    {{- end }}
{{- end }}
```

### templates/hpa.yaml

```yaml
{{- if .Values.autoscaling.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "backend.fullname" . }}
  labels:
    {{- include "backend.labels" . | nindent 4 }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "backend.fullname" . }}
  minReplicas: {{ .Values.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.autoscaling.maxReplicas }}
  metrics:
    {{- if .Values.autoscaling.targetCPUUtilizationPercentage }}
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: {{ .Values.autoscaling.targetCPUUtilizationPercentage }}
    {{- end }}
    {{- if .Values.autoscaling.targetMemoryUtilizationPercentage }}
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: {{ .Values.autoscaling.targetMemoryUtilizationPercentage }}
    {{- end }}
{{- end }}
```

### templates/pdb.yaml

```yaml
{{- if .Values.podDisruptionBudget.enabled }}
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: {{ include "backend.fullname" . }}
  labels:
    {{- include "backend.labels" . | nindent 4 }}
spec:
  {{- if .Values.podDisruptionBudget.minAvailable }}
  minAvailable: {{ .Values.podDisruptionBudget.minAvailable }}
  {{- end }}
  {{- if .Values.podDisruptionBudget.maxUnavailable }}
  maxUnavailable: {{ .Values.podDisruptionBudget.maxUnavailable }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "backend.selectorLabels" . | nindent 6 }}
{{- end }}
```

### templates/configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: backend-config
  labels:
    {{- include "backend.labels" . | nindent 4 }}
data:
  APP_NAME: "Backend API"
  MAX_CONNECTIONS: "100"
  TIMEOUT: "30s"
```

### templates/secret.yaml

```yaml
{{- if .Values.secrets.enabled }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "backend.fullname" . }}-secret
  labels:
    {{- include "backend.labels" . | nindent 4 }}
type: Opaque
data:
  {{- if .Values.secrets.data.DATABASE_URL }}
  DATABASE_URL: {{ .Values.secrets.data.DATABASE_URL | b64enc }}
  {{- end }}
  {{- if .Values.secrets.data.REDIS_URL }}
  REDIS_URL: {{ .Values.secrets.data.REDIS_URL | b64enc }}
  {{- end }}
  {{- if .Values.secrets.data.JWT_SECRET }}
  JWT_SECRET: {{ .Values.secrets.data.JWT_SECRET | b64enc }}
  {{- end }}
{{- end }}
```

---

## Frontend Service Helm Chart

Similar structure to backend, with frontend-specific configurations.

### values.yaml (Frontend)

```yaml
replicaCount: 3

image:
  repository: 123456789012.dkr.ecr.us-west-2.amazonaws.com/frontend
  pullPolicy: IfNotPresent
  tag: ""

service:
  type: ClusterIP
  port: 80
  targetPort: 80

ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
  hosts:
    - host: app.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: frontend-tls
      hosts:
        - app.example.com

resources:
  requests:
    cpu: 50m
    memory: 64Mi
  limits:
    cpu: 100m
    memory: 128Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5

# Frontend environment variables (injected at build time)
env:
  - name: REACT_APP_API_URL
    value: "https://api.example.com"
  - name: REACT_APP_ENV
    value: "production"
```

---

## Complete Multi-Environment Setup

### Deployment Commands

```bash
# Development Environment
helm upgrade --install backend ./helm/backend \
  -f ./helm/backend/values-dev.yaml \
  --set image.tag=dev-$(git rev-parse --short HEAD) \
  --namespace dev \
  --create-namespace

# Staging Environment
helm upgrade --install backend ./helm/backend \
  -f ./helm/backend/values-staging.yaml \
  --set image.tag=staging-$(git rev-parse --short HEAD) \
  --namespace staging \
  --create-namespace

# Production Environment (Manual Approval)
helm upgrade --install backend ./helm/backend \
  -f ./helm/backend/values-production.yaml \
  --set image.tag=$(git rev-parse --short HEAD) \
  --namespace production \
  --create-namespace \
  --wait \
  --timeout 10m
```

### Rollback Commands

```bash
# Check release history
helm history backend -n production

# Rollback to previous version
helm rollback backend -n production

# Rollback to specific revision
helm rollback backend 5 -n production
```

### Debugging Commands

```bash
# Dry-run to see generated YAML
helm install backend ./helm/backend \
  -f ./helm/backend/values-production.yaml \
  --dry-run --debug

# Lint chart
helm lint ./helm/backend

# Validate templates
helm template backend ./helm/backend \
  -f ./helm/backend/values-production.yaml

# Get values of deployed release
helm get values backend -n production

# Get manifest of deployed release
helm get manifest backend -n production
```

---

## Helm Best Practices

### 1. Values File Organization

```yaml
# ✅ Good: Organized by component
image:
  repository: myapp
  tag: v1.0.0
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

# ❌ Bad: Flat structure
imageRepository: myapp
imageTag: v1.0.0
imagePullPolicy: IfNotPresent
serviceType: ClusterIP
servicePort: 80
```

### 2. Template Helpers

```yaml
# ✅ Good: Use helpers for repeated logic
{{- define "myapp.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" }}
{{- end }}

metadata:
  name: {{ include "myapp.fullname" . }}

# ❌ Bad: Repeated logic in every template
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
```

### 3. Resource Naming

```yaml
# ✅ Good: Consistent naming with release name
name: {{ include "backend.fullname" . }}-deployment
name: {{ include "backend.fullname" . }}-service
name: {{ include "backend.fullname" . }}-ingress

# ❌ Bad: Hardcoded names
name: backend-deployment
name: backend-service
name: backend-ingress
```

### 4. Default Values

```yaml
# ✅ Good: Sensible defaults with override capability
replicaCount: 3
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 200m
    memory: 256Mi

# ❌ Bad: No defaults or extreme values
replicaCount: 100
resources:
  requests:
    cpu: 8
    memory: 16Gi
```

### 5. Secret Management

```yaml
# ✅ Good: Use External Secrets Operator (Production)
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: backend-secret
spec:
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: backend-secret
  data:
  - secretKey: DATABASE_URL
    remoteRef:
      key: prod/backend/database-url

# ❌ Bad: Hardcoded secrets in values.yaml
secrets:
  DATABASE_URL: "postgresql://user:password@host:5432/db"
```

### 6. Environment-Specific Overrides

```bash
# ✅ Good: Use values-<env>.yaml files
helm upgrade --install backend ./helm/backend -f values-production.yaml

# ❌ Bad: Pass individual --set flags
helm upgrade --install backend ./helm/backend \
  --set replicas=5 \
  --set resources.requests.cpu=200m \
  --set resources.requests.memory=256Mi \
  --set ingress.host=api.example.com
  # ... (100+ flags)
```

### 7. Version Management

```yaml
# Chart.yaml
# ✅ Good: Semantic versioning
version: 1.2.3  # Chart version
appVersion: "2.1.0"  # Application version

# ❌ Bad: No versioning or inconsistent
version: latest
appVersion: "prod"
```

### 8. Documentation

```yaml
# values.yaml
# ✅ Good: Document each value
## Number of replicas for the deployment
## @param replicaCount - Number of pod replicas
replicaCount: 3

## Docker image configuration
## @param image.repository - Container image repository
## @param image.tag - Container image tag
image:
  repository: myapp
  tag: ""

# ❌ Bad: No documentation
replicaCount: 3
image:
  repository: myapp
  tag: ""
```

---

## CI/CD Integration with Helm

### GitHub Actions Example

```yaml
name: Deploy to Kubernetes

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-west-2

      - name: Login to Amazon ECR
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push Docker image
        run: |
          docker build -t $ECR_REGISTRY/backend:${{ github.sha }} .
          docker push $ECR_REGISTRY/backend:${{ github.sha }}

      - name: Configure kubectl
        run: |
          aws eks update-kubeconfig --name production-cluster --region us-west-2

      - name: Deploy with Helm
        run: |
          helm upgrade --install backend ./helm/backend \
            -f ./helm/backend/values-production.yaml \
            --set image.tag=${{ github.sha }} \
            --namespace production \
            --wait \
            --timeout 10m
```

---

## Summary

- **Helm Charts** provide templated, reusable Kubernetes manifests
- **Values files** enable environment-specific configurations (dev/staging/prod)
- **Template helpers** reduce duplication and improve maintainability
- **Semantic versioning** tracks chart and application versions independently
- **External Secrets Operator** manages sensitive data securely
- **CI/CD integration** automates build, push, and deployment processes
- **Helm rollback** enables quick recovery from failed deployments

This Helm setup provides a production-ready, scalable, and maintainable way to deploy applications to Kubernetes clusters (EKS/AKS/GKE).
