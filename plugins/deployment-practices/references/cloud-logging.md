# Google Cloud Logging Integration Guide

## Table of Contents

- [Overview](#overview)
- [Integration Options](#integration-options)
- [Rails Configuration](#rails-configuration)
- [Log Retention Strategy](#log-retention-strategy)
- [Cost Optimization](#cost-optimization)
- [Monitoring & Alerting](#monitoring--alerting)
- [Deployment Examples](#deployment-examples)
- [Implementation Checklist](#implementation-checklist)

---

## Overview

Google Cloud Logging (formerly Stackdriver Logging) is a fully managed logging service that provides:

- **Automatic Collection**: Native integration with GKE, Cloud Run, App Engine, and Compute Engine
- **Automatic Rotation**: No manual logrotate configuration needed
- **Retention**: Default 30 days, extendable up to 10 years
- **Real-time Querying**: Sub-second latency with advanced filtering
- **Structured Logging**: First-class JSON support with automatic parsing

### Pricing (2025)

```
First 50 GB/month:     Free
Additional ingestion:  $0.50/GB
Long-term storage:     $0.01/GB/month
BigQuery export:       Standard BigQuery pricing
Cloud Storage export:  Standard GCS pricing
```

### When to Use Google Cloud Logging

**Good fit:**
- Running on Google Cloud Platform (GKE, Cloud Run, GCE)
- Need compliance with long retention requirements
- Want automatic log collection without configuration
- Need integration with Google Cloud's observability stack

**Consider alternatives:**
- Multi-cloud deployment (use Datadog, Logtail)
- Cost-sensitive with >100GB/day (consider self-hosted ELK)
- Need advanced APM features (use Datadog, New Relic)

---

## Integration Options

### Option A: Google Cloud Logging Gem (Recommended for Rails)

**Pros:**
- Native Rails integration
- Automatic request/response logging
- Custom field support
- Local development fallback

**Cons:**
- Requires gem dependency
- Slight performance overhead
- Need to manage credentials

**Use case:** Full-featured Rails applications with custom logging needs

### Option B: Fluentd/Fluent Bit (Container-based)

**Pros:**
- No application code changes
- Works with any language/framework
- Highly configurable filtering
- Community plugins available

**Cons:**
- Additional container overhead
- More complex configuration
- Requires sidecar or daemonset

**Use case:** Microservices architecture, polyglot environments

### Option C: Native GKE Integration (Simplest)

**Pros:**
- Zero configuration required
- Automatic metadata injection
- Lowest overhead
- Automatic updates

**Cons:**
- Limited customization
- GKE-only
- No structured logging unless app outputs JSON

**Use case:** Simple applications, standard GKE deployments

---

## Rails Configuration

### Setup 1: Using google-cloud-logging Gem

#### Installation

```ruby
# Gemfile
gem 'google-cloud-logging'
gem 'google-cloud-logging-rails'
```

#### Configuration

```ruby
# config/environments/production.rb
Rails.application.configure do
  # Enable Google Cloud Logging
  config.google_cloud.use_logging = true
  config.google_cloud.project_id = ENV['GOOGLE_CLOUD_PROJECT']

  # Authentication (choose one)
  # Option 1: Service account key file
  config.google_cloud.keyfile = ENV['GOOGLE_CLOUD_KEYFILE_JSON']

  # Option 2: Workload Identity (recommended for GKE)
  # config.google_cloud.keyfile = nil  # Auto-detect from metadata server

  # Log level
  config.log_level = :info

  # Resource metadata (for grouping/filtering)
  config.google_cloud.logging.resource_type = 'k8s_container'
  config.google_cloud.logging.labels = {
    environment: Rails.env,
    app: 'myapp',
    version: ENV['APP_VERSION'] || 'unknown'
  }

  # Fallback for local development
  if ENV['GOOGLE_CLOUD_PROJECT'].blank?
    config.logger = ActiveSupport::Logger.new(STDOUT)
  end
end
```

### Setup 2: Hybrid Approach (Local + Cloud)

```ruby
# config/initializers/logging.rb
if Rails.env.production? && ENV['GOOGLE_CLOUD_PROJECT'].present?
  require 'google/cloud/logging'

  logging = Google::Cloud::Logging.new(
    project_id: ENV['GOOGLE_CLOUD_PROJECT'],
    credentials: ENV['GOOGLE_CLOUD_KEYFILE_JSON']
  )

  resource = logging.resource(
    'k8s_container',
    namespace_name: ENV['K8S_NAMESPACE'],
    pod_name: ENV['HOSTNAME'],
    container_name: 'rails-app'
  )

  cloud_logger = logging.logger('rails-app', resource)

  # Broadcast to both local and cloud
  Rails.logger.extend(ActiveSupport::Logger.broadcast(cloud_logger))
end
```

### Setup 3: Structured Logging with Lograge

```ruby
# Gemfile
gem 'lograge'
gem 'google-cloud-logging'

# config/environments/production.rb
Rails.application.configure do
  config.lograge.enabled = true
  config.lograge.formatter = Lograge::Formatters::Json.new

  # Custom fields matching Google Cloud Logging standards
  config.lograge.custom_options = lambda do |event|
    {
      # Google Cloud Logging special fields
      severity: event.payload[:severity] || 'INFO',
      'logging.googleapis.com/trace': event.payload[:trace_id],
      'logging.googleapis.com/spanId': event.payload[:span_id],

      # Application fields
      ddsource: 'rails',
      service: 'myapp',
      request_id: event.payload[:request_id],
      user_id: event.payload[:user_id],
      duration_ms: event.duration,
      params: event.payload[:params].except('controller', 'action', 'password'),

      # Kubernetes metadata (auto-injected by GKE)
      pod_name: ENV['HOSTNAME'],
      namespace: ENV['K8S_NAMESPACE'],

      # Custom business metrics
      endpoint: "#{event.payload[:controller]}##{event.payload[:action]}",
      status: event.payload[:status]
    }
  end

  # Log errors with full stack trace
  config.lograge.custom_payload do |controller|
    {
      host: controller.request.host,
      user_agent: controller.request.user_agent,
      ip: controller.request.remote_ip
    }
  rescue
    {}
  end
end
```

### Authentication Setup

#### Option 1: Service Account Key (Development/Testing)

```bash
# Create service account
gcloud iam service-accounts create rails-app-logger \
  --display-name="Rails App Logger"

# Grant logging write permission
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:rails-app-logger@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/logging.logWriter"

# Create and download key
gcloud iam service-accounts keys create keyfile.json \
  --iam-account=rails-app-logger@PROJECT_ID.iam.gserviceaccount.com

# Set environment variable
export GOOGLE_CLOUD_KEYFILE_JSON=$(cat keyfile.json)
```

#### Option 2: Workload Identity (Production/GKE)

```bash
# Enable Workload Identity on cluster
gcloud container clusters update CLUSTER_NAME \
  --workload-pool=PROJECT_ID.svc.id.goog

# Create Kubernetes service account
kubectl create serviceaccount rails-app-sa

# Bind to Google service account
gcloud iam service-accounts add-iam-policy-binding \
  rails-app-logger@PROJECT_ID.iam.gserviceaccount.com \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:PROJECT_ID.svc.id.goog[default/rails-app-sa]"

# Annotate Kubernetes service account
kubectl annotate serviceaccount rails-app-sa \
  iam.gke.io/gcp-service-account=rails-app-logger@PROJECT_ID.iam.gserviceaccount.com
```

```yaml
# deployment.yaml
spec:
  template:
    spec:
      serviceAccountName: rails-app-sa  # Use Workload Identity
      containers:
      - name: app
        env:
        - name: GOOGLE_CLOUD_PROJECT
          value: "PROJECT_ID"
        # No need for GOOGLE_CLOUD_KEYFILE_JSON
```

---

## Log Retention Strategy

### Tiered Storage Architecture

```
Hot Data (0-7 days)
  ↓ Cloud Logging (real-time queries)
  Cost: $0.50/GB ingestion

Warm Data (7-30 days)
  ↓ Log Buckets (30-day retention)
  Cost: $0.01/GB/month storage

Cool Data (30-90 days)
  ↓ BigQuery (analytics)
  Cost: $0.02/GB/month storage + $6.25/TB query

Cold Data (>90 days)
  ↓ Cloud Storage Nearline
  Cost: $0.01/GB/month

Archive Data (>365 days)
  ↓ Cloud Storage Archive
  Cost: $0.0012/GB/month
```

### Creating Log Buckets

```bash
# Short-term logs (30 days) - within free tier
gcloud logging buckets create short-term \
  --location=asia-east1 \
  --retention-days=30 \
  --description="Application logs with 30-day retention"

# Medium-term logs (90 days) - compliance
gcloud logging buckets create medium-term \
  --location=asia-east1 \
  --retention-days=90 \
  --description="Error and audit logs with 90-day retention"

# Long-term archive (3650 days = 10 years) - regulatory
gcloud logging buckets create long-term \
  --location=asia-east1 \
  --retention-days=3650 \
  --description="Audit logs with 10-year retention for compliance"
```

### Creating Log Sinks (Routing Rules)

```bash
# Route INFO/DEBUG logs to short-term bucket
gcloud logging sinks create app-logs-30d \
  logging.googleapis.com/projects/PROJECT_ID/locations/asia-east1/buckets/short-term \
  --log-filter='
    resource.type="k8s_container" AND
    labels.app="myapp" AND
    severity<"ERROR"
  '

# Route ERROR logs to medium-term bucket
gcloud logging sinks create error-logs-90d \
  logging.googleapis.com/projects/PROJECT_ID/locations/asia-east1/buckets/medium-term \
  --log-filter='
    severity>="ERROR"
  '

# Route audit logs to long-term bucket
gcloud logging sinks create audit-logs-10y \
  logging.googleapis.com/projects/PROJECT_ID/locations/asia-east1/buckets/long-term \
  --log-filter='
    protoPayload.methodName=~".*audit.*" OR
    jsonPayload.audit=true
  '

# Export to BigQuery for analytics
gcloud logging sinks create logs-to-bigquery \
  bigquery.googleapis.com/projects/PROJECT_ID/datasets/logs \
  --log-filter='resource.type="k8s_container"'

# Export old logs to Cloud Storage (cost-effective archive)
gcloud logging sinks create logs-to-gcs \
  storage.googleapis.com/PROJECT_ID-logs-archive \
  --log-filter='timestamp<"2025-01-01T00:00:00Z"'
```

### BigQuery Dataset Setup

```bash
# Create dataset for log exports
bq mk --dataset \
  --location=asia-east1 \
  --description="Application logs for analytics" \
  PROJECT_ID:logs

# Create partitioned table (recommended)
bq mk --table \
  --time_partitioning_field=timestamp \
  --time_partitioning_expiration=7776000 \
  PROJECT_ID:logs.application_logs \
  schema.json

# Example query: Daily error rate
bq query --use_legacy_sql=false '
SELECT
  DATE(timestamp) as date,
  COUNT(*) as error_count,
  APPROX_TOP_COUNT(jsonPayload.error_message, 10) as top_errors
FROM `PROJECT_ID.logs.application_logs`
WHERE severity = "ERROR"
GROUP BY date
ORDER BY date DESC
LIMIT 30
'
```

---

## Cost Optimization

### 1. Exclude Unnecessary Logs

```bash
# Exclude health check endpoints
gcloud logging exclusions create exclude-health-checks \
  --log-filter='
    jsonPayload.path="/health" OR
    jsonPayload.path="/up" OR
    jsonPayload.path="/readiness"
  ' \
  --description="Exclude health check requests"

# Exclude static assets
gcloud logging exclusions create exclude-static-assets \
  --log-filter='
    jsonPayload.path=~"/assets/.*" OR
    jsonPayload.path=~".*\\.(css|js|png|jpg|svg|woff2)$"
  ' \
  --description="Exclude static asset requests"

# Exclude debug logs in production
gcloud logging exclusions create exclude-debug-logs \
  --log-filter='severity="DEBUG"' \
  --description="Exclude debug-level logs"

# Exclude noisy third-party logs
gcloud logging exclusions create exclude-kubernetes-noise \
  --log-filter='
    resource.type="k8s_cluster" AND
    logName=~".*kube-system.*"
  ' \
  --description="Exclude Kubernetes system logs"
```

### 2. Sample High-Volume Logs

```bash
# Sample 10% of successful requests
gcloud logging exclusions create sample-success-logs \
  --log-filter='
    severity="INFO" AND
    jsonPayload.status>=200 AND
    jsonPayload.status<300 AND
    sample(insertId, 0.1)  # Keep 10%
  ' \
  --description="Sample 90% of successful requests"

# Sample 50% of access logs
gcloud logging exclusions create sample-access-logs \
  --log-filter='
    logName=~".*nginx-access.*" AND
    sample(insertId, 0.5)  # Keep 50%
  ' \
  --description="Sample 50% of nginx access logs"
```

### 3. Use Appropriate Retention Periods

```ruby
# In application code: Don't log unnecessary detail
class ApplicationController < ActionController::API
  # Bad: Logs every parameter (potential PII leak + cost)
  # config.lograge.custom_options = ->(event) { event.payload[:params] }

  # Good: Log only essential fields
  config.lograge.custom_options = lambda do |event|
    {
      request_id: event.payload[:request_id],
      user_id: event.payload[:user_id],
      endpoint: "#{event.payload[:controller]}##{event.payload[:action]}",
      status: event.payload[:status],
      duration_ms: event.duration
      # Omit: full params, headers, request body
    }
  end
end
```

### 4. Monitor Your Logging Costs

```bash
# Create log-based metric for ingestion volume
gcloud logging metrics create log_ingestion_bytes \
  --description="Total log ingestion in bytes" \
  --value-extractor='EXTRACT(resource.labels.pod_name)' \
  --metric-kind=DELTA \
  --value-type=INT64

# Query current month's ingestion
gcloud logging read '
  logName="projects/PROJECT_ID/logs/cloudaudit.googleapis.com%2Fdata_access" AND
  protoPayload.serviceName="logging.googleapis.com" AND
  protoPayload.methodName="google.logging.v2.LoggingServiceV2.WriteLogEntries"
' \
  --limit=10 \
  --format=json | jq '.[] | .protoPayload.request.entries | length'

# Estimate monthly cost
# Monthly ingestion (GB) * $0.50 = Monthly logging cost
```

### 5. Cost Comparison Table

| Scenario | Monthly Volume | Strategy | Estimated Cost |
|----------|---------------|----------|----------------|
| Small app | 10 GB | Default (30d retention) | $0 (free tier) |
| Medium app | 100 GB | Exclusions + 30d retention | $25 |
| Large app | 500 GB | Exclusions + Sampling (50%) + BigQuery | $150 |
| Enterprise | 2 TB | Sampling + Tiered storage + GCS archive | $600 |

---

## Monitoring & Alerting

### Log-based Metrics

```bash
# Track error rate
gcloud logging metrics create error_rate \
  --description="Number of errors per minute" \
  --log-filter='severity="ERROR"' \
  --metric-kind=DELTA \
  --value-type=INT64

# Track slow requests (>1s)
gcloud logging metrics create slow_requests \
  --description="Requests slower than 1 second" \
  --log-filter='jsonPayload.duration_ms>1000' \
  --value-extractor='EXTRACT(jsonPayload.duration_ms)' \
  --metric-kind=DELTA \
  --value-type=INT64

# Track specific errors
gcloud logging metrics create database_errors \
  --description="Database connection errors" \
  --log-filter='
    severity="ERROR" AND
    jsonPayload.error_class=~".*ActiveRecord.*"
  ' \
  --metric-kind=DELTA \
  --value-type=INT64

# Track user actions (for analytics)
gcloud logging metrics create user_logins \
  --description="User login events" \
  --log-filter='jsonPayload.event="user.login"' \
  --metric-kind=DELTA \
  --value-type=INT64
```

### Alert Policies

```bash
# Alert on high error rate
gcloud alpha monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="High Error Rate Alert" \
  --condition-display-name="Error rate > 10/min for 5 min" \
  --condition-threshold-value=10 \
  --condition-threshold-duration=300s \
  --condition-threshold-comparison=COMPARISON_GT \
  --condition-filter='
    metric.type="logging.googleapis.com/user/error_rate" AND
    resource.type="k8s_container"
  ' \
  --documentation='
    ## High Error Rate Detected

    The application is experiencing elevated error rates.

    **Runbook:**
    1. Check recent deployments
    2. Review error logs: gcloud logging read "severity=ERROR" --limit=50
    3. Check application health: kubectl get pods
    4. Rollback if needed: kubectl rollout undo deployment/rails-app
  '

# Alert on slow requests
gcloud alpha monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="Slow Request Alert" \
  --condition-display-name="Avg response time > 2s for 10 min" \
  --condition-threshold-value=2000 \
  --condition-threshold-duration=600s \
  --condition-threshold-aggregation='
    alignmentPeriod: 60s
    perSeriesAligner: ALIGN_MEAN
  ' \
  --condition-filter='
    metric.type="logging.googleapis.com/user/slow_requests" AND
    resource.type="k8s_container"
  '

# Alert on missing logs (application down)
gcloud alpha monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="No Logs Received Alert" \
  --condition-display-name="No logs for 10 minutes" \
  --condition-threshold-value=1 \
  --condition-threshold-duration=600s \
  --condition-threshold-comparison=COMPARISON_LT \
  --condition-filter='
    metric.type="logging.googleapis.com/log_entry_count" AND
    resource.labels.namespace_name="production"
  '
```

### Dashboard Creation

```bash
# Create custom dashboard
cat > dashboard.json << 'EOF'
{
  "displayName": "Rails Application Logs Dashboard",
  "mosaicLayout": {
    "columns": 12,
    "tiles": [
      {
        "width": 6,
        "height": 4,
        "widget": {
          "title": "Error Rate",
          "xyChart": {
            "dataSets": [{
              "timeSeriesQuery": {
                "timeSeriesFilter": {
                  "filter": "metric.type=\"logging.googleapis.com/user/error_rate\"",
                  "aggregation": {
                    "alignmentPeriod": "60s",
                    "perSeriesAligner": "ALIGN_RATE"
                  }
                }
              }
            }]
          }
        }
      },
      {
        "width": 6,
        "height": 4,
        "widget": {
          "title": "Request Duration P95",
          "xyChart": {
            "dataSets": [{
              "timeSeriesQuery": {
                "timeSeriesFilter": {
                  "filter": "metric.type=\"logging.googleapis.com/user/slow_requests\"",
                  "aggregation": {
                    "alignmentPeriod": "60s",
                    "perSeriesAligner": "ALIGN_PERCENTILE_95"
                  }
                }
              }
            }]
          }
        }
      }
    ]
  }
}
EOF

gcloud monitoring dashboards create --config-from-file=dashboard.json
```

---

## Deployment Examples

### GKE Deployment (Native Integration)

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rails-app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rails-app
  template:
    metadata:
      labels:
        app: rails-app
        version: v1.0.0
    spec:
      serviceAccountName: rails-app-sa  # Workload Identity

      containers:
      - name: app
        image: gcr.io/PROJECT_ID/rails-app:latest

        env:
        # Enable stdout logging for GKE auto-collection
        - name: RAILS_LOG_TO_STDOUT
          value: "true"

        # Google Cloud configuration
        - name: GOOGLE_CLOUD_PROJECT
          value: "PROJECT_ID"

        # Kubernetes metadata (auto-injected by Downward API)
        - name: K8S_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name

        # Application configuration
        - name: RAILS_ENV
          value: "production"
        - name: APP_VERSION
          value: "v1.0.0"

        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"

        # Health checks
        livenessProbe:
          httpGet:
            path: /up
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10

        readinessProbe:
          httpGet:
            path: /up
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5

---
# Service account with Workload Identity
apiVersion: v1
kind: ServiceAccount
metadata:
  name: rails-app-sa
  namespace: production
  annotations:
    iam.gke.io/gcp-service-account: rails-app-logger@PROJECT_ID.iam.gserviceaccount.com
```

### Cloud Run Deployment

```yaml
# cloud-run-service.yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: rails-app
  labels:
    cloud.googleapis.com/location: asia-east1
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/minScale: "1"
        autoscaling.knative.dev/maxScale: "100"
    spec:
      serviceAccountName: rails-app-logger@PROJECT_ID.iam.gserviceaccount.com

      containers:
      - image: gcr.io/PROJECT_ID/rails-app:latest

        env:
        - name: RAILS_LOG_TO_STDOUT
          value: "true"
        - name: GOOGLE_CLOUD_PROJECT
          value: "PROJECT_ID"
        - name: RAILS_ENV
          value: "production"

        resources:
          limits:
            memory: "1Gi"
            cpu: "2"

        ports:
        - containerPort: 3000
```

```bash
# Deploy to Cloud Run
gcloud run deploy rails-app \
  --image gcr.io/PROJECT_ID/rails-app:latest \
  --platform managed \
  --region asia-east1 \
  --service-account rails-app-logger@PROJECT_ID.iam.gserviceaccount.com \
  --set-env-vars "RAILS_LOG_TO_STDOUT=true,GOOGLE_CLOUD_PROJECT=PROJECT_ID" \
  --memory 1Gi \
  --cpu 2 \
  --min-instances 1 \
  --max-instances 100 \
  --allow-unauthenticated
```

### Dockerfile Configuration

```dockerfile
# Dockerfile
FROM ruby:3.4-slim AS base

# Install dependencies
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y \
      build-essential \
      curl \
      libpq-dev && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives

WORKDIR /app

# Install gems
COPY Gemfile Gemfile.lock ./
RUN bundle config set --local deployment 'true' && \
    bundle config set --local without 'development test' && \
    bundle install && \
    rm -rf ~/.bundle/ "${BUNDLE_PATH}"/ruby/*/cache "${BUNDLE_PATH}"/ruby/*/bundler/gems/*/.git

# Copy application code
COPY . .

# Precompile assets (if needed)
# RUN SECRET_KEY_BASE=DUMMY bundle exec rails assets:precompile

# Runtime configuration
ENV RAILS_ENV=production \
    RAILS_LOG_TO_STDOUT=true \
    RAILS_SERVE_STATIC_FILES=true

EXPOSE 3000

CMD ["bundle", "exec", "puma", "-C", "config/puma.rb"]
```

### Using Fluentd Sidecar (Advanced)

```yaml
# deployment-with-fluentd.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rails-app-with-fluentd
spec:
  template:
    spec:
      containers:
      # Main application container
      - name: app
        image: gcr.io/PROJECT_ID/rails-app:latest
        volumeMounts:
        - name: log-volume
          mountPath: /app/log

      # Fluentd sidecar
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1-debian-stackdriver
        env:
        - name: FLUENT_STACKDRIVER_PROJECT_ID
          value: "PROJECT_ID"
        - name: FLUENT_STACKDRIVER_LOG_NAME
          value: "rails-app"
        volumeMounts:
        - name: log-volume
          mountPath: /app/log
          readOnly: true
        - name: fluentd-config
          mountPath: /fluentd/etc

      volumes:
      - name: log-volume
        emptyDir: {}
      - name: fluentd-config
        configMap:
          name: fluentd-config

---
# Fluentd configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluent.conf: |
    <source>
      @type tail
      path /app/log/production.log
      pos_file /tmp/production.log.pos
      tag rails.app
      <parse>
        @type json
        time_key timestamp
        time_format %Y-%m-%dT%H:%M:%S.%L%z
      </parse>
    </source>

    <filter rails.app>
      @type record_transformer
      <record>
        severity ${record["severity"] || "INFO"}
        app rails
        environment production
      </record>
    </filter>

    <filter rails.app>
      @type grep
      <exclude>
        key path
        pattern ^/(up|health|readiness)$
      </exclude>
    </filter>

    <match rails.app>
      @type google_cloud
      project_id "#{ENV['FLUENT_STACKDRIVER_PROJECT_ID']}"
      log_name "#{ENV['FLUENT_STACKDRIVER_LOG_NAME']}"
      <buffer>
        @type file
        path /tmp/fluentd-buffer
        flush_interval 5s
        retry_max_interval 30s
        chunk_limit_size 5M
      </buffer>
    </match>
```

---

## Implementation Checklist

### Phase 1: Setup (Day 1)

- [ ] Enable Cloud Logging API
  ```bash
  gcloud services enable logging.googleapis.com
  ```

- [ ] Create service account and grant permissions
  ```bash
  gcloud iam service-accounts create rails-app-logger
  gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="serviceAccount:rails-app-logger@PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/logging.logWriter"
  ```

- [ ] Set up Workload Identity (for GKE)
  ```bash
  gcloud iam service-accounts add-iam-policy-binding \
    rails-app-logger@PROJECT_ID.iam.gserviceaccount.com \
    --role roles/iam.workloadIdentityUser \
    --member "serviceAccount:PROJECT_ID.svc.id.goog[default/rails-app-sa]"
  ```

- [ ] Add google-cloud-logging gem to Gemfile
  ```ruby
  gem 'google-cloud-logging'
  gem 'google-cloud-logging-rails'
  ```

- [ ] Configure Rails production environment
  - Enable Cloud Logging
  - Set up Lograge for structured logging
  - Configure custom fields

### Phase 2: Log Retention (Day 2)

- [ ] Create log buckets with appropriate retention
  ```bash
  gcloud logging buckets create short-term --retention-days=30
  gcloud logging buckets create medium-term --retention-days=90
  gcloud logging buckets create long-term --retention-days=3650
  ```

- [ ] Create log sinks for routing
  - INFO/DEBUG → short-term bucket
  - ERROR → medium-term bucket
  - Audit logs → long-term bucket

- [ ] Set up BigQuery export (optional)
  ```bash
  bq mk --dataset PROJECT_ID:logs
  gcloud logging sinks create logs-to-bigquery \
    bigquery.googleapis.com/projects/PROJECT_ID/datasets/logs
  ```

- [ ] Set up Cloud Storage archive (optional)
  ```bash
  gsutil mb -l asia-east1 gs://PROJECT_ID-logs-archive
  gcloud logging sinks create logs-to-gcs \
    storage.googleapis.com/PROJECT_ID-logs-archive
  ```

### Phase 3: Cost Optimization (Day 3)

- [ ] Create exclusion filters
  - Health checks: `/up`, `/health`, `/readiness`
  - Static assets: `/assets/*`, `*.css`, `*.js`
  - Debug logs: `severity="DEBUG"`

- [ ] Enable sampling for high-volume logs
  - Sample 90% of successful requests (200-299 status)
  - Sample 50% of access logs

- [ ] Review and optimize log payload size
  - Remove sensitive data (passwords, tokens)
  - Exclude unnecessary parameters
  - Limit request/response body logging

- [ ] Set up cost monitoring
  ```bash
  gcloud logging metrics create log_ingestion_bytes
  # Create alert for unexpected cost increase
  ```

### Phase 4: Monitoring & Alerting (Day 4)

- [ ] Create log-based metrics
  - Error rate
  - Slow requests (>1s)
  - Database errors
  - User events

- [ ] Set up alert policies
  - High error rate (>10/min for 5 min)
  - No logs received (10 min window)
  - Slow requests (avg >2s for 10 min)

- [ ] Create monitoring dashboard
  - Error rate chart
  - Request duration P50/P95/P99
  - Log ingestion volume
  - Top errors table

- [ ] Configure notification channels
  ```bash
  gcloud alpha monitoring channels create \
    --display-name="Slack Alerts" \
    --type=slack \
    --channel-labels=url=WEBHOOK_URL
  ```

### Phase 5: Testing & Validation (Day 5)

- [ ] Test local logging (development environment)
  ```bash
  GOOGLE_CLOUD_PROJECT="" rails s
  # Should fall back to STDOUT logging
  ```

- [ ] Test Cloud Logging (staging environment)
  ```bash
  # Deploy to staging
  # Generate test logs
  curl https://staging.example.com/api/test

  # Verify logs appear in Cloud Logging
  gcloud logging read "resource.labels.namespace_name=staging" --limit=10
  ```

- [ ] Verify log routing
  - Check short-term bucket for INFO logs
  - Check medium-term bucket for ERROR logs
  - Check BigQuery for exported logs

- [ ] Test exclusion filters
  ```bash
  # Generate health check requests
  while true; do curl https://app.example.com/up; sleep 1; done

  # Verify they don't appear in logs
  gcloud logging read 'jsonPayload.path="/up"' --limit=10
  ```

- [ ] Test alerts
  ```bash
  # Trigger error condition (e.g., generate 20 errors)
  # Verify alert fires within expected time
  ```

### Phase 6: Documentation (Day 6)

- [ ] Document logging architecture
  - Log flow diagram
  - Retention policies
  - Cost breakdown

- [ ] Create runbooks
  - How to query logs
  - How to troubleshoot common issues
  - How to analyze logs in BigQuery

- [ ] Document alert response procedures
  - High error rate → Check recent deployments
  - No logs → Check pod health
  - Slow requests → Check database performance

- [ ] Train team on Cloud Logging console
  - Basic log queries
  - Log Explorer filters
  - Creating custom queries

### Phase 7: Production Rollout (Day 7)

- [ ] Enable Cloud Logging in production
  - Update Kubernetes deployment
  - Set `GOOGLE_CLOUD_PROJECT` env var
  - Deploy with Workload Identity

- [ ] Monitor for 24 hours
  - Check log ingestion rate
  - Verify alerts are working
  - Monitor costs in billing console

- [ ] Tune exclusion filters based on actual traffic
  - Adjust sampling rates
  - Add/remove excluded paths
  - Optimize log payload size

- [ ] Schedule regular reviews
  - Weekly: Review top errors
  - Monthly: Review logging costs
  - Quarterly: Review retention policies

---

## Troubleshooting

### Issue: Logs not appearing in Cloud Logging

**Possible causes:**
1. Missing IAM permissions
2. Wrong project ID
3. Invalid credentials
4. Logs filtered by exclusion rules

**Debug steps:**
```bash
# Check service account permissions
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:rails-app-logger@*"

# Test authentication
gcloud auth activate-service-account --key-file=keyfile.json
gcloud logging write test-log "Test message" --severity=INFO

# Check exclusion filters
gcloud logging exclusions list

# Enable debug logging in Rails
# config/environments/production.rb
config.google_cloud.logging.log_level = Logger::DEBUG
```

### Issue: High logging costs

**Possible causes:**
1. No exclusion filters for health checks
2. Logging full request/response bodies
3. Debug logs in production
4. No sampling for high-volume endpoints

**Solutions:**
```bash
# Review top log sources
gcloud logging read --format=json --limit=1000 | \
  jq -r '.[] | .logName' | sort | uniq -c | sort -rn

# Check ingestion volume by log name
gcloud logging read '
  protoPayload.methodName="google.logging.v2.LoggingServiceV2.WriteLogEntries"
' --format=json | jq '.[] | .protoPayload.request.entries | length'

# Add exclusion for top offenders
gcloud logging exclusions create exclude-noisy-logs \
  --log-filter='logName="projects/PROJECT_ID/logs/NOISY_LOG"'
```

### Issue: Workload Identity not working

**Possible causes:**
1. Workload Identity not enabled on cluster
2. Missing IAM binding
3. Wrong service account annotation

**Debug steps:**
```bash
# Check Workload Identity is enabled
gcloud container clusters describe CLUSTER_NAME \
  --format="value(workloadIdentityConfig.workloadPool)"

# Verify IAM binding
gcloud iam service-accounts get-iam-policy \
  rails-app-logger@PROJECT_ID.iam.gserviceaccount.com

# Check Kubernetes service account annotation
kubectl get serviceaccount rails-app-sa -o yaml

# Test from inside pod
kubectl run -it --rm debug --image=google/cloud-sdk:slim --restart=Never -- bash
gcloud auth list
gcloud logging write test "Test from pod"
```

### Issue: Missing fields in structured logs

**Possible causes:**
1. Lograge not parsing correctly
2. Missing custom_options configuration
3. JSON parsing errors

**Debug steps:**
```ruby
# Test Lograge configuration
Rails.logger.info("Test structured log", {
  user_id: 123,
  action: "test",
  duration_ms: 456
})

# Check raw log output
tail -f log/production.log

# Verify JSON format
tail -1 log/production.log | jq .

# Enable Lograge debugging
config.lograge.logger = ActiveSupport::Logger.new(STDOUT)
```

---

## References

- [Google Cloud Logging Documentation](https://cloud.google.com/logging/docs)
- [google-cloud-logging Ruby Gem](https://github.com/googleapis/google-cloud-ruby/tree/main/google-cloud-logging)
- [Lograge GitHub](https://github.com/roidrage/lograge)
- [GKE Logging Best Practices](https://cloud.google.com/architecture/best-practices-for-operating-containers#logging)
- [Cloud Logging Pricing](https://cloud.google.com/stackdriver/pricing)

---

**Last Updated:** 2025-10-28
**Version:** 1.0
**Maintainer:** jlhg
