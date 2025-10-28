---
name: Deploying Applications
description: Deployment and DevOps best practices covering zero-downtime deployment, Cloudflare Tunnel configuration, WebSocket support, cloud logging, monitoring, and database migration strategies. Use when deploying to production, designing CI/CD pipelines, configuring infrastructure, setting up logging and observability, or when user mentions deployment, zero-downtime, Cloudflare Tunnel, WebSocket, ActionCable, Google Cloud Logging, monitoring, alerting, log retention, database migration, Docker, Kubernetes, rolling update, blue-green, or canary.
---

# Deployment Practices

## Overview

Comprehensive deployment and operations best practices guide, focusing on zero-downtime deployment, safe database migrations, and modern infrastructure configuration. These guidelines are for Ruby on Rails applications, but many principles apply to other web applications.

## Zero-Downtime Deployment

Deployment strategies for seamless updates. See [Zero-Downtime Deployment Guide](references/zero-downtime-deployment.md) for:

### Core Principles

Core principles and challenges of zero-downtime deployment.

- **Backward Compatibility**
  - New versions must coexist peacefully with old version data
  - Database schema changes need phased rollout
  - API response formats must remain compatible

- **Expand-Contract Pattern**
  - **Phase 1 (Expand)**: Add new fields/features, new and old coexist
  - **Phase 2 (Migrate)**: Switch to new features, keep old features
  - **Phase 3 (Contract)**: Remove old features

- **Key Challenge**
  - New and old versions will run simultaneously
  - Database schema must support both versions
  - APIs need versioning or compatibility design

### Database Migration Strategies

Safe database migration strategies.

- **Safe Operations**
  - Add nullable column
  - Add index (using `algorithm: :concurrently`)
  - Add table

- **Dangerous Operations and Solutions**
  - Remove column → Use Expand-Contract 3-phase
  - Rename column → Add column → migrate data → remove old column
  - Change column type → Add column → migrate data → remove old column
  - Add NOT NULL column → nullable first → backfill data → add constraint
  - Add foreign key → Use `validate: false` → manual validation

- **strong_migrations Gem**
  - Auto-detect dangerous operations
  - Provide safe alternatives
  - Enforce best practices

### Deployment Strategies

Comparison of different deployment strategies.

- **Rolling Deployment**
  - Gradually replace old containers
  - Kubernetes default strategy
  - Suitable for most applications

- **Blue-Green Deployment**
  - Maintain two complete environments
  - Instant traffic switch
  - Easy rollback, but requires 2x resources

- **Canary Deployment**
  - New version gradually receives traffic (5% → 25% → 50% → 100%)
  - Reduces risk
  - Requires Istio or similar tools

### Health Checks

Health checks to ensure application readiness.

- **Rails 8 Built-in `/up` Endpoint**
  - Checks if application started successfully
  - Doesn't check external dependencies
  - Suitable for basic liveness probe

- **Custom Health Check**
  - Check database, Redis, and other dependencies
  - Used for readiness probe
  - Can detect migration status

- **Liveness vs Readiness**
  - Liveness: "Is application alive?" → Failure = restart
  - Readiness: "Is application ready for traffic?" → Failure = remove traffic

### Graceful Shutdown

Gracefully stop services to avoid interrupting requests.

- **Puma Graceful Shutdown**
  - Stop accepting new requests
  - Wait for existing requests to complete (worker_timeout)
  - Force shutdown after timeout

- **Docker Configuration**
  - `stop_signal: SIGTERM`
  - `stop_grace_period: 60s`
  - PreStop hook (Kubernetes)

- **Background Job Shutdown**
  - Sidekiq automatically waits for running jobs
  - Set appropriate timeout

## Cloudflare Tunnel

Using Cloudflare Tunnel for secure remote access. See [Cloudflare Tunnel Guide](references/cloudflare-tunnel.md) for:

### Architecture

Cloudflare Tunnel architecture and advantages.

- **No Open Ports Needed**
  - All connections are outbound (cloudflared → Cloudflare)
  - No firewall rule configuration needed
  - Automatic SSL/TLS

- **DDoS Protection**
  - Cloudflare network layer protection
  - Automatic malicious traffic blocking
  - Global CDN acceleration

- **WebSocket Support**
  - Supports ActionCable
  - Auto-reconnect mechanism
  - 100-second timeout (Free plan)

### WebSocket Configuration

Configuration optimized for ActionCable.

- **Critical Settings**
  ```yaml
  # cloudflared-config.yaml
  ingress:
    - hostname: api.yourdomain.com
      path: /cable
      service: http://web:3000
      originRequest:
        http2Origin: false        # Note: Required! WebSocket needs HTTP/1.1
        keepAliveTimeout: 90s     # Long connection timeout
  ```

- **Why `http2Origin: false` is Necessary**
  - WebSocket requires HTTP/1.1 upgrade
  - HTTP/2 doesn't support protocol upgrade
  - Missing this setting causes WebSocket connection failures

### Setup Steps

- **Create Tunnel**
  - Cloudflare Zero Trust Dashboard
  - Download credentials JSON
  - Record Tunnel ID

- **Configure DNS**
  - Add CNAME record
  - Point to `<tunnel-id>.cfargotunnel.com`
  - Enable Proxy (orange cloud)

- **Docker Compose Integration**
  - cloudflared service
  - Secrets management
  - Health check

### Limitations and Solutions

- **WebSocket Timeout**
  - Free plan: 100 seconds
  - Solution: ActionCable auto-reconnect
  - Upgrade plan for longer timeout

