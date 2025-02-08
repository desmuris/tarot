# System Monitoring Documentation

## Overview
This document details the comprehensive monitoring strategy, metrics collection, alerting criteria, and response procedures for the Tarot Reading Bot system. It ensures consistent monitoring across all system components.

## Key Metrics

### System Health
1. Application Metrics
   - Request latency (p95, p99)
   - Error rates by category
   - Success rates by operation
   - Active users and sessions
   - Concurrent operations
   - API endpoint performance

2. Resource Utilization
   - CPU usage (warning: 70%, critical: 85%)
   - Memory usage (warning: 80%, critical: 90%)
   - Disk space (warning: 85%, critical: 95%)
   - Network bandwidth and latency
   - Connection pool utilization
   - Queue depths and processing rates

3. Database Metrics
   - Query performance and latency
   - Connection pool status
   - Transaction throughput
   - Lock contention and deadlocks
   - Replication lag
   - Index effectiveness
   - Cache hit rates

4. Cache Metrics
   - Hit/miss rates by cache type
   - Eviction rates and patterns
   - Memory usage and fragmentation
   - Key expiration metrics
   - Connection status
   - Cache consistency metrics

## Performance Baselines

### API Performance
| Endpoint Category    | p95 Latency | p99 Latency | Error Rate |
|---------------------|-------------|-------------|------------|
| Reading Generation  | 3s         | 5s         | < 0.1%     |
| Credit Operations  | 500ms      | 1s         | < 0.01%    |
| User Operations    | 200ms      | 500ms      | < 0.05%    |
| Payment Processing | 2s         | 4s         | < 0.1%     |
| Cache Operations   | 100ms      | 200ms      | < 0.01%    |
| Database Queries   | 300ms      | 600ms      | < 0.05%    |

### System Performance
| Metric              | Warning     | Critical    | Action Required |
|--------------------|-------------|-------------|-----------------|
| CPU Usage          | 70%        | 85%        | Scale up/out    |
| Memory Usage       | 80%        | 90%        | Investigate leaks|
| Disk Usage         | 85%        | 95%        | Cleanup/expand   |
| Error Rate         | 1%         | 5%         | Investigation    |
| Response Time      | +50%       | +100%      | Performance review|
| Failed Payments    | 3/15min    | 10/15min   | Payment system check|
| Failed Readings    | 5/15min    | 20/15min   | Reading system check|

## Alert Configuration

### Priority Levels
1. P1 - Critical (Response: Immediate)
   - System downtime
   - Data loss risk
   - Payment system failure
   - Security breach
   - Critical service outage

2. P2 - High (Response: < 30 minutes)
   - Performance degradation
   - High error rates
   - Component failures
   - Resource exhaustion
   - Payment processing delays

3. P3 - Medium (Response: < 2 hours)
   - Elevated error rates
   - Resource warnings
   - Backup failures
   - Non-critical services
   - Performance warnings

4. P4 - Low (Response: Next business day)
   - Minor anomalies
   - Performance variations
   - Warning thresholds
   - Routine checks
   - Non-urgent maintenance

### Alert Thresholds

#### System Alerts
```yaml
alerts:
  system:
    cpu_usage:
      warning: 70
      critical: 85
      duration: "5m"
      priority: "P2"
      
    memory_usage:
      warning: 80
      critical: 90
      duration: "5m"
      priority: "P2"
      
    disk_space:
      warning: 85
      critical: 95
      duration: "15m"
      priority: "P2"
      
    error_rate:
      warning: 1
      critical: 5
      duration: "5m"
      priority: "P1"
```

#### Application Alerts
```yaml
alerts:
  application:
    response_time:
      warning: "p95 > 2s"
      critical: "p95 > 5s"
      duration: "5m"
      priority: "P2"
      
    error_rate:
      warning: "1%"
      critical: "5%"
      duration: "5m"
      priority: "P1"
      
    failed_readings:
      warning: 5
      critical: 20
      duration: "15m"
      priority: "P2"
      
    payment_failures:
      warning: 3
      critical: 10
      duration: "15m"
      priority: "P1"
```

## Monitoring Implementation

### Metrics Collection
```python
class MetricsCollector:
    """Comprehensive metrics collection system"""
    
    async def collect_metrics(self):
        """Collect all system metrics"""
        metrics = {
            "system": await self.get_system_metrics(),
            "application": await self.get_app_metrics(),
            "database": await self.get_db_metrics(),
            "cache": await self.get_cache_metrics(),
            "payment": await self.get_payment_metrics(),
            "reading": await self.get_reading_metrics()
        }
        
        await self.store_metrics(metrics)
        await self.check_thresholds(metrics)
        await self.update_dashboards(metrics)
```

### Health Checks
```python
class HealthChecker:
    """System health monitoring"""
    
    async def check_health(self) -> HealthStatus:
        """Perform comprehensive health check"""
        checks = {
            "database": await self.check_database(),
            "redis": await self.check_redis(),
            "openai": await self.check_openai_api(),
            "telegram": await self.check_telegram_api(),
            "payment": await self.check_payment_system(),
            "credit": await self.check_credit_system()
        }
        
        status = self.calculate_overall_status(checks)
        await self.record_health_status(status)
        
        return HealthStatus(
            status=status,
            checks=checks,
            timestamp=datetime.utcnow().isoformat()
        )
```

## Incident Response

### Response Procedures
1. Alert Detection
   - Alert received and classified
   - Priority assessment
   - Team notification
   - Initial investigation started

2. Investigation
   - Log analysis and metric review
   - Impact assessment
   - Root cause analysis
   - Service status verification
   - User impact evaluation

3. Resolution
   - Apply fix or mitigation
   - Verify solution effectiveness
   - Monitor recovery
   - Update documentation
   - User communication if needed

4. Post-mortem
   - Incident review meeting
   - Prevention measures identified
   - Documentation updates
   - Process improvements
   - Team training needs

## Dashboard Configuration

### Main Dashboard
1. System Overview
   - Overall health status
   - Key performance metrics
   - Active alerts
   - Recent incidents
   - System capacity

2. Performance Metrics
   - Response times by endpoint
   - Error rates by category
   - Resource utilization
   - User activity patterns
   - API performance

3. Business Metrics
   - Reading completion rates
   - Credit transaction volume
   - User engagement metrics
   - Revenue analytics
   - Subscription status

### Technical Dashboards
1. Database Performance
   - Query performance
   - Connection pools
   - Lock statistics
   - Index usage
   - Transaction rates

2. Cache Performance
   - Hit rates
   - Memory usage
   - Eviction rates
   - Key statistics
   - Cache efficiency

3. API Performance
   - Endpoint latency
   - Error rates
   - Usage patterns
   - Rate limiting
   - Integration status

## Related Documentation
- [Error Handling](error-handling.md) - Error management procedures
- [Deployment](deployment.md) - Deployment monitoring
- [Security](security.md) - Security monitoring
- [Credit System](credit-system.md) - Credit system metrics
- [Payment System](payment-system.md) - Payment system metrics