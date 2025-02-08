# Data Management

## Database Schema

### Users Table
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    telegram_id BIGINT UNIQUE NOT NULL,
    credits INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_active_at TIMESTAMP,
    status VARCHAR(20) DEFAULT 'active',
    preferences JSONB DEFAULT '{}',
    subscription_tier VARCHAR(20),
    subscription_expires_at TIMESTAMP,
    CONSTRAINT valid_status CHECK (status IN ('active', 'suspended', 'deleted'))
);
```

### Readings Table
```sql
CREATE TABLE readings (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    spread_type VARCHAR(50) NOT NULL,
    question TEXT,
    cards JSONB NOT NULL,
    interpretation TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP,
    duration_seconds INTEGER,
    status VARCHAR(20) DEFAULT 'in_progress',
    questionnaire_responses JSONB,
    error_message TEXT,
    CONSTRAINT valid_status CHECK (status IN ('in_progress', 'completed', 'error', 'interrupted'))
);
```

### Transactions Table
```sql
CREATE TABLE transactions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    amount INTEGER NOT NULL,
    type VARCHAR(20) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',
    payment_provider VARCHAR(50),
    payment_reference VARCHAR(100),
    refund_reference VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    error_message TEXT,
    metadata JSONB,
    CONSTRAINT valid_type CHECK (type IN ('purchase', 'reading_cost', 'refund', 'subscription')),
    CONSTRAINT valid_status CHECK (status IN ('pending', 'completed', 'failed', 'refunded'))
);
```

### User Sessions Table
```sql
CREATE TABLE user_sessions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    session_type VARCHAR(50),
    started_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_activity_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP,
    status VARCHAR(20) DEFAULT 'active',
    data JSONB,
    CONSTRAINT valid_status CHECK (status IN ('active', 'completed', 'expired', 'error'))
);
```

### Rate Limiting Table
```sql
CREATE TABLE rate_limits (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    endpoint VARCHAR(100) NOT NULL,
    request_count INTEGER DEFAULT 1,
    window_start TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (user_id, endpoint, window_start)
);
```

## Caching Strategy

### Cache Configuration
```python
class CacheManager:
    """Implements comprehensive caching strategy"""
    
    def __init__(
        self,
        redis_client: Redis,
        max_size_mb: int = 1000,
        default_ttl: int = 3600
    ):
        self.redis = redis_client
        self.max_size = max_size_mb * 1024 * 1024  # Convert to bytes
        self.default_ttl = default_ttl
        
        # Cache configuration
        self.cache_configs = {
            "user_data": {"ttl": 3600, "max_size": 100000},
            "reading_history": {"ttl": 7200, "max_size": 500000},
            "card_data": {"ttl": 86400, "max_size": 1000000}
        }
```

### Cache Operations
```python
    async def get_or_set(
        self,
        key: str,
        getter: Callable,
        ttl: int = None,
        cache_type: str = "default"
    ) -> Any:
        """
        Enhanced get-or-set pattern:
        1. Check cache size limits
        2. Try to get from cache
        3. Handle cache misses
        4. Implement cache warming
        5. Monitor cache health
        """
        config = self.cache_configs.get(cache_type, {"ttl": self.default_ttl})
        
        # Check cache size
        await self._ensure_cache_size(cache_type)
        
        # Try cache first
        cached = await self.redis.get(key)
        if cached:
            await self._update_cache_metrics(cache_type, "hit")
            return json.loads(cached)
        
        # Cache miss - get fresh data
        try:
            value = await getter()
            
            # Validate value size
            value_size = len(json.dumps(value).encode())
            if value_size > config["max_size"]:
                logger.warning(f"Value too large for cache: {value_size} bytes")
                return value
            
            # Cache with TTL
            await self.redis.set(
                key,
                json.dumps(value),
                ex=ttl or config["ttl"]
            )
            
            await self._update_cache_metrics(cache_type, "miss")
            return value
            
        except Exception as e:
            logger.error(f"Cache getter failed: {str(e)}")
            raise
```

### Cache Management
```python
    async def invalidate(
        self,
        pattern: str,
        cache_type: str = "default"
    ) -> int:
        """Invalidate cache entries matching pattern"""
        keys = await self.redis.keys(pattern)
        if keys:
            await self.redis.delete(*keys)
        return len(keys)
    
    async def _ensure_cache_size(self, cache_type: str):
        """Ensure cache size is within limits"""
        current_size = await self.redis.memory_usage()
        if current_size > self.max_size:
            await self._evict_entries(cache_type)
    
    async def _evict_entries(self, cache_type: str):
        """Implement LRU eviction for cache type"""
        config = self.cache_configs.get(cache_type, {})
        pattern = f"{cache_type}:*"
        
        # Get all matching keys with their idle time
        keys = []
        async for key in self.redis.scan_iter(match=pattern):
            idle_time = await self.redis.object("idletime", key)
            keys.append((key, idle_time))
        
        # Sort by idle time and remove oldest entries
        keys.sort(key=lambda x: x[1], reverse=True)
        for key, _ in keys[:100]:  # Remove 100 oldest entries
            await self.redis.delete(key)
```

## Data Flow

### Reading Request Flow
1. User initiates reading request
2. System checks cache for user data
3. Validates user credits and status
4. Processes reading generation
5. Stores reading result
6. Updates user credits
7. Caches relevant data

### Payment Flow
1. User initiates payment
2. System validates payment request
3. Processes payment through provider
4. Updates user credits
5. Records transaction
6. Updates cache

## Related Documentation
- [System Architecture](architecture.md) - Overall system design
- [Payment System](payment-system.md) - Payment processing details
- [Development Guide](development.md) - Implementation guidelines