- **Concurrency Limit**
  - Single instance: ~1,000 concurrent connections
  - Solution: Multiple cloudflared replicas

## Logging and Observability

Comprehensive logging strategy for production applications. See [Cloud Logging Guide](references/cloud-logging.md) for:

### Google Cloud Logging Integration

Full-featured logging service for GKE, Cloud Run, and Compute Engine deployments.

- **Integration Options**
  - google-cloud-logging gem for Rails (recommended)
  - Fluentd/Fluent Bit sidecar (polyglot environments)
  - Native GKE integration (zero configuration)

- **Rails Configuration**
  - Structured logging with Lograge
  - JSON output for automatic parsing
  - Workload Identity authentication
  - Custom field mapping

### Log Retention Strategy

Tiered storage for cost-effective log retention.

- **Hot Data (0-7 days)**: Cloud Logging (real-time queries)
- **Warm Data (7-30 days)**: Log Buckets (30-day retention)
- **Cool Data (30-90 days)**: BigQuery (analytics)
- **Cold Data (>90 days)**: Cloud Storage Nearline
- **Archive Data (>365 days)**: Cloud Storage Archive

### Cost Optimization

Strategies to control logging costs.

- **Exclusion Filters**
  - Health checks: `/up`, `/health`, `/readiness`
  - Static assets: `/assets/*`, `*.css`, `*.js`
  - Debug logs in production

- **Sampling**
  - 90% sampling for successful requests (200-299 status)
  - 50% sampling for high-volume access logs

- **Payload Optimization**
  - Remove sensitive data (passwords, tokens)
  - Exclude unnecessary parameters
  - Limit request/response body logging

### Monitoring & Alerting

Log-based metrics and alert policies.

- **Key Metrics**
  - Error rate (target: < 0.1%)
  - Slow requests (>1s)
  - Database connection errors
  - User events for analytics

- **Alert Policies**
  - High error rate (>10/min for 5 min)
  - No logs received (10 min window)
  - Slow requests (avg >2s for 10 min)

- **Dashboards**
  - Error rate trends
  - Request duration P50/P95/P99
  - Log ingestion volume
  - Top errors table

## Deployment Checklist

Complete checklist before deployment.

### Pre-Deployment
- [ ] Migration backward compatible
  - [ ] No remove_column
  - [ ] No rename_column
  - [ ] New columns are nullable
  - [ ] Index uses `algorithm: :concurrently`
- [ ] Code backward compatible
  - [ ] API response format unchanged or versioned
  - [ ] Enum value order unchanged
  - [ ] Background job compatible with old schema
- [ ] Health check configured
- [ ] Graceful shutdown configured
- [ ] Rollback plan ready

### During Deployment
- [ ] Monitor error rate (target: < 0.1%)
- [ ] Monitor response time (P95, P99)
- [ ] Check database connection count
- [ ] Observe new/old version traffic distribution
- [ ] Review Rails logs

### Post-Deployment
- [ ] Smoke test main features
- [ ] Performance metrics normal
- [ ] Old containers fully stopped
- [ ] Monitor 24 hours without anomalies
- [ ] Can start next phase (Contract phase)

## Rollback Strategies

Safe rollback strategies.

### Application Rollback

- **Kubernetes**
  ```bash
  kubectl rollout undo deployment/rails-app
  kubectl rollout status deployment/rails-app
  ```

- **Docker Compose**
  - Switch to old version image
  - `docker compose up -d`

### Database Rollback

- **Important Principle**: Database migrations should usually **NOT be rolled back**
  - Rollback causes data loss
  - Should only rollback application, keep database schema

- **Exceptions**
  - Adding nullable column can be safely rolled back
  - Tables with no data written can be dropped

### Feature Flags

Quickly disable problematic features without redeployment.

- **Flipper Gem**
  - Dynamic feature toggles
  - Supports percentage rollout
  - Takes effect immediately

## Common Pitfalls

Common deployment pitfalls and solutions.

### NOT NULL Constraint
- **Problem**: Directly add NOT NULL column
- **Consequence**: Old version INSERT fails
- **Solution**: Four phases (nullable → validate → backfill → NOT NULL)

### Enum Value Changes
- **Problem**: Change enum order
- **Consequence**: Database values map incorrectly
- **Solution**: Explicitly specify enum values, only add never modify

### API Format Changes
- **Problem**: Directly change JSON structure
- **Consequence**: Old version frontend crashes
- **Solution**: API versioning

### Removing Index
- **Problem**: Directly remove index
- **Consequence**: Query performance plummets
- **Solution**: First deploy code not using index, monitor, then remove

## Monitoring

Key monitoring metrics during deployment.

### Key Metrics
- **Error Rate**: < 0.1%
- **Response Time**: P95, P99
- **Database Connection Pool**: Usage rate
- **Memory Usage**: RSS, heap size
- **Request Rate**: QPS

### Alerting
- Error rate exceeds threshold → Immediate alert
- Response time increase > 20% → Warning
- Memory usage > 80% → Notification

## Resources

- [Zero-Downtime Deployment Guide](references/zero-downtime-deployment.md) - Complete guide to zero-downtime deployments
- [Cloudflare Tunnel Guide](references/cloudflare-tunnel.md) - Cloudflare Tunnel setup with WebSocket support
- [Cloud Logging Guide](references/cloud-logging.md) - Google Cloud Logging integration and best practices

## Related Skills

- **ruby-development**: Database migration best practices
- **backend-architecture**: Redis and database configuration
- **performance-optimization**: Deployment performance considerations
