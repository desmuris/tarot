# Data Management Documentation

## Overview
This document details the database schema, optimization strategies, caching mechanisms, and data handling procedures for the Tarot Reading Bot system.

## Database Schema

### Users Table
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

### Credits Table
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

### Transactions Table
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

### Readings Table
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

## Query Optimization

### Index Strategy
1. Primary Access Patterns
   - User lookup by telegram_id
   - Credit balance checks
   - Recent transactions
   - Reading history

2. Composite Indexes
   ```sql
   -- Fast user activity lookup
   CREATE INDEX idx_users_activity ON users(is_active, last_active_at);
   
   -- Efficient reading history retrieval
   CREATE INDEX idx_readings_user_lookup ON readings(user_id, created_at DESC);
   
   -- Transaction analysis
   CREATE INDEX idx_transactions_analysis ON transactions(user_id, type, created_at);
   ```

3. Partial Indexes
   ```sql
   -- Active subscriptions
   CREATE INDEX idx_active_subscriptions 
   ON users(subscription_expires_at) 
   WHERE subscription_tier != 'basic' AND is_active = true;
   
   -- Pending transactions
   CREATE INDEX idx_pending_transactions 
   ON transactions(created_at) 
   WHERE status = 'pending';
   ```

### Query Performance
1. Credit Balance Updates
   ```sql
   -- Atomic credit update with optimistic locking
   UPDATE credits 
   SET balance = balance + :amount,
       version = version + 1,
       last_updated_at = CURRENT_TIMESTAMP
   WHERE user_id = :user_id 
   AND version = :current_version
   RETURNING *;
   ```

2. Reading History
   ```sql
   -- Efficient reading history with pagination
   SELECT r.*, t.amount as credit_cost
   FROM readings r
   LEFT JOIN transactions t ON r.transaction_id = t.id
   WHERE r.user_id = :user_id
   ORDER BY r.created_at DESC
   LIMIT :limit OFFSET :offset;
   ```

## Caching Strategy

### Cache Layers
1. Session Cache
   ```python
   class SessionCache:
       """Handles user session caching"""
       
       CACHE_KEY_PREFIX = "session:"
       SESSION_TTL = 3600  # 1 hour
       
       async def get_session(self, session_id: str) -> Optional[Session]:
           cache_key = f"{self.CACHE_KEY_PREFIX}{session_id}"
           
           # Try cache first
           if cached := await self.redis.get(cache_key):
               return Session.from_json(cached)
           
           # Load from database
           if session := await self.load_session(session_id):
               await self.cache_session(session)
               return session
           
           return None
   ```

2. Credit Balance Cache
   ```python
   class CreditCache:
       """Manages credit balance caching"""
       
       CACHE_KEY_PREFIX = "credits:"
       BALANCE_TTL = 300  # 5 minutes
       
       async def get_balance(self, user_id: int) -> Optional[int]:
           cache_key = f"{self.CACHE_KEY_PREFIX}{user_id}"
           
           # Check cache
           if cached := await self.redis.get(cache_key):
               return int(cached)
           
           # Load from database
           if balance := await self.load_balance(user_id):
               await self.cache_balance(user_id, balance)
               return balance
           
           return None
   ```

3. Reading Cache
   ```python
   class ReadingCache:
       """Handles reading result caching"""
       
       CACHE_KEY_PREFIX = "reading:"
       READING_TTL = 3600  # 1 hour
       
       async def cache_reading(self, reading_id: str, reading: Reading):
           cache_key = f"{self.CACHE_KEY_PREFIX}{reading_id}"
           
           await self.redis.setex(
               cache_key,
               self.READING_TTL,
               reading.to_json()
           )
   ```

### Cache Invalidation
1. Write-Through Strategy
   ```python
   class WriteThoughCache:
       """Implements write-through caching"""
       
       async def update_balance(
           self,
           user_id: int,
           amount: int
       ) -> bool:
           try:
               # Update database first
               new_balance = await self.db.update_balance(
                   user_id,
                   amount
               )
               
               # Update cache
               await self.cache.set_balance(
                   user_id,
                   new_balance
               )
               
               return True
               
           except Exception as e:
               logger.error(f"Cache update failed: {str(e)}")
               await self.cache.invalidate_balance(user_id)
               return False
   ```

2. Cache Warming
   ```python
   class CacheWarmer:
       """Handles cache warming procedures"""
       
       async def warm_user_caches(self, user_id: int):
           """Warm all caches for a user"""
           await asyncio.gather(
               self.warm_session_cache(user_id),
               self.warm_balance_cache(user_id),
               self.warm_reading_cache(user_id)
           )
   ```

## Data Migration

### Migration Procedures
1. Schema Migrations
   ```python
   class SchemaMigration:
       """Handles database schema migrations"""
       
       async def migrate(self, version: str):
           async with self.db.transaction():
               # Apply migrations in order
               migrations = self.get_pending_migrations(version)
               for migration in migrations:
                   await self.apply_migration(migration)
                   await self.record_migration(migration)
   ```

2. Data Migrations
   ```python
   class DataMigration:
       """Handles data migrations"""
       
       async def migrate_data(
           self,
           batch_size: int = 1000
       ):
           """Migrate data in batches"""
           while True:
               batch = await self.get_next_batch(batch_size)
               if not batch:
                   break
                   
               await self.process_batch(batch)
               await self.mark_batch_complete(batch)
   ```

## Backup Procedures

### Backup Strategy
1. Database Backups
   ```python
   class DatabaseBackup:
       """Manages database backups"""
       
       async def create_backup(self):
           """Create full database backup"""
           timestamp = datetime.utcnow().isoformat()
           
           # Create backup
           backup_path = f"backups/db_{timestamp}.sql"
           await self.pg_dump(backup_path)
           
           # Encrypt backup
           encrypted_path = await self.encrypt_backup(backup_path)
           
           # Upload to storage
           await self.upload_backup(encrypted_path)
   ```

2. Recovery Procedures
   ```python
   class DatabaseRecovery:
       """Handles database recovery"""
       
       async def restore_database(
           self,
           backup_id: str
       ):
           """Restore database from backup"""
           # Download and decrypt backup
           backup_path = await self.get_backup(backup_id)
           
           # Verify backup integrity
           if not await self.verify_backup(backup_path):
               raise BackupVerificationError()
           
           # Restore database
           await self.pg_restore(backup_path)
   ```

## Related Documentation
- [Architecture](architecture.md) - System architecture
- [Security](security.md) - Data security measures
- [Monitoring](monitoring.md) - Data monitoring
- [Error Handling](error-handling.md) - Error management