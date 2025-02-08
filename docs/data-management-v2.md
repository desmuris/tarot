# Data Management System

## Document Information
- **Last Updated:** 2025-02-08
- **Version:** 1.0.0
- **Status:** Draft
- **Owner:** Data Management Team

## Overview
The Data Management System handles all data storage, retrieval, caching, and optimization for the Tarot Reading Bot. It ensures data integrity, performance, and security through a well-structured Database schema and multi-layer Cache strategy.

## Background
The system requires robust data management to handle User Profile information, Reading History, transactions, and credit balances. The implementation uses Database (PostgreSQL) for persistent storage and Cache (Redis) for caching, with careful consideration for data integrity, performance, and security.

## Detailed Content

### Database Schema Design

#### Users Table
Primary User Profile information storage:
```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    telegram_id BIGINT UNIQUE NOT NULL,
    username VARCHAR(255),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    last_active_at TIMESTAMP WITH TIME ZONE,
    is_active BOOLEAN DEFAULT true,
    subscription_tier VARCHAR(50) DEFAULT 'basic',
    subscription_expires_at TIMESTAMP WITH TIME ZONE,
    CONSTRAINT valid_subscription_tier CHECK (subscription_tier IN ('basic', 'premium', 'professional'))
);

CREATE INDEX idx_users_telegram_id ON users(telegram_id);
CREATE INDEX idx_users_subscription_expires ON users(subscription_expires_at) WHERE subscription_expires_at IS NOT NULL;
```

#### Credits Table
Credit balance management:
```sql
CREATE TABLE credits (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    balance INTEGER NOT NULL DEFAULT 0,
    last_updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    version INTEGER DEFAULT 1,
    CONSTRAINT positive_balance CHECK (balance >= 0)
);

CREATE UNIQUE INDEX idx_credits_user_id ON credits(user_id);
```

#### Transactions Table
Financial transaction tracking:
```sql
CREATE TABLE transactions (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    type VARCHAR(50) NOT NULL,
    amount INTEGER NOT NULL,
    credits_before INTEGER NOT NULL,
    credits_after INTEGER NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'pending',
    payment_reference VARCHAR(255),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP WITH TIME ZONE,
    error_message TEXT,
    CONSTRAINT valid_transaction_type CHECK (type IN ('purchase', 'reading', 'refund', 'bonus')),
    CONSTRAINT valid_transaction_status CHECK (status IN ('pending', 'completed', 'failed', 'refunded'))
);

CREATE INDEX idx_transactions_user_id ON transactions(user_id);
CREATE INDEX idx_transactions_created_at ON transactions(created_at);
CREATE INDEX idx_transactions_status ON transactions(status) WHERE status = 'pending';
```

#### Readings Table
Reading History storage:
```sql
CREATE TABLE readings (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    spread_type VARCHAR(50) NOT NULL,
    question TEXT,
    cards JSONB NOT NULL,
    interpretation TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    transaction_id BIGINT REFERENCES transactions(id),
    CONSTRAINT valid_spread_type CHECK (spread_type IN ('single', 'three', 'celtic', 'relationship', 'career', 'week_ahead', 'monthly_guidance', 'yearly_path', 'life_path'))
);

CREATE INDEX idx_readings_user_id ON readings(user_id);
CREATE INDEX idx_readings_created_at ON readings(created_at);
CREATE INDEX idx_readings_spread_type ON readings(spread_type);
```

### Performance Optimization

#### Index Strategy
1. Primary Access Patterns
   - User identification via telegram_id
   - Credit balance verification
   - Transaction history retrieval
   - Reading History access

2. Composite Indexes
```sql
-- User activity tracking
CREATE INDEX idx_users_activity ON users(is_active, last_active_at);

-- Reading History optimization
CREATE INDEX idx_readings_user_lookup ON readings(user_id, created_at DESC);

-- Transaction analysis
CREATE INDEX idx_transactions_analysis ON transactions(user_id, type, created_at);
```

3. Partial Indexes
```sql
-- Active subscription management
CREATE INDEX idx_active_subscriptions 
ON users(subscription_expires_at) 
WHERE subscription_tier != 'basic' AND is_active = true;

-- Pending transaction tracking
CREATE INDEX idx_pending_transactions 
ON transactions(created_at) 
WHERE status = 'pending';
```

### Caching Architecture

#### Cache Layers

1. Session Cache
   - TTL: 1 hour
   - Purpose: Session management
   - Implementation: Cache key-value storage

2. Credit Balance Cache
   - TTL: 5 minutes
   - Purpose: Fast balance checks
   - Implementation: Cache with atomic updates

3. Reading Cache
   - TTL: 1 hour
   - Purpose: Recent Reading storage
   - Implementation: Cache with JSON serialization

#### Cache Invalidation Strategies
1. Write-Through Caching
   - Database updates first
   - Cache updates after successful DB write
   - Automatic invalidation on errors

2. Cache Warming
   - Proactive Cache population
   - User-specific Cache warming
   - Background refresh processes

## Technical Specifications

### Performance Requirements
- Query Response Time: < 100ms
- Cache hit rate: > 90%
- Write latency: < 50ms
- Backup completion: < 30 minutes
- Recovery time: < 60 minutes

### Data Security
- Encryption at rest
- Secure backup storage
- Access control implementation
- Audit logging
- Data retention policies

### Backup Procedures
1. Database Backups
   - Full daily backups
   - Incremental hourly backups
   - Encrypted storage
   - Integrity verification

2. Recovery Procedures
   - Point-in-time recovery
   - Backup verification
   - Recovery testing
   - Documentation maintenance

## Related Documents
- [Glossary](glossary.md) - Standard terminology reference
- [Architecture](architecture-v2.md) - System architecture details
- [Security](security-v2.md) - Security measures
- [Monitoring](monitoring-v2.md) - System monitoring
- [Error Handling](error-handling-v2.md) - Error management

## Appendix

### Implementation Details
Key implementation files:
- `src/data/schema.py`
- `src/data/cache.py`
- `src/data/backup.py`
- `src/data/migration.py`

### Migration Scripts
Located in `src/data/migrations/`:
- Schema migration scripts
- Data migration utilities
- Rollback procedures

## Version History
| Version | Date | Author | Changes |
|---------|------|---------|---------|
| 1.0.0 | 2025-02-08 | Data Management Team | Initial version following new template |

## Notes
- Regular performance monitoring required
- Monthly backup verification scheduled
- Quarterly schema review planned
- Cache strategy review bi-weekly