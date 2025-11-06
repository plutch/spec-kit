# Deployment Runbook & Operations

**Inclusion Mode**: Manual
**Trigger**: Use `@deployment-runbook.md` when working on deployment, infrastructure, or operational tasks
**Purpose**: Deployment procedures, infrastructure configuration, and operational guidelines

---

## Deployment Overview

### Deployment Strategy
<!-- Choose: Blue-Green, Rolling, Canary, Recreate -->
- **Strategy**: [Blue-Green / Rolling / Canary / Recreate]
- **Frequency**: [On-demand / Daily / Weekly / Per Sprint]
- **Approval**: [Automated / Manual / Hybrid]

### Environments
```
Development → Staging → Production
     ↓           ↓           ↓
  feature   pre-release   stable
   tests      QA/UAT      users
```

| Environment | Purpose | Deploy Trigger | Rollback Policy |
|-------------|---------|----------------|-----------------|
| Development | Feature dev | Auto on merge to `dev` | Manual |
| Staging | QA/UAT | Auto on merge to `staging` | Manual |
| Production | Live users | Manual approval | Automatic on health check failure |

## Pre-Deployment Checklist

### Code Quality Gates
- [ ] All tests passing (unit, integration, E2E)
- [ ] Code coverage meets threshold (≥ 75%)
- [ ] No critical security vulnerabilities (Snyk, Dependabot)
- [ ] Code review approved by 2+ reviewers
- [ ] Linter and formatter checks pass

### Documentation
- [ ] CHANGELOG.md updated with release notes
- [ ] API documentation updated (if API changes)
- [ ] Migration guides written (if breaking changes)
- [ ] Feature flags documented

### Database Migrations
- [ ] Migration scripts tested in staging
- [ ] Rollback script prepared
- [ ] Data backup verified
- [ ] Migration execution time estimated (< 5min acceptable)

### Infrastructure
- [ ] Resource capacity checked (CPU, memory, disk, network)
- [ ] Scaling policies reviewed
- [ ] DNS/Load balancer configuration verified
- [ ] SSL certificates valid (> 30 days remaining)

### Monitoring & Alerting
- [ ] Dashboards updated with new metrics
- [ ] Alerts configured for new features
- [ ] On-call schedule confirmed
- [ ] Incident response plan reviewed

## Deployment Procedure

### Step 1: Pre-Deployment
```bash
# 1. Verify environment
./scripts/verify-environment.sh production

# 2. Database backup
./scripts/backup-database.sh production

# 3. Health check baseline
./scripts/health-check.sh production > pre-deploy-health.log
```

### Step 2: Deployment
<!-- Adjust based on deployment tool: Kubernetes, Docker, Heroku, AWS, etc. -->

#### Option A: Kubernetes (Blue-Green)
```bash
# 1. Deploy new version (blue)
kubectl apply -f k8s/deployment-blue.yml

# 2. Wait for pods ready
kubectl wait --for=condition=ready pod -l app=myapp,version=blue --timeout=300s

# 3. Run smoke tests
./scripts/smoke-test.sh blue

# 4. Switch traffic (update service selector)
kubectl patch service myapp -p '{"spec":{"selector":{"version":"blue"}}}'

# 5. Monitor for 10 minutes
./scripts/monitor-deployment.sh 600

# 6. If stable, delete old version (green)
kubectl delete deployment myapp-green
```

#### Option B: Docker Compose
```bash
# 1. Pull latest images
docker-compose pull

# 2. Stop old containers
docker-compose down

# 3. Start new containers
docker-compose up -d

# 4. Check logs
docker-compose logs -f --tail=100
```

#### Option C: Serverless (AWS Lambda)
```bash
# 1. Deploy new version
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip

# 2. Publish version
VERSION=$(aws lambda publish-version --function-name my-function --query 'Version' --output text)

# 3. Update alias (weighted routing for canary)
aws lambda update-alias \
  --function-name my-function \
  --name production \
  --function-version $VERSION \
  --routing-config AdditionalVersionWeights={"$OLD_VERSION"=0.9,"$VERSION"=0.1}
```

