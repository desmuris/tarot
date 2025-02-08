# Deployment Guide

## Deployment Configuration

### Application Configuration
```yaml
# config/deployment.yaml
app:
  name: tarot-bot
  version: 1.0.0
  environment: production

server:
  host: 0.0.0.0
  port: 8000
  workers: 4
  max_requests_per_worker: 10000
  keepalive: 120
  timeout: 120
  max_request_size: 10485760  # 10MB

security:
  allowed_hosts: ["api.tarot-bot.com"]
  cors_origins: ["https://tarot-bot.com"]
  ssl_redirect: true
  hsts_seconds: 31536000
  frame_deny: true
  content_security_policy: "default-src 'self'"
  rate_limiting:
    default:
      requests: 100
      window_seconds: 60
    premium:
      requests: 300
      window_seconds: 60

database:
  host: localhost
  port: 5432
  name: tarot_db
  user: ${DB_USER}
  password: ${DB_PASSWORD}
  pool_size: 20
  max_overflow: 10
  pool_timeout: 30
  pool_recycle: 1800
  ssl_mode: require

redis:
  host: localhost
  port: 6379
  db: 0
  password: ${REDIS_PASSWORD}
  ssl: true
  pool_size: 10
  ttl: 3600
  max_memory: 1GB
  eviction_policy: volatile-lru

monitoring:
  log_level: INFO
  metrics_port: 9090
  tracing_enabled: true
  profiling_enabled: true
  health_check_interval: 30
  alerting:
    slack_webhook: ${SLACK_WEBHOOK}
    email: alerts@tarot-bot.com

backup:
  database:
    schedule: "0 */6 * * *"  # Every 6 hours
    retention_days: 30
    storage:
      type: s3
      bucket: tarot-bot-backups
      path: db-backups
  redis:
    schedule: "0 */12 * * *"  # Every 12 hours
    retention_days: 7

resources:
  cpu_limit: "2"
  memory_limit: "4Gi"
  storage:
    data: "50Gi"
    backups: "100Gi"

scaling:
  min_replicas: 2
  max_replicas: 10
  target_cpu_utilization: 70
  target_memory_utilization: 80
```

## Component Details

### 1. Load Balancer
- Basic HTTP/HTTPS routing
- SSL termination
- Health checks
- Rate limiting
- DDoS protection

### 2. Application Server
- Python FastAPI application
- Modular monolith structure
- Environment-based configuration
- Graceful shutdown handling
- Request timeout management

### 3. Database (PostgreSQL)
- High availability setup
- Regular backups
- Connection pooling
- Query optimization
- Data retention policies

### 4. Cache (Redis)
- Session management
- Rate limiting
- Result caching
- Memory management
- Eviction policies

### 5. Monitoring
- Health metrics
- Error logging
- Performance tracking
- Alert management
- Resource utilization

## Deployment Process

### 1. Pre-deployment Checks
- Run test suite
- Verify dependencies
- Check configuration
- Validate environment variables
- Review security settings

### 2. Database Migration
```bash
# Run database migrations
python scripts/migrate.py

# Verify migration status
python scripts/verify_migrations.py
```

### 3. Application Deployment
```bash
# Build application
python -m build

# Deploy to server
python scripts/deploy.py --env production

# Verify deployment
python scripts/health_check.py
```

### 4. Post-deployment Tasks
- Verify application health
- Check database connectivity
- Validate cache operation
- Monitor error rates
- Review performance metrics

## Monitoring Setup

### Health Checks
```python
@app.get("/health")
async def health_check():
    """
    Comprehensive health check endpoint:
    - Database connectivity
    - Redis connectivity
    - API dependencies
    - System resources
    """
    checks = {
        "database": await check_database(),
        "redis": await check_redis(),
        "openai": await check_openai_api(),
        "telegram": await check_telegram_api()
    }
    
    return {
        "status": "healthy" if all(checks.values()) else "unhealthy",
        "checks": checks,
        "timestamp": datetime.utcnow().isoformat()
    }
```

### Performance Monitoring
```python
@app.middleware("http")
async def performance_monitoring(request: Request, call_next):
    """Monitor request performance and resource usage"""
    start_time = time.time()
    
    response = await call_next(request)
    
    # Calculate metrics
    duration = time.time() - start_time
    
    # Record metrics
    metrics.record_request(
        path=request.url.path,
        method=request.method,
        duration=duration,
        status_code=response.status_code
    )
    
    return response
```

## Backup Strategy

### Database Backups
- Full backup every 6 hours
- Transaction log backup every hour
- 30-day retention period
- Encrypted storage in S3
- Regular restore testing

### Redis Backups
- RDB snapshot every 12 hours
- AOF persistence enabled
- 7-day retention period
- Automated recovery testing

## Security Measures

### SSL/TLS Configuration
- TLS 1.3 enabled
- Strong cipher suites
- HSTS enabled
- Certificate auto-renewal
- Regular security scans

### Access Control
- Role-based access
- IP whitelisting
- API key rotation
- Audit logging
- Failed login monitoring

## Related Documentation
- [System Architecture](architecture.md) - System design and components
- [Data Management](data-management.md) - Database operations
- [Development Guide](development.md) - Development workflow