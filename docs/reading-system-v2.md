# Reading System

## Document Information
- **Last Updated:** 2025-02-08
- **Version:** 1.0.0
- **Status:** Draft
- **Owner:** Reading System Team

## Overview
The Reading System is the core component responsible for delivering tarot card readings to users through the Telegram bot interface. It handles spread selection, card selection, interpretation generation, and result delivery.

## Background
The system combines traditional tarot reading practices with modern AI-driven interpretation to provide accurate and meaningful readings. It supports multiple spread types and tarot decks while maintaining the authenticity of tarot reading practices.

## Detailed Content

### Spread System
The Reading System supports multiple spread types categorized by purpose:

#### Quick Readings
- Single Card (5 credits)
  - Purpose: Quick insight or daily guidance
  - Positions: 1
  - No questionnaire required

- Past-Present-Future (12 credits)
  - Purpose: Temporal insight into a situation
  - Positions: 3
  - No questionnaire required

#### Situation Readings
- Celtic Cross (15 credits)
  - Purpose: Comprehensive situation analysis
  - Positions: 10
  - No questionnaire required

- Relationship Spread (12 credits)
  - Purpose: Relationship insights
  - Positions: 5
  - Requires relationship context questionnaire

- Career Path (12 credits)
  - Purpose: Professional guidance
  - Positions: 4
  - Requires career goals questionnaire

#### Time-Based Readings
- Week Ahead (20 credits)
- Month Ahead (100 credits)
- Year Ahead (500 credits)
- Life Path Reading (1000 credits)

### Deck Management
Available decks include:
- Rider-Waite-Smith (Default)
- Thoth
- Marseille

Each deck contains 78 cards and maintains its traditional symbolism and interpretation patterns.

### Interpretation System
The interpretation process generates AI-driven analysis of cards in context of the user's question, considering multiple factors:
1. Card position significance
2. Card combinations and interactions
3. Card directions (upright/reversed)
4. Spread type context
5. User's question or focus
6. Questionnaire responses (when applicable)

### Questionnaire System
Enhanced readings include contextual questionnaires:
- Weekly Focus
- Monthly Goals
- Yearly Aspirations
- Life Assessment

Each questionnaire is designed to gather relevant context for more accurate and personalized readings.

## Technical Specifications

### Dependencies
- OpenAI Integration for interpretation generation
- Database (PostgreSQL) for reading history storage
- Cache (Redis) for frequent interpretations

### Performance Requirements
- Reading generation: < 2 seconds (Response Time target)
- History retrieval: < 1 second
- Concurrent reading support: 100+ simultaneous readings

## Related Documents
- [Glossary](glossary.md) - Standard terminology reference
- [System Architecture](architecture-v2.md) - System integration details
- [Data Management](data-management-v2.md) - Reading history storage
- [Development Guide](development-v2.md) - Implementation details

## Appendix

### Code References
The implementation details for spreads, decks, interpretation logic, and questionnaires can be found in:
- `src/reading/spreads.py`
- `src/reading/decks.py`
- `src/reading/interpreter.py`
- `src/reading/questionnaires.py`

### Message Templates
Standard message templates for user communication are maintained in `src/templates/reading_messages.py`

## Version History
| Version | Date | Author | Changes |
|---------|------|---------|---------|
| 1.0.0 | 2025-02-08 | Reading System Team | Initial version following new template |

## Notes
- Future updates planned for additional spread types
- Considering adding support for custom deck implementations
- Planning enhanced interpretation algorithms using advanced AI models