# Personalized Tarot Reading Bot Design Document
*Comprehensive Technical Specification and Implementation Guide*

## 1. Introduction

The Personalized Tarot Reading Bot provides users with tailored tarot readings by combining traditional tarot with modern AI-driven insights. Using the OpenAI API for generating personalized interpretations, the bot delivers an accessible and intuitive experience through Telegram.

### 1.1 Purpose
- Provide accessible, personalized tarot readings
- Maintain authenticity of tarot reading practice
- Ensure reliable and user-friendly experience

### 1.2 Target Audience
- Tarot enthusiasts seeking convenient readings
- Newcomers to tarot
- Regular Telegram users
- Professional tarot readers

### 1.3 Core Features
- Card selection and interpretation
- Basic and advanced spread options
- Flexible credit-based payments
- User session management
- Comprehensive reading history
- Daily insights and guidance

## 2. System Architecture

The system employs a modular monolith architecture, organizing functionality into clear boundaries while maintaining simplicity and ease of maintenance.

### 2.1 Core Modules

1. **Bot Interface Module**
   - Handles Telegram API integration
   - Manages user commands and responses
   - Maintains user sessions
   - Routes requests to appropriate modules
   - Handles rate limiting and throttling

2. **Tarot Core Module**
   - Manages card selection and spreads
   - Coordinates reading generation
   - Handles OpenAI API integration
   - Processes interpretation results
   - Manages deck preferences

3. **Data Management Module**
   - Handles database operations
   - Manages caching strategy
   - Stores user data and readings
   - Maintains credit balances
   - Handles data retention policies

4. **Payment Module**
   - Processes payments securely
   - Manages credit system
   - Handles subscription tiers
   - Records detailed transactions
   - Manages refunds and disputes

### 2.2 Module Communication
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

### 2.3 Enhanced Error Handling
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
    
    async def handle_operation(
        self,
        operation: Callable,
        error_type: str,
        max_retries: int = 3,
        retry_delay: int = 1
    ) -> Result:
        """
        Enhanced error handling with:
        - Error type classification
        - Proper retry logic
        - User-friendly messages
        - Comprehensive logging
        - Error tracking
        """
        error_config = self.ERROR_TYPES.get(
            error_type,
            {
                "retryable": False,
                "user_message": "An error occurred",
                "log_level": "error"
            }
        )
        
        for attempt in range(max_retries if error_config["retryable"] else 1):
            try:
                result = await operation()
                return Result(success=True, data=result)
                
            except Exception as e:
                # Log error with context
                log_message = (
                    f"Operation failed: {str(e)}\n"
                    f"Type: {error_type}\n"
                    f"Attempt: {attempt + 1}/{max_retries}"
                )
                
                logger_method = getattr(
                    logger,
                    error_config["log_level"]
                )
                logger_method(log_message)
                
                # Track error
                await self._track_error(error_type, str(e))
                
                # Last attempt or non-retryable
                if (
                    not error_config["retryable"] or
                    attempt == max_retries - 1
                ):
                    return Result(
                        success=False,
                        error=error_config["user_message"],
                        details=str(e)
                    )
                
                # Wait before retry with exponential backoff
                await asyncio.sleep(retry_delay * (2 ** attempt))
```

## 3. Data Management

### 3.1 Enhanced Database Schema
```sql
-- Users with enhanced fields
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

-- Readings with additional tracking
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

-- Enhanced transactions
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

-- Session tracking
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

-- Rate limiting
CREATE TABLE rate_limits (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    endpoint VARCHAR(100) NOT NULL,
    request_count INTEGER DEFAULT 1,
    window_start TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (user_id, endpoint, window_start)
);
```

### 3.2 Enhanced Caching Strategy
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

## 4. Payment System

### 4.1 Enhanced Pricing Tiers
```python
class PricingTiers:
    """Defines comprehensive pricing structure"""
    
    READING_PRICES = {
        # Basic Readings
        "single": 5,      # 5 credits
        "three": 12,      # 12 credits
        "celtic": 15,     # 15 credits
        
        # Advanced Readings
        "relationship": 12,    # 12 credits
        "career": 12,         # 12 credits
        "week_ahead": 20,     # 20 credits
        
        # Premium Readings
        "monthly_guidance": 100,   # 100 credits
        "yearly_path": 500,       # 500 credits
        "life_path": 1000         # 1000 credits
    }
    
    CREDIT_PACKAGES = {
        "starter": {"credits": 50, "price": 10.00},
        "basic": {"credits": 120, "price": 20.00},
        "premium": {"credits": 300, "price": 45.00},
        "professional": {"credits": 1000, "price": 140.00},
        "unlimited": {"credits": 5000, "price": 600.00}
    }

    SUBSCRIPTION_TIERS = {
        "basic": {
            "price": 15.00,
            "monthly_credits": 150,
            "benefits": ["10% bonus credits", "Basic spreads only"]
        },
        "premium": {
            "price": 45.00,
            "monthly_credits": 500,
            "benefits": ["15% bonus credits", "All spreads except life_path"]
        },
        "professional": {
            "price": 100.00,
            "monthly_credits": 1000,
            "benefits": ["20% bonus credits", "All spreads", "Priority support"]
        }
    }
