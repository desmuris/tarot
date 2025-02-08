# System Architecture

## Overview
The Tarot Reading Bot system employs a modular monolith architecture, organizing functionality into clear boundaries while maintaining simplicity and ease of maintenance. The system is designed with security, scalability, and reliability as core principles.

## Core Modules

### 1. Bot Interface Module
```python
class BotInterface:
    """Handles Telegram bot interactions and request routing"""
    
    def __init__(self):
        self.auth_handler = APIAuthenticator()
        self.rate_limiter = RateLimiter()
        self.session_manager = SessionManager()
    
    async def handle_request(
        self,
        update: Update,
        context: Context
    ) -> Response:
        """
        Process incoming bot requests
        
        Args:
            update: Telegram update object
            context: Request context
            
        Returns:
            Response object
        """
        try:
            # Authenticate request
            auth_result = await self.auth_handler.authenticate_request(
                update.effective_user.id
            )
            if not auth_result.success:
                return ErrorResponse("Authentication failed")
            
            # Check rate limits
            await self.rate_limiter.check_rate_limit(
                auth_result.user_id,
                "bot_request"
            )
            
            # Route request
            handler = self.get_request_handler(update)
            response = await handler.handle(update, context)
            
            # Log successful request
            await self.log_request(
                auth_result.user_id,
                "success",
                update.effective_message.text
            )
            
            return response
            
        except Exception as e:
            await self.handle_error(e, update)
            return ErrorResponse(str(e))
```

### 2. Tarot Core Module
```python
class TarotCore:
    """Manages reading generation and interpretation"""
    
    def __init__(self):
        self.openai_client = OpenAIClient()
        self.spread_manager = SpreadManager()
        self.interpretation_engine = InterpretationEngine()
    
    async def generate_reading(
        self,
        user_id: int,
        spread_type: str,
        question: str
    ) -> Reading:
        """
        Generate a tarot reading
        
        Args:
            user_id: User identifier
            spread_type: Type of spread requested
            question: User's question
            
        Returns:
            Reading object containing the interpretation
        """
        try:
            # Validate spread type
            spread = await self.spread_manager.get_spread(spread_type)
            if not spread:
                raise InvalidSpreadError()
            
            # Select cards
            cards = await self.spread_manager.select_cards(spread)
            
            # Generate interpretation
            interpretation = await self.interpretation_engine.interpret(
                cards=cards,
                spread=spread,
                question=question
            )
            
            # Create reading object
            reading = Reading(
                user_id=user_id,
                spread_type=spread_type,
                cards=cards,
                interpretation=interpretation,
                timestamp=datetime.utcnow()
            )
            
            return reading
            
        except Exception as e:
            logger.error(f"Reading generation failed: {str(e)}")
            raise ReadingGenerationError(str(e))
```

### 3. Data Management Module
```python
class DataManager:
    """Handles database operations and caching"""
    
    def __init__(self):
        self.db = PostgreSQLConnection()
        self.cache = RedisCache()
        self.encryption = DataEncryption()
    
    async def store_reading(
        self,
        reading: Reading
    ) -> bool:
        """
        Store reading with proper encryption
        
        Args:
            reading: Reading object to store
            
        Returns:
            Boolean indicating success
        """
        try:
            # Encrypt sensitive data
            encrypted_data = await self.encryption.encrypt_data(
                reading.to_dict(),
                "reading"
            )
            
            # Store in database
            await self.db.store_reading(
                user_id=reading.user_id,
                encrypted_data=encrypted_data
            )
            
            # Update cache
            await self.cache.set_reading(
                reading.id,
                encrypted_data,
                ttl=3600  # 1 hour
            )
            
            return True
            
        except Exception as e:
            logger.error(f"Reading storage failed: {str(e)}")
            raise DataStorageError(str(e))
```