### Step 3: Post-Deployment Verification
```bash
# 1. Health checks
./scripts/health-check.sh production > post-deploy-health.log
diff pre-deploy-health.log post-deploy-health.log

# 2. Smoke tests (critical user paths)
npm run test:smoke -- --env=production

# 3. Check error rates
./scripts/check-error-rate.sh 5m  # Last 5 minutes

# 4. Verify metrics
./scripts/verify-metrics.sh
```

### Step 4: Monitoring Window
<!-- Monitor for anomalies after deployment -->
- **Duration**: 30 minutes (critical features) / 2 hours (major releases)
- **Watch**: Error rate, latency (p95, p99), throughput, CPU, memory
- **Alert thresholds**: Error rate > 1%, latency > 2x baseline, 5xx errors

## Rollback Procedure

### Automated Rollback Triggers
- Error rate > 5% for 2 minutes
- p99 latency > 5x baseline for 5 minutes
- Health check failures > 50% of instances

### Manual Rollback
```bash
# Option A: Kubernetes (revert to previous version)
kubectl rollout undo deployment/myapp

# Option B: Docker Compose (use previous image tag)
docker-compose down
docker-compose up -d --force-recreate --build --no-deps myapp:previous

# Option C: AWS Lambda (update alias to previous version)
aws lambda update-alias \
  --function-name my-function \
  --name production \
  --function-version $PREVIOUS_VERSION \
  --routing-config '{}'
```

### Post-Rollback
1. Verify application is stable (health checks, metrics)
2. Investigate root cause (logs, traces, metrics)
3. Create incident report
4. Fix issue in development
5. Plan re-deployment

## Database Migrations

### Running Migrations
```bash
# Development
npm run migrate:dev

# Staging
npm run migrate:staging

# Production (manual approval required)
npm run migrate:prod
```

### Migration Best Practices
- **Backwards compatible**: Old code works with new schema during deployment
- **Zero-downtime**: Use multi-phase migrations (add column → deploy code → backfill data → remove old column)
- **Reversible**: Always write `down` migration
- **Tested**: Run in staging first, verify rollback works

### Example: Zero-Downtime Column Rename
```sql
-- Phase 1: Add new column
ALTER TABLE users ADD COLUMN email_address VARCHAR(255);

-- Phase 2: Backfill data (deploy app that writes to both columns)
UPDATE users SET email_address = email WHERE email_address IS NULL;

-- Phase 3: Deploy app that reads from email_address only

-- Phase 4: Drop old column (after stability period)
ALTER TABLE users DROP COLUMN email;
```

## Infrastructure as Code (IaC)

### Tools
<!-- Choose based on cloud provider -->
- **Terraform**: Multi-cloud provisioning
- **CloudFormation**: AWS-native
- **Pulumi**: Code-first infrastructure
- **Ansible**: Configuration management

### Repository Structure
```
infrastructure/
├── terraform/
│   ├── modules/          # Reusable modules
│   ├── environments/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── prod/
│   └── backend.tf        # Remote state config
└── ansible/
    ├── playbooks/
    └── roles/
```

### Best Practices
- **Version control**: All infrastructure code in Git
- **Remote state**: Use S3, GCS, or Terraform Cloud
- **State locking**: Prevent concurrent modifications
- **Modules**: Reusable, versioned infrastructure components
- **Secrets management**: Never commit secrets (use Vault, AWS Secrets Manager, etc.)

## Monitoring & Observability

### Metrics to Monitor
<!-- Adjust based on application type -->

**Application Metrics**:
- Request rate (requests/sec)
- Error rate (errors/sec, error %)
- Latency (p50, p95, p99)
- Throughput (MB/s)

**Infrastructure Metrics**:
- CPU utilization (%)
- Memory utilization (%)
- Disk I/O (IOPS, MB/s)
- Network I/O (packets/s, MB/s)

**Business Metrics**:
- Active users
- Conversion rate
- Revenue per request
- Feature usage

### Dashboards
<!-- Create in Grafana, Datadog, New Relic, CloudWatch, etc. -->
1. **System Health**: CPU, memory, disk, network
2. **Application Performance**: Request rate, error rate, latency
3. **Business Metrics**: User activity, conversions, revenue
4. **Database**: Query performance, connection pool, slow queries