```

### 4.2 Enhanced Credit Management
```python
class CreditManager:
    """Handles comprehensive credit operations"""
    
    async def process_purchase(
        self,
        user_id: int,
        package: str,
        payment_method: str
    ) -> Result:
        """
        Enhanced credit purchase process:
        1. Validate purchase request
        2. Reserve credits
        3. Process payment
        4. Confirm credit addition
        5. Record transaction
        6. Handle failures at any step
        """
        package_info = PricingTiers.CREDIT_PACKAGES.get(package)
        if not package_info:
            return Result(success=False, error="Invalid package")

        async with self.db.transaction() as transaction:
            try:
                # Validate user account
                user = await self.db.get_user(user_id)
                if not user.is_active:
                    return Result(success=False, error="Account inactive")

                # Reserve credits
                reservation = await self.db.reserve_credits(
                    user_id,
                    package_info["credits"]
                )
                
                # Process payment
                payment_result = await self.payment_processor.charge(
                    user_id,
                    package_info["price"],
                    payment_method,
                    idempotency_key=reservation.id
                )
                
                if payment_result.success:
                    # Confirm credit addition
                    await self.db.confirm_credits(reservation.id)
                    
                    # Record transaction
                    await self.db.record_transaction(
                        user_id=user_id,
                        amount=package_info["credits"],
                        type="purchase",
                        payment_ref=payment_result.reference,
                        status="completed"
                    )
                    
                    return Result(
                        success=True,
                        data={
                            "credits_added": package_info["credits"],
                            "new_balance": await self.db.get_credit_balance(user_id)
                        }
                    )
                
                # Payment failed
                await transaction.rollback()
                return Result(
                    success=False,
                    error="Payment failed",
                    details=payment_result.error
                )
                
            except Exception as e:
                await transaction.rollback()
                logger.error(f"Purchase failed: {str(e)}")
                return Result(
                    success=False,
                    error="System error",
                    details=str(e)
                )
```

## 5. Reading System

### 5.1 Available Spreads
```python
class SpreadTypes:
    """Defines available tarot spreads"""
    
    SPREADS = {
        # Quick Readings
        "single": {
            "name": "Single Card",
            "positions": 1,
            "description": "Quick insight or daily guidance",
            "credits": 5,
            "questionnaire": False
        },
        "three_card": {
            "name": "Past-Present-Future",
            "positions": 3,
            "description": "Temporal insight into a situation",
            "credits": 12,
            "questionnaire": False
        },

        # Situation Readings
        "celtic_cross": {
            "name": "Celtic Cross",
            "positions": 10,
            "description": "Comprehensive situation analysis",
            "credits": 15,
            "questionnaire": False
        },
        "relationship": {
            "name": "Relationship Spread",
            "positions": 5,
            "description": "Insight into relationships",
            "credits": 12,
            "questionnaire": True,
            "questionnaire_type": "relationship_context"
        },
        "career": {
            "name": "Career Path",
            "positions": 4,
            "description": "Career and professional guidance",
            "credits": 12,
            "questionnaire": True,
            "questionnaire_type": "career_goals"
        },

        # Time-Based Readings
        "week_ahead": {
            "name": "Week Ahead",
            "positions": 7,
            "description": "Guidance for the coming week",
            "credits": 20,
            "questionnaire": True,
            "questionnaire_type": "weekly_focus"
        },
        "monthly_guidance": {
            "name": "Month Ahead",
            "positions": 12,
            "description": "Monthly overview and guidance",
            "credits": 100,
            "questionnaire": True,
            "questionnaire_type": "monthly_goals"
        },
        "yearly_path": {
            "name": "Year Ahead",
            "positions": 13,
            "description": "Year overview with monthly insights",
            "credits": 500,
            "questionnaire": True,
            "questionnaire_type": "yearly_aspirations"
        },
        "life_path": {
            "name": "Life Path Reading",
            "positions": 15,
            "description": "Deep life path analysis and guidance",
            "credits": 1000,
            "questionnaire": True,
            "questionnaire_type": "life_assessment"
        }
    }