### 4. Payment Module
```python
class PaymentHandler:
    """Manages payment processing and credit system"""
    
    def __init__(self):
        self.payment_gateway = PaymentGateway()
        self.credit_manager = CreditManager()
        self.transaction_manager = TransactionManager()
    
    async def process_credit_purchase(
        self,
        user_id: int,
        package: str,
        payment_method: str
    ) -> PaymentResult:
        """
        Process credit package purchase
        
        Args:
            user_id: User identifier
            package: Credit package identifier
            payment_method: Payment method to use
            
        Returns:
            PaymentResult object
        """
        async with self.transaction_manager.transaction() as txn:
            try:
                # Validate package
                package_info = await self.credit_manager.get_package(package)
                if not package_info:
                    raise InvalidPackageError()
                
                # Process payment
                payment_result = await self.payment_gateway.process_payment(
                    amount=package_info.price,
                    currency="USD",
                    payment_method=payment_method
                )
                
                if payment_result.success:
                    # Add credits
                    await self.credit_manager.add_credits(
                        user_id=user_id,
                        amount=package_info.credits
                    )
                    
                    # Record transaction
                    await self.transaction_manager.record_transaction(
                        user_id=user_id,
                        type="credit_purchase",
                        amount=package_info.price,
                        credits=package_info.credits
                    )
                    
                    return PaymentResult(
                        success=True,
                        transaction_id=payment_result.id
                    )
                
                raise PaymentProcessingError(payment_result.error)
                
            except Exception as e:
                await txn.rollback()
                logger.error(f"Payment processing failed: {str(e)}")
                raise PaymentError(str(e))
```

## Error Handling
```python
class ErrorHandler:
    """Implements comprehensive error handling"""
    
    ERROR_TYPES = {
        "validation": {
            "retryable": False,
            "user_message": "Invalid input provided",
            "log_level": "warning"
        },
        "payment": {
            "retryable": True,
            "user_message": "Payment processing error",
            "log_level": "error"
        },
        "api": {
            "retryable": True,
            "user_message": "Service temporarily unavailable",
            "log_level": "error"
        },
        "database": {
            "retryable": True,
            "user_message": "System temporarily unavailable",
            "log_level": "error"
        }
    }
    
    async def handle_error(
        self,
        error: Exception,
        context: dict = None
    ) -> ErrorResult:
        """
        Handle system errors with proper logging and recovery
        
        Args:
            error: Exception that occurred
            context: Error context
            
        Returns:
            ErrorResult object
        """
        try:
            # Classify error
            error_type = self.classify_error(error)
            error_config = self.ERROR_TYPES[error_type]
            
            # Log error
            logger.log(
                error_config["log_level"],
                f"Error occurred: {str(error)}",
                extra=context
            )
            
            # Attempt recovery if retryable
            if error_config["retryable"]:
                recovery_result = await self.attempt_recovery(
                    error_type,
                    error,
                    context
                )
                if recovery_result.success:
                    return ErrorResult(
                        handled=True,
                        recovered=True
                    )
            
            # Return error result
            return ErrorResult(
                handled=True,
                recovered=False,
                user_message=error_config["user_message"]
            )
            
        except Exception as e:
            logger.error(f"Error handling failed: {str(e)}")
            return ErrorResult(
                handled=False,
                error="System error"
            )
```

## System Architecture Diagram
```
                                    [HTTPS/SSL]
                                         │
                                         ▼
┌──────────────────────────────────[Load Balancer]──────────────────────────────┐
│                                        │                                       │
│                                        ▼                                       │
│                              [Application Server]                              │
│                                        │                                       │
│                    ┌──────────────────┴───────────────────┐                   │
│                    ▼                   ▼                   ▼                   │
│              [PostgreSQL]           [Redis]           [OpenAI API]            │
│                    │                   │                   │                   │
│                    └──────────────────┬───────────────────┘                   │
│                                       │                                        │
└───────────────────────────────[Monitoring]────────────────────────────────────┘
```

## Component Details

### 1. Load Balancer
- HTTPS/SSL termination
- Request routing
- Health checking
- Rate limiting
- DDoS protection

### 2. Application Server
- Python FastAPI application
- Modular monolith structure
- Environment-based configuration
- Comprehensive error handling
- Security middleware

### 3. PostgreSQL
- User data storage
- Reading history
- Transaction records
- Encrypted sensitive data
- Audit logging

### 4. Redis
- Session management
- Rate limiting
- Result caching
- Job queues
- Real-time metrics

### 5. Monitoring
- System health metrics
- Error tracking
- Performance monitoring
- Security monitoring
- Business metrics

## Related Documentation
- [Data Management](data-management.md) - Database and caching details
- [Security](security.md) - Security measures
- [Error Handling](error-handling.md) - Error management
- [Monitoring](monitoring.md) - System monitoring
- [Credit System](credit-system.md) - Credit management
- [Payment System](payment-system.md) - Payment processing