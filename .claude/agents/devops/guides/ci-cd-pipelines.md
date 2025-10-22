# CI/CD Pipeline Complete Guide

本文件提供 GitHub Actions 與 GitLab CI 的完整 CI/CD Pipeline 配置。

---

## GitHub Actions - Complete Pipeline

`.github/workflows/backend-deploy.yml`:

```yaml
name: Backend CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: backend

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.21'
      - run: go test -v -race -coverprofile=coverage.out ./...

  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          format: 'sarif'
          output: 'trivy-results.sarif'

  build-and-push:
    needs: [test, security-scan]
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      - uses: aws-actions/amazon-ecr-login@v2
      - run: |
          docker build -t ${{ env.ECR_REPOSITORY }}:${{ github.sha }} .
          docker push ${{ env.ECR_REPOSITORY }}:${{ github.sha }}

  deploy-production:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
    steps:
      - run: |
          aws ecs update-service --cluster production \
            --service backend --force-new-deployment
```

---

## GitLab CI - Complete Pipeline

`.gitlab-ci.yml`:

```yaml
stages:
  - test
  - build
  - deploy

test:
  stage: test
  image: golang:1.21
  script:
    - go test -v ./...

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t $ECR_REPOSITORY:$CI_COMMIT_SHA .
    - docker push $ECR_REPOSITORY:$CI_COMMIT_SHA

deploy:production:
  stage: deploy
  script:
    - aws ecs update-service --cluster production --service backend --force-new-deployment
  only:
    - main
  when: manual
```

---

## Best Practices

1. ✅ Test before build
2. ✅ Security scan in pipeline
3. ✅ Manual approval for production
4. ✅ Automatic rollback on failure
5. ✅ Notify team on deployment
