# Error Handling Documentation

## Overview
This document outlines the comprehensive error handling strategies, recovery procedures, and best practices implemented across the Tarot Reading Bot system. It provides a consistent approach to error management across all system components.

## Error Categories

### System Errors
1. Database Errors
   - Connection failures
   - Query timeouts
   - Constraint violations
   - Deadlock situations
   - Replication issues
   - Connection pool exhaustion

2. Cache Errors
   - Redis connection issues
   - Cache inconsistency
   - Memory limits
   - Eviction problems
   - Key conflicts
   - Serialization errors

3. API Errors
   - OpenAI API failures
   - Telegram API issues
   - Payment gateway errors
   - Rate limit exceeded
   - Network timeouts
   - Invalid responses

### Business Logic Errors
1. Credit System
   ```python
   class CreditSystemErrors:
       """Credit system specific error handling"""
       
       ERROR_TYPES = {
           "insufficient_credits": {
               "code": "CREDIT001",
               "message": "Insufficient credits for operation",
               "retryable": False,
               "user_message": "Not enough credits available"
           },
           "failed_transaction": {
               "code": "CREDIT002",
               "message": "Credit transaction failed",
               "retryable": True,
               "user_message": "Unable to process credit transaction"
           },
           "reservation_timeout": {
               "code": "CREDIT003",
               "message": "Credit reservation timed out",
               "retryable": True,
               "user_message": "Transaction timed out"
           },
           "balance_inconsistency": {
               "code": "CREDIT004",
               "message": "Credit balance inconsistency detected",
               "retryable": False,
               "user_message": "System temporarily unavailable"
           }
       }
   ```

2. Payment System
   ```python
   class PaymentSystemErrors:
       """Payment system specific error handling"""
       
       ERROR_TYPES = {
           "payment_failed": {
               "code": "PAY001",
               "message": "Payment processing failed",
               "retryable": True,
               "user_message": "Payment could not be processed"
           },
           "invalid_payment_method": {
               "code": "PAY002",
               "message": "Invalid payment method",
               "retryable": False,
               "user_message": "Payment method not accepted"
           },
           "gateway_error": {
               "code": "PAY003",
               "message": "Payment gateway error",
               "retryable": True,
               "user_message": "Payment service temporarily unavailable"
           },
           "fraud_detection": {
               "code": "PAY004",
               "message": "Transaction flagged for fraud",
               "retryable": False,
               "user_message": "Transaction declined for security"
           }
       }
   ```

3. Reading System
   ```python
   class ReadingSystemErrors:
       """Reading system specific error handling"""
       
       ERROR_TYPES = {
           "generation_failed": {
               "code": "READ001",
               "message": "Reading generation failed",
               "retryable": True,
               "user_message": "Unable to generate reading"
           },
           "invalid_spread": {
               "code": "READ002",
               "message": "Invalid spread type",
               "retryable": False,
               "user_message": "Selected spread is not available"
           },
           "api_error": {
               "code": "READ003",
               "message": "OpenAI API error",
               "retryable": True,
               "user_message": "Service temporarily unavailable"
           },
           "validation_error": {
               "code": "READ004",
               "message": "Reading validation failed",
               "retryable": False,
               "user_message": "Invalid reading parameters"
           }
       }
   ```

## Error Recovery Procedures

### Transaction Recovery
```python
class TransactionRecoveryHandler:
    """Handles transaction recovery across systems"""
    
    async def handle_failed_transaction(
        self,
        transaction_id: str,
        error_type: str,
        context: dict
    ) -> RecoveryResult:
        """
        Comprehensive transaction recovery handler
        
        Args:
            transaction_id: Unique transaction identifier
            error_type: Type of error that occurred
            context: Additional context about the transaction
            
        Returns:
            RecoveryResult indicating success/failure of recovery
        """
        try:
            # 1. Log recovery attempt
            logger.info(f"Starting recovery for transaction {transaction_id}")
            
            # 2. Determine recovery strategy
            strategy = await self.get_recovery_strategy(error_type)
            
            # 3. Execute recovery steps
            async with transaction_manager.transaction() as txn:
                # Release any holds
                await self.release_resources(transaction_id)
                
                # Restore system state
                await self.restore_state(transaction_id)
                
                # Update transaction status
                await self.update_transaction_status(
                    transaction_id,
                    status="recovered",
                    details=context
                )
                
                # Notify relevant systems
                await self.notify_systems(transaction_id, strategy)
            
            # 4. Verify recovery
            recovery_status = await self.verify_recovery(transaction_id)
            
            return RecoveryResult(
                success=recovery_status.success,
                transaction_id=transaction_id,
                recovery_id=recovery_status.id
            )
            
        except Exception as e:
            logger.error(f"Recovery failed for transaction {transaction_id}: {str(e)}")
            return RecoveryResult(
                success=False,
                error=str(e),
                requires_manual=True
            )
```

