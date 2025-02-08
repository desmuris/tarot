# Development Guide

## Development with Roo Cline

### 1. Development Session Structure
```
Session Flow:
1. Context Establishment
   - Share relevant file contents
   - Reference previous work
   - Outline current objectives

2. Task Definition
   - Clear, specific requirements
   - Explicit file paths and changes needed
   - Required dependencies and context

3. Implementation Review
   - Code review and validation
   - Functionality testing
   - Performance assessment
```

### 2. Milestone-Based Development

#### Core Feature Development Phases

1. Reading System Implementation
```
Phase 1: Card and Spread Foundation
- Implement base card definitions
- Develop spread layouts
- Create card selection logic
- Test basic card operations

Phase 2: Interpretation Engine
- Develop position analysis
- Implement card combination logic
- Create direction interpretation
- Test interpretation accuracy

Phase 3: Questionnaire Integration
- Implement questionnaire system
- Develop context integration
- Create response handling
- Test questionnaire flow
```

2. User Interaction Flow
```
Phase 1: Basic Interaction
- Implement command handling
- Develop message templates
- Create session management
- Test user interactions

Phase 2: Reading Experience
- Implement reading flow
- Develop result formatting
- Create guidance generation
- Test user experience

Phase 3: Advanced Features
- Implement spread selection
- Develop deck management
- Create history tracking
- Test feature integration
```

#### Development Guidelines

1. Feature Implementation Structure
```
a. Planning
   - Feature scope definition
   - Technical requirements
   - Integration points
   - Success criteria

b. Development Phases
   - Component implementation
   - Integration development
   - Testing strategy
   - Documentation updates

c. Review Points
   - Code review
   - Testing validation
   - Performance verification
   - Security assessment

d. Completion Criteria
   - Functionality verification
   - Documentation completion
   - Test coverage validation
   - Performance benchmarks
```

2. Quality Assurance Focus
```
Reading System Validation:
- Card selection accuracy
- Spread layout correctness
- Interpretation coherence
- Questionnaire effectiveness

User Experience Verification:
- Command response accuracy
- Message clarity
- Session consistency
- Error handling
```

#### Development Workflow
1. Initial Planning
   - Review requirements
   - Define milestones
   - Identify dependencies
   - Set success criteria

2. Implementation Phase
   - Step-by-step development
   - Regular checkpoints
   - Progress documentation
   - Issue tracking

3. Review and Validation
   - Code review
   - Testing
   - Performance analysis
   - Security verification

### 3. Progress Tracking
- Document implementation decisions
- Track completed milestones
- Record technical challenges
- Monitor development progress

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

## Development Standards

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

### 3. Testing Strategy

#### Reading System Testing
```python
# test_tarot/test_readings.py
class TestReadingSystem:
    """Test suite for tarot reading functionality"""
    
    async def test_card_selection(self):
        """Verify card selection logic"""
        spread = SpreadTypes.SPREADS["celtic_cross"]
        cards = await card_selector.select_cards(spread)
        assert len(cards) == spread["positions"]
        assert len(set(cards)) == len(cards)  # No duplicates
    
    async def test_interpretation_engine(self):
        """Test interpretation generation"""
        reading = await reading_interpreter.interpret_reading(
            cards=test_cards,
            positions=test_positions,
            spread_type="celtic_cross",
            question="Test question"
        )
        assert reading.theme is not None
        assert reading.card_analysis is not None
        assert len(reading.practical_advice) > 0
    
    async def test_questionnaire_integration(self):
        """Verify questionnaire processing"""
        responses = {
            "focus": "Career growth",
            "challenges": "Time management",
            "goals": "Leadership development"
        }
        reading = await generate_reading_with_questionnaire(
            spread_type="career",
            responses=responses
        )
        assert "career" in reading.theme.lower()
        assert "leadership" in reading.practical_advice.lower()
```

#### Test Execution
```bash
# Run all tests
pytest

# Run reading system tests
pytest tests/test_tarot/test_readings.py

# Run specific test class
pytest tests/test_tarot/test_readings.py::TestReadingSystem

# Run with coverage
pytest --cov=src tests/
```

#### Test Categories
1. Unit Tests
   - Card selection logic
   - Spread layout validation
   - Interpretation components
   - Questionnaire processing

2. Integration Tests
   - Reading generation flow
   - User session handling
   - Payment integration
   - Database operations

3. System Tests
   - End-to-end reading flow
   - Error recovery scenarios
   - Performance benchmarks
   - Security validations

### 4. Code Review Guidelines
- Check code style
- Verify test coverage
- Review documentation
- Check error handling
- Validate security measures

## Error Handling

### 1. Reading System Error Handling

#### Exception Hierarchy
```python
class TarotBotError(Exception):
    """Base exception for all bot errors"""
    pass

class ReadingError(TarotBotError):
    """Reading generation errors"""
    def __init__(self, message: str, context: dict):
        self.context = context
        super().__init__(message)

class InterpretationError(ReadingError):
    """Card interpretation errors"""
    pass

class SpreadError(ReadingError):
    """Spread layout and card selection errors"""
    pass

class QuestionnaireError(ReadingError):
    """Questionnaire processing errors"""
    pass

class PaymentError(TarotBotError):
    """Payment processing errors"""
    pass
```

