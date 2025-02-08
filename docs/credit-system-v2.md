1# Credit System

## Document Information
- **Last Updated:** 2025-02-08
- **Version:** 1.0.0
- **Status:** Draft
- **Owner:** Credit Management Team

## Overview
The Credit System manages the virtual currency used for Readings in the Tarot Reading Bot, handling Credit purchases, deductions, and Subscription benefits. It integrates closely with the Payment System to ensure consistent and reliable Credit management across the platform.

## Background
Credits serve as the primary transaction medium for Readings, requiring careful management of purchases, usage, and Subscription benefits. The system is designed to handle various Credit operations while maintaining strict accuracy and security.

## Detailed Content

### Credit Structure

#### Reading Costs
| Reading Type     | Credits | Description                    |
|-----------------|---------|--------------------------------|
| Single Card     | 5       | Quick insight Reading          |
| Three Card      | 12      | Past-Present-Future Spread     |
| Celtic Cross    | 15      | Comprehensive situation Spread |
| Relationship    | 12      | Relationship guidance Spread   |
| Career          | 12      | Career path guidance Spread    |
| Week Ahead      | 20      | Weekly guidance Spread         |
| Monthly Guidance| 100     | Monthly overview Spread        |
| Yearly Path     | 500     | Yearly overview Spread         |
| Life Path       | 1000    | Complete life path analysis    |

#### Credit Packages
| Package      | Credits | Price (USD) | Value Ratio  |
|--------------|---------|-------------|--------------|
| Starter      | 50      | $10.00      | $0.20/Credit |
| Basic        | 120     | $20.00      | $0.17/Credit |
| Premium      | 300     | $45.00      | $0.15/Credit |
| Professional | 1000    | $140.00     | $0.14/Credit |
| Unlimited    | 5000    | $600.00     | $0.12/Credit |

### Subscription Plans

#### Basic Tier ($15.00/month)
- Monthly Credits: 150
- Benefits:
  * 10% bonus Credits on purchases
  * Access to Basic Operations
  * 100 requests per minute
  * Standard priority

#### Premium Tier ($45.00/month)
- Monthly Credits: 500
- Benefits:
  * 15% bonus Credits on purchases
  * All Spreads except Life Path
  * 300 requests per minute
  * Enhanced priority

#### Professional Tier ($100.00/month)
- Monthly Credits: 1000
- Benefits:
  * 20% bonus Credits on purchases
  * All Spreads including Life Path
  * 500 requests per minute
  * Highest priority

### Credit Operations

#### Purchase Flow
1. Account Validation
   - Verify user status
   - Check package validity
   - Validate payment method

2. Payment Processing
   - Process payment
   - Calculate bonus Credits
   - Handle payment failures

3. Credit Allocation
   - Add base Credits
   - Apply bonus Credits
   - Update balance
   - Record transaction

#### Reading Deduction
1. Validation
   - Check Reading type
   - Verify Subscription access
   - Confirm Credit balance

2. Credit Management
   - Reserve Credits
   - Process Reading
   - Update balance
   - Record transaction

3. Error Handling
   - Release reservations
   - Rollback transactions
   - Notify user
   - Log errors

### Security Implementation

#### Transaction Security
- Atomic operations
- Transactional integrity
- Concurrent operation protection
- Balance verification
- Rate limiting

#### Data Protection
- Encrypted storage
- Secure transmission
- Access controls
- Audit logging
- Regular verification

## Technical Specifications

### Performance Requirements
- Transaction time: < 2 seconds
- Concurrent operations: 1000/minute
- System availability: 99.99%
- Balance accuracy: 100%
- Response Time: < 500ms

### Security Requirements
- Atomic transactions
- Audit logging
- Access control
- Rate limiting
- Fraud detection

### Monitoring Thresholds
- Error rate: < 1%
- Transaction time: < 2s
- Failed operations: < 5/minute
- System load: < 70%
- Concurrent ops: < 1000/minute

## Related Documents
- [Glossary](glossary.md) - Standard terminology reference
- [Payment System](payment-system-v2.md) - Payment Processing
- [Reading System](reading-system-v2.md) - Reading operations
- [Data Management](data-management-v2.md) - Data handling
- [Security](security-v2.md) - Security measures

## Appendix

### Implementation Details
Key Credit System files:
- `src/credit/manager.py`
- `src/credit/operations.py`
- `src/credit/subscription.py`
- `src/credit/security.py`

### Error Handling
1. Failed Payments
   - Transaction marking
   - Credit management
   - User notification
   - Retry handling
   - Audit logging

2. Failed Readings
   - Credit release
   - Error logging
   - User notification
   - Transaction rollback
   - Metric updates

3. Partial Failures
   - State restoration
   - Error logging
   - User notification
   - Support ticketing
   - System verification

## Version History
| Version | Date | Author | Changes |
|---------|------|---------|---------|
| 1.0.0 | 2025-02-08 | Credit Management Team | Initial version following new template |

## Notes
- Daily balance reconciliation
- Weekly security reviews
- Monthly performance analysis
- Quarterly audit checks
- Continuous monitoring active