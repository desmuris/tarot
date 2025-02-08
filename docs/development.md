# Development Guide

## Project Structure

```
tarot-bot/
├── src/
│   ├── bot/
│   │   ├── __init__.py
│   │   ├── commands.py      # Command handlers
│   │   ├── messages.py      # Message templates
│   │   └── session.py       # Session management
│   │
│   ├── tarot/
│   │   ├── __init__.py
│   │   ├── cards.py         # Card definitions
│   │   ├── spreads.py       # Spread layouts
│   │   └── readings.py      # Reading generation
│   │
│   ├── data/
│   │   ├── __init__.py
│   │   ├── models.py        # Database models
│   │   ├── queries.py       # Database queries
│   │   └── cache.py         # Caching logic
│   │
│   ├── payments/
│   │   ├── __init__.py
│   │   ├── pricing.py       # Pricing definitions
│   │   └── transactions.py  # Payment handling
│   │
│   └── utils/
│       ├── __init__.py
│       ├── config.py        # Configuration
│       ├── logging.py       # Logging setup
│       └── errors.py        # Error handling
│
├── tests/
│   ├── test_bot/           # Bot tests
│   ├── test_tarot/         # Tarot logic tests
│   ├── test_data/          # Data layer tests
│   └── test_payments/      # Payment tests
│
├── config/
│   ├── config.yaml         # Main configuration
│   └── logging.yaml        # Logging configuration
│
├── scripts/
│   ├── setup.py           # Setup script
│   └── migrations/        # Database migrations
│
├── requirements.txt       # Dependencies
├── README.md             # Documentation
└── .env.example          # Environment template
```

## Setup Instructions

### 1. Environment Setup
```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
.\venv\Scripts\activate
# On Unix:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Configuration
```bash
# Copy environment template
cp .env.example .env

# Edit environment variables
# Required variables:
# - TELEGRAM_BOT_TOKEN
# - OPENAI_API_KEY
# - DATABASE_URL
# - REDIS_URL
# - PAYMENT_PROVIDER_KEY
```

### 3. Database Setup
```bash
# Create database
python scripts/setup_db.py

# Run migrations
python scripts/migrate.py
```

### 4. Running the Application
```bash
# Development mode
python src/main.py --env development

# Production mode
python src/main.py --env production
```

## Development Workflow

### 1. Code Style
- Follow PEP 8 guidelines
- Use type hints
- Document functions and classes
- Keep functions focused and small
- Use meaningful variable names

### 2. Git Workflow
```bash
# Create feature branch
git checkout -b feature/feature-name

# Make changes and commit
git add .
git commit -m "Description of changes"

# Push changes
git push origin feature/feature-name

# Create pull request
# - Include description of changes
# - Reference related issues
# - Add tests
```

### 3. Testing
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_bot/test_commands.py

# Run with coverage
pytest --cov=src tests/
```

### 4. Code Review Guidelines
- Check code style
- Verify test coverage
- Review documentation
- Check error handling
- Validate security measures

## Module Development

### 1. Adding New Commands
```python
from bot.commands import Command

class NewCommand(Command):
    """Implement new bot command"""
    
    async def handle(self, update: Update, context: Context):
        """Handle command execution"""
        # Implementation
```

### 2. Creating New Spreads
```python
from tarot.spreads import Spread

class NewSpread(Spread):
    """Define new tarot spread"""
    
    def get_positions(self) -> List[str]:
        """Define card positions"""
        return [
            "Position 1",
            "Position 2",
            # Add positions
        ]
```

### 3. Adding Payment Methods
```python
from payments.providers import PaymentProvider

class NewPaymentProvider(PaymentProvider):
    """Implement new payment provider"""
    
    async def process_payment(
        self,
        amount: Decimal,
        currency: str,
        metadata: Dict
    ) -> PaymentResult:
        """Process payment"""
        # Implementation
```

## Error Handling

### 1. Exception Hierarchy
```python
class TarotBotError(Exception):
    """Base exception for all bot errors"""
    pass

class PaymentError(TarotBotError):
    """Payment processing errors"""
    pass

class ReadingError(TarotBotError):
    """Reading generation errors"""
    pass
```

### 2. Error Logging
```python
import logging

logger = logging.getLogger(__name__)

try:
    # Operation that might fail
    result = await process_operation()
except Exception as e:
    logger.error(
        "Operation failed",
        exc_info=e,
        extra={
            "operation": "process_operation",
            "parameters": parameters
        }
    )
```

## Performance Optimization

### 1. Caching Strategy
- Use Redis for session data
- Cache frequent database queries
- Cache API responses
- Implement cache invalidation
- Monitor cache hit rates

### 2. Database Optimization
- Index frequently queried fields
- Use database connection pooling
- Implement query optimization
- Monitor query performance
- Use database partitioning

### 3. API Optimization
- Implement rate limiting
- Use connection pooling
- Cache API responses
- Monitor API performance
- Handle API failures gracefully

## Monitoring and Debugging

### 1. Logging
```python
# Configure logging
logging.config.fileConfig('config/logging.yaml')
logger = logging.getLogger(__name__)

# Log with context
logger.info(
    "Operation completed",
    extra={
        "user_id": user_id,
        "operation": operation_name,
        "duration": duration
    }
)
```

### 2. Metrics
```python
# Record metrics
metrics.increment('readings_completed')
metrics.timing('reading_duration', duration)
metrics.gauge('active_users', active_users_count)
```

## Related Documentation
- [System Architecture](architecture.md) - System design
- [Deployment Guide](deployment.md) - Deployment process
- [Data Management](data-management.md) - Data operations