### Alerting Rules
| Metric | Threshold | Severity | Action |
|--------|-----------|----------|--------|
| Error rate | > 1% for 5min | Warning | Investigate |
| Error rate | > 5% for 2min | Critical | Page on-call, consider rollback |
| p99 latency | > 2s for 10min | Warning | Investigate |
| CPU | > 80% for 15min | Warning | Check for resource leak |
| Disk space | < 10% free | Critical | Free up space immediately |

## Logging

### Log Levels
- **ERROR**: Application errors, exceptions (always logged)
- **WARN**: Potential issues, deprecated usage
- **INFO**: Key application events (startup, shutdown, major operations)
- **DEBUG**: Detailed diagnostic information (development/staging only)

### Structured Logging
```json
{
  "timestamp": "2025-11-05T12:34:56.789Z",
  "level": "ERROR",
  "service": "user-service",
  "request_id": "abc123",
  "user_id": "user-456",
  "message": "Failed to create user",
  "error": {
    "type": "ValidationError",
    "message": "Invalid email format",
    "stack": "..."
  }
}
```

### Log Aggregation
<!-- Choose based on scale -->
- **ELK Stack**: Elasticsearch, Logstash, Kibana
- **Grafana Loki**: Lightweight, cost-effective
- **Datadog Logs**: Cloud-native, full observability
- **CloudWatch Logs**: AWS-native

## Security Operations

### Secrets Management
<!-- Choose based on infrastructure -->
- **AWS Secrets Manager / Parameter Store**
- **HashiCorp Vault**
- **Azure Key Vault**
- **Google Secret Manager**

**Never commit secrets to Git**. Use environment variables, secrets management services, or encrypted configuration.

### SSL/TLS Certificates
- **Issuer**: Let's Encrypt, AWS ACM, DigiCert
- **Renewal**: Automated via cert-bot or cloud provider
- **Monitoring**: Alert 30 days before expiration

### Vulnerability Scanning
- **Tools**: Snyk, Dependabot, Trivy, Clair
- **Frequency**: Every commit (CI/CD), weekly scans
- **Action**: Critical vulnerabilities block deployment

## Backup & Disaster Recovery

### Backup Strategy
| Resource | Frequency | Retention | Tested |
|----------|-----------|-----------|--------|
| Database | Daily (full), Hourly (incremental) | 30 days | Monthly |
| File storage | Daily | 90 days | Monthly |
| Configuration | On change (Git) | Forever | N/A |

### Disaster Recovery Plan
1. **RTO (Recovery Time Objective)**: [Target: 4 hours]
2. **RPO (Recovery Point Objective)**: [Target: 1 hour]
3. **Backup restoration procedure**: [Document steps]
4. **Failover procedure**: [Document steps for multi-region failover]
5. **Test frequency**: Quarterly DR drill

## On-Call & Incident Response

### On-Call Rotation
- **Schedule**: Weekly rotation, primary + secondary
- **Response SLA**: Acknowledge within 15 min, engage within 30 min
- **Tools**: PagerDuty, Opsgenie, VictorOps

### Incident Response Process
1. **Detect**: Alert triggers, user report, monitoring
2. **Acknowledge**: On-call engineer acknowledges incident
3. **Assess**: Determine severity (P0-Critical, P1-High, P2-Medium, P3-Low)
4. **Mitigate**: Immediate actions to reduce impact (rollback, disable feature)
5. **Resolve**: Root cause fix, deploy resolution
6. **Postmortem**: Blameless review within 48 hours

### Incident Severity Levels
- **P0 (Critical)**: Complete outage, data loss, security breach → Page immediately
- **P1 (High)**: Major feature degradation, significant user impact → Page during business hours
- **P2 (Medium)**: Minor feature issue, small user impact → Ticket, fix in 24h
- **P3 (Low)**: Cosmetic issue, no user impact → Ticket, fix in next sprint

---

## Usage Instructions

This file is **manually loaded** via `@deployment-runbook.md` when working on deployment, infrastructure, or operational tasks. It provides AI assistants with deployment procedures and operational context.

**Last Updated**: [YYYY-MM-DD]
**Next Review**: [YYYY-MM-DD]