#### Error Recovery Strategies
```python
class ReadingErrorHandler:
    """Specialized handler for reading-related errors"""
    
    RECOVERY_STRATEGIES = {
        "interpretation": {
            "retry_count": 3,
            "fallback": "basic_interpretation",
            "user_message": "Providing essential insights"
        },
        "spread_layout": {
            "retry_count": 2,
            "fallback": "simple_spread",
            "user_message": "Using simplified spread layout"
        },
        "questionnaire": {
            "retry_count": 2,
            "fallback": "skip_questionnaire",
            "user_message": "Proceeding with basic reading"
        }
    }

    async def handle_error(
        self,
        error: ReadingError,
        context: dict
    ) -> dict:
        """
        Handle reading system errors with appropriate recovery
        
        Args:
            error: The encountered error
            context: Error context including spread type, cards, etc.
            
        Returns:
            Dict containing recovery result and next steps
        """
        strategy = self.RECOVERY_STRATEGIES.get(
            error.context.get("error_type"),
            self.RECOVERY_STRATEGIES["interpretation"]
        )
        
        return await self._apply_recovery_strategy(
            error,
            strategy,
            context
        )
```

### 2. Comprehensive Error Logging

#### Reading System Logging
```python
class ReadingLogger:
    """Specialized logging for reading operations"""
    
    def log_reading_error(
        self,
        error: ReadingError,
        context: dict
    ):
        """Log reading-related errors with full context"""
        logger.error(
            "Reading operation failed",
            extra={
                "error_type": error.__class__.__name__,
                "spread_type": context.get("spread_type"),
                "cards_drawn": context.get("cards"),
                "questionnaire_type": context.get("questionnaire_type"),
                "user_question": context.get("question"),
                "recovery_attempted": context.get("recovery_attempted"),
                "error_context": error.context
            }
        )

    def log_interpretation_result(
        self,
        success: bool,
        context: dict
    ):
        """Log interpretation results for quality monitoring"""
        logger.info(
            "Interpretation completed",
            extra={
                "success": success,
                "spread_type": context.get("spread_type"),
                "cards_count": len(context.get("cards", [])),
                "interpretation_quality": context.get("quality_score"),
                "processing_time": context.get("processing_time"),
                "questionnaire_used": context.get("questionnaire_used", False)
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

### 2. Reading System Metrics

#### Core Metrics
```python
class ReadingMetrics:
    """Track reading system performance and quality"""
    
    def record_reading_metrics(self, reading: Reading):
        """Record comprehensive reading metrics"""
        # Reading completion metrics
        metrics.increment(f'readings.completed.{reading.spread_type}')
        metrics.timing('reading.generation_time', reading.processing_time)
        metrics.gauge('reading.quality_score', reading.quality_score)
        
        # Spread and deck usage
        metrics.increment(f'spreads.used.{reading.spread_type}')
        metrics.increment(f'decks.used.{reading.deck_type}')
        
        # Questionnaire metrics
        if reading.questionnaire_used:
            metrics.increment(f'questionnaire.completed.{reading.questionnaire_type}')
            metrics.timing('questionnaire.completion_time', reading.questionnaire_time)
        
        # User engagement
        metrics.timing('session.duration', reading.session_duration)
        metrics.gauge('user.satisfaction', reading.satisfaction_score)

    def track_system_health(self):
        """Monitor system performance metrics"""
        metrics.gauge('system.interpretation_queue', self.queue_size)
        metrics.timing('api.response_time', self.api_latency)
        metrics.gauge('cache.hit_rate', self.cache_hit_rate)
        metrics.gauge('error.rate', self.error_rate)

class PerformanceMonitor:
    """Monitor reading system performance"""
    
    THRESHOLDS = {
        'reading_generation': {
            'warning': 5.0,  # seconds
            'critical': 10.0
        },
        'api_latency': {
            'warning': 2.0,  # seconds
            'critical': 5.0
        },
        'error_rate': {
            'warning': 0.05,  # 5%
            'critical': 0.10
        },
        'queue_size': {
            'warning': 100,
            'critical': 500
        }
    }
    
    def check_performance(self):
        """Verify system performance against thresholds"""
        for metric, value in self.collect_metrics().items():
            if value > self.THRESHOLDS[metric]['critical']:
                self.alert_critical(metric, value)
            elif value > self.THRESHOLDS[metric]['warning']:
                self.alert_warning(metric, value)
```

#### Monitoring Dashboards
```python
class ReadingDashboards:
    """Define reading system monitoring views"""
    
    DASHBOARDS = {
        "reading_performance": {
            "metrics": [
                "readings.completed.*",
                "reading.generation_time",
                "reading.quality_score",
                "spreads.used.*"
            ],
            "refresh_rate": "1m",
            "alerts": {
                "slow_readings": "reading.generation_time > 10s",
                "low_quality": "reading.quality_score < 0.8"
            }
        },
        "user_experience": {
            "metrics": [
                "session.duration",
                "user.satisfaction",
                "questionnaire.completed.*"
            ],
            "refresh_rate": "5m",
            "alerts": {
                "low_satisfaction": "user.satisfaction < 0.7",
                "high_abandonment": "session.abandonment > 0.1"
            }
        },
        "system_health": {
            "metrics": [
                "system.interpretation_queue",
                "api.response_time",
                "cache.hit_rate",
                "error.rate"
            ],
            "refresh_rate": "30s",
            "alerts": {
                "high_queue": "system.interpretation_queue > 500",
                "high_latency": "api.response_time > 5s"
            }
        }
    }
```

## Related Documentation
- [System Architecture](architecture.md) - System design
- [Deployment Guide](deployment.md) - Deployment process
- [Data Management](data-management.md) - Data operations