```

### 5.2 Deck Configuration
```python
class DeckManager:
    """Manages available tarot decks"""
    
    DECKS = {
        "rider_waite": {
            "name": "Rider-Waite-Smith",
            "cards": 78,
            "description": "Traditional tarot deck",
            "default": True
        },
        "thoth": {
            "name": "Thoth",
            "cards": 78,
            "description": "Aleister Crowley's deck",
            "default": False
        },
        "marseille": {
            "name": "Marseille",
            "cards": 78,
            "description": "Historical French deck",
            "default": False
        }
    }

    def get_deck_cards(self, deck_name: str) -> List[Card]:
        """Returns full card list for specified deck"""
        deck = self.DECKS.get(deck_name)
        if not deck:
            raise ValueError(f"Unknown deck: {deck_name}")
            
        return self._load_deck_data(deck_name)
```

### 5.3 Reading Interpretation Factors

#### 5.3.1 Card Order and Position
```python
class ReadingInterpreter:
    """Handles comprehensive reading interpretation"""
    
    def interpret_reading(
        self,
        cards: List[Card],
        positions: List[str],
        directions: List[str],
        spread_type: str,
        question: str
    ) -> ReadingInterpretation:
        """
        Generates a comprehensive reading interpretation considering:
        1. Card order and position significance
        2. Card combinations and interactions
        3. Card directions (upright/reversed)
        4. Spread type context
        5. User's question
        """
        # Analyze card order significance
        order_analysis = self._analyze_card_order(cards, positions)
        
        # Analyze card combinations
        combinations = self._analyze_card_combinations(cards, positions)
        
        # Consider card directions
        direction_impacts = self._analyze_directions(cards, directions)
        
        # Integrate all factors
        return self._generate_integrated_interpretation(
            order_analysis,
            combinations,
            direction_impacts,
            spread_type,
            question
        )
    
    def _analyze_card_order(
        self,
        cards: List[Card],
        positions: List[str]
    ) -> OrderAnalysis:
        """
        Analyzes the significance of card order:
        - Sequential story progression
        - Position-based meaning modification
        - Temporal flow (past->present->future)
        - Energy progression
        - Narrative arc development
        """
        order_impacts = []
        for i, (card, position) in enumerate(zip(cards, positions)):
            # Consider position-specific meanings
            position_impact = POSITION_MEANINGS[position]
            
            # Analyze relationship with adjacent cards
            prev_card = cards[i-1] if i > 0 else None
            next_card = cards[i+1] if i < len(cards)-1 else None
            
            # Determine progression influence
            progression = self._analyze_progression(
                prev_card,
                card,
                next_card,
                position
            )
            
            order_impacts.append({
                "card": card,
                "position": position,
                "position_impact": position_impact,
                "progression": progression
            })
            
        return OrderAnalysis(impacts=order_impacts)
    
    def _analyze_card_combinations(
        self,
        cards: List[Card],
        positions: List[str]
    ) -> CombinationAnalysis:
        """
        Analyzes meaningful card combinations:
        - Element interactions (fire, water, air, earth)
        - Number patterns and numerological significance
        - Suit patterns and elemental balances
        - Archetype combinations and story patterns
        - Reinforcing or contrasting energies
        """
        combinations = []
        
        # Analyze adjacent pairs
        for i in range(len(cards) - 1):
            pair_impact = self._analyze_pair(
                cards[i],
                cards[i + 1],
                positions[i],
                positions[i + 1]
            )
            combinations.append(pair_impact)
        
        # Analyze triads if applicable
        if len(cards) >= 3:
            for i in range(len(cards) - 2):
                triad_impact = self._analyze_triad(
                    cards[i:i+3],
                    positions[i:i+3]
                )
                combinations.append(triad_impact)
        
        # Analyze overall spread patterns
        pattern_impact = self._analyze_spread_pattern(cards, positions)
        combinations.append(pattern_impact)
        
        return CombinationAnalysis(combinations=combinations)
    
    def _analyze_directions(
        self,
        cards: List[Card],
        directions: List[str]
    ) -> DirectionAnalysis:
        """
        Analyzes impact of card directions:
        - Individual card reversals
        - Pattern of reversals across spread
        - Modified energy flows
        - Blocked or challenged aspects
        - Enhanced or diminished qualities
        """
        direction_impacts = []
        
        # Analyze individual card directions
        for card, direction in zip(cards, directions):
            impact = self._analyze_direction_impact(card, direction)
            direction_impacts.append(impact)
        
        # Analyze overall direction patterns
        reversal_pattern = self._analyze_reversal_pattern(directions)
        
        return DirectionAnalysis(
            individual_impacts=direction_impacts,
            pattern_impact=reversal_pattern
        )
