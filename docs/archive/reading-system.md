# Reading System

## Available Spreads

### Spread Types
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

## Deck Configuration

### Available Decks
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
```

## Reading Interpretation

### Interpretation Factors
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
```

## Questionnaire System

### Available Questionnaires
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
```

## Communication Templates

### Message Formatting
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
        }
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
```

## Related Documentation
- [System Architecture](architecture.md) - Integration with other components
- [Data Management](data-management.md) - Storage of readings and user preferences
- [Development Guide](development.md) - Implementation details