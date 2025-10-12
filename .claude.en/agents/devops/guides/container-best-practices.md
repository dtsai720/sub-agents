# Container Best Practices Guide

## Security Best Practices

1. ✅ **Run as non-root user**
2. ✅ **Use minimal base images** (alpine, distroless, scratch)
3. ✅ **Scan for vulnerabilities** (Trivy, Snyk)
4. ✅ **Don't hardcode secrets**
5. ✅ **Use multi-stage builds**
6. ✅ **Keep images small**
7. ✅ **Update base images regularly**
8. ✅ **Use specific version tags**

## Performance Optimization

1. Layer caching - Order layers by change frequency
2. .dockerignore - Exclude unnecessary files
3. Multi-stage builds - Separate build and runtime
4. Minimize layers - Combine RUN commands
5. Use appropriate base image size

## Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD ["/app/healthcheck"]
```

## Resource Management

- Set CPU and memory limits
- Use health checks
- Implement graceful shutdown
- Handle signals properly (SIGTERM)