```

### 5.4 Bot Communication

#### 5.4.1 Message Templates
```python
class MessageTemplates:
    """Defines professional and engaging message templates"""
    
    MESSAGES = {
        "welcome": "🔮 Welcome to your tarot reading space. I'm here to help you explore the cards and their insights.",
        
        "reading_start": "🎴 Beginning your {spread_type} reading. Take a moment to reflect on your question.",
        
        "card_drawn": "{position_name}: {card_name}\n{position_meaning}",
        
        "position_meanings": {
            "past_path": "📜 What has shaped your present path",
            "present_mirror": "🌟 Current influences in your situation",
            "future_path": "🌅 Possibilities unfolding ahead",
            "mind_space": "💭 Thoughts and conscious influences",
            "foundation": "🌱 Core themes supporting you",
            "inner_self": "💫 Your personal perspective",
            "outer_world": "🌍 External factors shaping your path"
        },
        
        "interpretation_ready": "🎯 Your reading is ready to explore.",
        
        "credits_low": "🕯️ You have {credits} readings remaining. Would you like to add more to continue?",
        
        "purchase_success": """
        🎯 Readings Added
        • New readings: {amount}
        • Total available: {new_total}
        Ready for your next exploration
        """,
        
        "error": "🌑 Something didn't work quite right: {error_message}. Let's try again, or reach out to support if you need help.",
        
        "daily_insight": """
        ⭐ Your Card Today
        Card: {card}
        Insight: {message}
        """,
        
        "help": """
        Available Options

        Readings
        /read 🔮 - Start a reading
        /spreads 🎴 - View available spreads
        /decks 📚 - Choose your deck
        /today ⭐ - Today's card

        Your Account
        /past 📖 - Past readings
        /balance ✨ - Available readings
        /add ➕ - Get more readings
        /preferences ⚙️ - Your settings
        """
    }

    def format_reading_result(self, reading: Reading) -> str:
        """Formats reading result with clear structure"""
        return f"""
        Theme:
        {reading.theme}

        Card Analysis:
        {reading.card_analysis}

        Guidance:
        {reading.practical_advice}

        Consider taking a moment to reflect on these insights.
        """

    def format_card_details(
        self,
        card: Card,
        position: str,
        orientation: str
    ) -> str:
        """Formats detailed card information"""
        return f"""
        {card.name} - {position}
        
        Position: {self.position_meanings.get(position, position)}
        Orientation: {orientation}
        
        Traditional Meaning:
        {card.meaning[orientation]}
        
        Key Elements:
        • {card.element}
        • {card.astrological_association}
        
        Core Themes:
        {', '.join(card.themes)}
        
        Consider:
        {card.reflection_prompt}
        """
