# System Architecture

## Overview

The system employs a modular monolith architecture, organizing functionality into clear boundaries while maintaining simplicity and ease of maintenance.

## Core Modules

### 1. Bot Interface Module
- Handles Telegram API integration
- Manages user commands and responses
- Maintains user sessions
- Routes requests to appropriate modules
- Handles rate limiting and throttling

### 2. Tarot Core Module
- Manages card selection and spreads
- Coordinates reading generation
- Handles OpenAI API integration
- Processes interpretation results
- Manages deck preferences

### 3. Data Management Module
- Handles database operations
- Manages caching strategy
- Stores user data and readings
- Maintains credit balances
- Handles data retention policies

### 4. Payment Module
- Processes payments securely
- Manages credit system
- Handles subscription tiers
- Records detailed transactions
- Manages refunds and disputes

## Module Communication

### Communication Flow
```python
class ModuleManager:
    """Coordinates communication between modules"""
    
    def __init__(self):
        self.bot_interface = BotInterface()
        self.tarot_core = TarotCore()
        self.data_manager = DataManager()
        self.payment_handler = PaymentHandler()
        self.error_handler = ErrorHandler()

    async def handle_reading_request(
        self,
        user_id: int,
        spread_type: str,
        question: str
    ) -> ReadingResult:
        """
        Coordinates a reading request across modules:
        1. Validate request and user status
        2. Verify and reserve credits
        3. Generate reading
        4. Store result
        5. Update balance
        6. Handle any failures
        """
        async with self.data_manager.transaction() as transaction:
            try:
                # Validate user status
                user = await self.data_manager.get_user(user_id)
                if not user.is_active:
                    return ReadingResult(error="Account inactive")

                # Verify and reserve credits
                if not await self.payment_handler.verify_credits(user_id):
                    return ReadingResult(error="Insufficient credits")
                
                await self.payment_handler.reserve_credits(user_id)

                # Generate reading
                reading = await self.tarot_core.generate_reading(
                    spread_type,
                    question,
                    user.preferences
                )
                
                # Store result
                await self.data_manager.save_reading(user_id, reading)
                
                # Update credits
                await self.payment_handler.deduct_credits(user_id)
                
                await transaction.commit()
                return ReadingResult(reading=reading)

            except Exception as e:
                await transaction.rollback()
                return await self.error_handler.handle_error(e)
```

## Error Handling

### Error Management System
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
- Basic HTTP/HTTPS routing
- SSL termination
- Health checks

### 2. Application Server
- Python FastAPI application
- Modular monolith structure
- Environment-based configuration

### 3. PostgreSQL
- User data
- Reading history
- Transaction records

### 4. Redis
- Session caching
- Rate limiting
- Basic result caching

### 5. Monitoring
- Basic health metrics
- Error logging
- Performance monitoring

## Related Documentation
- [Data Management](data-management.md) - Detailed database and caching implementation
- [Deployment](deployment.md) - Deployment configuration and process
- [Development Guide](development.md) - Setup and development workflow