### System Recovery
```python
class SystemRecoveryHandler:
    """Handles system-level recovery procedures"""
    
    async def handle_system_failure(
        self,
        system_name: str,
        error_context: dict
    ) -> SystemRecoveryResult:
        """
        System-wide recovery handler
        
        Args:
            system_name: Name of the failed system
            error_context: Context about the failure
            
        Returns:
            SystemRecoveryResult indicating recovery status
        """
        try:
            # 1. Assess system state
            current_state = await self.assess_system_state(system_name)
            
            # 2. Plan recovery steps
            recovery_plan = self.create_recovery_plan(
                system_name,
                current_state,
                error_context
            )
            
            # 3. Execute recovery
            for step in recovery_plan.steps:
                await self.execute_recovery_step(step)
                await self.verify_step_completion(step)
            
            # 4. Verify system health
            health_status = await self.verify_system_health(system_name)
            
            return SystemRecoveryResult(
                success=health_status.healthy,
                system=system_name,
                recovery_id=recovery_plan.id
            )
            
        except Exception as e:
            logger.error(f"System recovery failed for {system_name}: {str(e)}")
            return SystemRecoveryResult(
                success=False,
                error=str(e),
                requires_escalation=True
            )
```

## Error Prevention

### Circuit Breakers
```python
class CircuitBreaker:
    """Implements circuit breaker pattern"""
    
    def __init__(
        self,
        failure_threshold: int = 5,
        reset_timeout: int = 60
    ):
        self.failure_count = 0
        self.last_failure = None
        self.state = "closed"
        self.failure_threshold = failure_threshold
        self.reset_timeout = reset_timeout
    
    async def execute(self, func: Callable, *args, **kwargs):
        """Execute function with circuit breaker protection"""
        if self.state == "open":
            if self._should_retry():
                self.state = "half-open"
            else:
                raise CircuitBreakerError("Circuit breaker is open")
        
        try:
            result = await func(*args, **kwargs)
            if self.state == "half-open":
                self.state = "closed"
                self.failure_count = 0
            return result
            
        except Exception as e:
            self._handle_failure(e)
            raise
```

### Rate Limiting
```python
class RateLimiter:
    """Implements rate limiting across services"""
    
    async def check_rate_limit(
        self,
        user_id: int,
        action: str
    ) -> bool:
        """
        Check if action is within rate limits
        
        Args:
            user_id: User identifier
            action: Action being rate limited
            
        Returns:
            Boolean indicating if action is allowed
        """
        # Get user's subscription tier
        tier = await subscription_service.get_user_tier(user_id)
        
        # Get rate limit for tier and action
        limit = RATE_LIMITS[tier][action]
        
        # Check current usage
        current = await redis.get(f"rate:{user_id}:{action}")
        if current and int(current) >= limit:
            raise RateLimitExceeded(
                f"Rate limit exceeded for {action}"
            )
        
        # Update usage
        await redis.incr(f"rate:{user_id}:{action}")
        await redis.expire(
            f"rate:{user_id}:{action}",
            RATE_LIMIT_WINDOW
        )
        
        return True
```

## Monitoring & Alerting

### Error Tracking
1. Error Metrics
   - Error rate by category
   - Recovery success rate
   - System degradation indicators
   - User impact metrics
   - Response time variations

2. Alert Thresholds
   - Critical error count > 5/minute
   - Recovery failure rate > 1%
   - System error spike > 200%
   - API error rate > 5%
   - Transaction failure rate > 1%

### Error Reporting
```python
class ErrorReport:
    """Standard error report format"""
    
    def __init__(
        self,
        error_type: str,
        message: str,
        context: dict
    ):
        self.timestamp = datetime.utcnow().isoformat()
        self.error_type = error_type
        self.message = message
        self.context = context
        self.trace_id = str(uuid.uuid4())
    
    def to_dict(self) -> dict:
        """Convert error report to dictionary"""
        return {
            "timestamp": self.timestamp,
            "error_type": self.error_type,
            "message": self.message,
            "trace_id": self.trace_id,
            "context": self.context
        }
```

## Related Documentation
- [System Architecture](architecture.md) - System design and error handling
- [Monitoring](monitoring.md) - System monitoring and alerts
- [Credit System](credit-system.md) - Credit system error handling
- [Payment System](payment-system.md) - Payment system error handling
- [Security](security.md) - Security-related error handling