```

### 5.5 Questionnaire System

#### 5.5.1 Questionnaire Types
```python
class QuestionnaireManager:
    """Manages reading questionnaires for enhanced context"""
    
    QUESTIONNAIRES = {
        "weekly_focus": {
            "name": "🌅 Week Ahead",
            "questions": [
                "What would you like to focus on this week?",
                "What challenges are you navigating?",
                "Which areas of life need attention now?",
                "What recent changes are influencing you?"
            ]
        },
        "monthly_goals": {
            "name": "🌙 Month's Journey",
            "questions": [
                "What changes would you like to see this month?",
                "What important events are approaching?",
                "Where do you feel ready for growth?",
                "What situations need clarity?",
                "What support do you have available?"
            ]
        },
        "yearly_aspirations": {
            "name": "🌟 Year's Path",
            "questions": [
                "Which life areas need guidance?",
                "What do you hope to achieve this year?",
                "What changes are you preparing for?",
                "How would you like to grow?",
                "What are your work aspirations?",
                "How are your relationships evolving?",
                "What personal growth interests you?"
            ]
        },
        "life_assessment": {
            "name": "🌍 Life Journey",
            "questions": [
                "How aligned do you feel with your path? (1-10)",
                "What themes keep appearing in your life?",
                "What matters most to you now?",
                "What questions about purpose arise?",
                "How do your relationships shape you?",
                "How does your past influence today?",
                "What dreams guide you forward?",
                "Where do you seek growth?",
                "What gives your life meaning?",
                "How do different parts of life balance?"
            ]
        }
    }

    async def conduct_questionnaire(
        self,
        questionnaire_type: str,
        user_id: int
    ) -> QuestionnaireResult:
        """
        Conducts questionnaire session:
        1. Validates questionnaire type
        2. Presents questions sequentially
        3. Collects and processes responses
        4. Generates context for reading
        """
        questions = self.QUESTIONNAIRES[questionnaire_type]["questions"]
        responses = await self._collect_responses(user_id, questions)
        return self._process_responses(responses, questionnaire_type)

    def _process_responses(
        self,
        responses: Dict[str, str],
        questionnaire_type: str
    ) -> QuestionnaireResult:
        """
        Analyzes questionnaire responses:
        - Identifies key themes
        - Extracts priorities
        - Maps to reading aspects
        - Generates reading context
        """
        analysis = self._analyze_responses(responses)
        return QuestionnaireResult(
            themes=analysis.themes,
            priorities=analysis.priorities,
            context=analysis.context,
            focus_areas=analysis.focus_areas
        )
```

### 5.6 Prompt Design

#### 5.6.1 Base Prompt Template
```python
class PromptTemplate:
    """Manages OpenAI prompt generation"""
    
    BASE_PROMPT = """You are an experienced tarot reader offering guidance through {spread_type}.
Your role is to illuminate insights while honoring both the wisdom of tarot and practical application.

Reading Context:
🔮 Question: {question}
🎴 Spread: {spread_name}
📚 Deck: {deck_name}

Cards Revealed:
{cards_drawn}

Shape your interpretation to:
1. Address the heart of the question
2. Reveal each card's unique message in its position
3. Bridge insight into practical steps
4. Maintain clarity and authenticity

Structure your guidance as:
1. Key Theme & Energy
2. Journey Through the Cards
3. Practical Wisdom
"""

    def generate_prompt(
        self,
        question: str,
        spread_type: str,
        cards: List[Card],
        deck_name: str
    ) -> str:
        """Generates formatted prompt for OpenAI"""
        cards_text = "\n".join([
            f"- Position {i+1}: {card.name} ({card.orientation})"
            for i, card in enumerate(cards)
        ])
        
        return self.BASE_PROMPT.format(
            question=question,
            spread_type=spread_type,
            spread_name=SpreadTypes.SPREADS[spread_type]["name"],
            deck_name=deck_name,
            cards_drawn=cards_text
        )
```

## 6. Deployment

### 6.1 Enhanced Deployment Configuration
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

### 6.2 System Architecture Diagram
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

### 6.3 Component Details

1. **Load Balancer**
   - Basic HTTP/HTTPS routing
   - SSL termination
   - Health checks

2. **Application Server**
   - Python FastAPI application
   - Modular monolith structure
   - Environment-based configuration

3. **PostgreSQL**
   - User data
   - Reading history
   - Transaction records

4. **Redis**
   - Session caching
   - Rate limiting
   - Basic result caching

5. **Monitoring**
   - Basic health metrics
   - Error logging
   - Performance monitoring

### 6.4 Deployment Process
1. Build application
2. Run database migrations
3. Deploy to server
4. Verify health checks

## 7. Project Structure

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

## 8. Future Considerations

### 8.1 Immediate Priorities
- Stable core functionality
- Reliable payment processing
- Basic monitoring
- User feedback collection
- Comprehensive test coverage
- Documentation maintenance

### 8.2 Future Enhancements
- Additional spread types and deck options
- Multi-language support
- Enhanced analytics and insights
- User preferences and customization
- API access for third-party integration
- Community features and sharing
