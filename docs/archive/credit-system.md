# Credit System Documentation

## Overview
The credit system manages user credits for tarot readings, handling purchases, deductions, and subscription benefits. This system integrates closely with the payment system to ensure consistent and reliable credit management.

## Credit Costs

### Reading Credits
| Reading Type        | Credits | Description                    |
|--------------------|---------|--------------------------------|
| single             | 5       | Quick insight reading          |
| three             | 12      | Past-Present-Future spread     |
| celtic            | 15      | Comprehensive situation spread |
| relationship      | 12      | Relationship guidance spread   |
| career            | 12      | Career path guidance spread    |
| week_ahead        | 20      | Weekly guidance spread         |
| monthly_guidance  | 100     | Monthly overview spread        |
| yearly_path       | 500     | Yearly overview spread         |
| life_path         | 1000    | Complete life path analysis    |

### Credit Packages
| Package      | Credits | Price (USD) | Value Ratio |
|--------------|---------|-------------|-------------|
| starter      | 50      | $10.00      | $0.20/credit|
| basic        | 120     | $20.00      | $0.17/credit|
| premium      | 300     | $45.00      | $0.15/credit|
| professional | 1000    | $140.00     | $0.14/credit|
| unlimited    | 5000    | $600.00     | $0.12/credit|

## Subscription Tiers

### Basic Tier
- Price: $15.00/month
- Monthly Credits: 150
- Benefits:
  - 10% bonus credits on purchases
  - Access to basic spreads (single, three, celtic)
- Rate Limits:
  - 100 requests per minute
  - Standard processing priority

### Premium Tier
- Price: $45.00/month
- Monthly Credits: 500
- Benefits:
  - 15% bonus credits on purchases
  - Access to all spreads except life_path
  - Enhanced processing priority
- Rate Limits:
  - 300 requests per minute
  - Priority processing

### Professional Tier
- Price: $100.00/month
- Monthly Credits: 1000
- Benefits:
  - 20% bonus credits on purchases
  - Access to all spreads including life_path
  - Priority support and processing
- Rate Limits:
  - 500 requests per minute
  - Highest priority processing

## Credit Operations

### Purchase Flow
```python
async def process_credit_purchase(user_id: int, package: str, payment_method: str) -> Result:
    """
    Process credit purchase with comprehensive validation and error handling
    
    Args:
        user_id: Unique identifier for the user
        package: Credit package identifier
        payment_method: Payment method identifier
        
    Returns:
        Result object containing success status and transaction details
    """
    async with transaction_manager.transaction() as txn:
        try:
            # 1. Validate user account
            user = await user_service.get_user(user_id)
            if not user.is_active:
                return Result(success=False, error="Account inactive")

            # 2. Validate package
            package_info = await credit_service.get_package(package)
            if not package_info:
                return Result(success=False, error="Invalid package")

            # 3. Process payment
            payment_result = await payment_service.process_payment(
                user_id=user_id,
                amount=package_info.price,
                payment_method=payment_method,
                description=f"Credit package: {package}"
            )

            if not payment_result.success:
                return Result(success=False, error=payment_result.error)

            # 4. Calculate credits with bonuses
            total_credits = await credit_service.calculate_credits_with_bonus(
                user_id=user_id,
                base_credits=package_info.credits
            )

            # 5. Add credits to user account
            await credit_service.add_credits(
                user_id=user_id,
                amount=total_credits,
                transaction_id=payment_result.transaction_id
            )

            # 6. Record transaction
            await transaction_service.record_credit_purchase(
                user_id=user_id,
                package=package,
                credits=total_credits,
                payment_ref=payment_result.reference
            )

            return Result(
                success=True,
                data={
                    "credits_added": total_credits,
                    "new_balance": await credit_service.get_balance(user_id),
                    "transaction_id": payment_result.transaction_id
                }
            )

        except Exception as e:
            await txn.rollback()
            logger.error(f"Credit purchase failed: {str(e)}")
            return Result(success=False, error="System error", details=str(e))
```

### Reading Deduction Flow
```python
async def process_reading_deduction(
    user_id: int,
    reading_type: str,
    reading_id: str
) -> Result:
    """
    Process credit deduction for a reading request
    
    Args:
        user_id: Unique identifier for the user
        reading_type: Type of reading requested
        reading_id: Unique identifier for the reading
        
    Returns:
        Result object containing success status and transaction details
    """
    async with transaction_manager.transaction() as txn:
        try:
            # 1. Validate reading type and get cost
            reading_cost = await credit_service.get_reading_cost(reading_type)
            if not reading_cost:
                return Result(success=False, error="Invalid reading type")

            # 2. Check user subscription and access
            subscription = await subscription_service.get_subscription(user_id)
            if not subscription.can_access_reading(reading_type):
                return Result(success=False, error="Reading type not available in current subscription")

            # 3. Verify credit balance
            if not await credit_service.has_sufficient_credits(user_id, reading_cost):
                return Result(success=False, error="Insufficient credits")

            # 4. Reserve credits
            reservation = await credit_service.reserve_credits(
                user_id=user_id,
                amount=reading_cost,
                reading_id=reading_id
            )

            # 5. Record transaction
            await transaction_service.record_reading_deduction(
                user_id=user_id,
                reading_type=reading_type,
                credits=reading_cost,
                reading_id=reading_id
            )

            return Result(
                success=True,
                data={
                    "credits_reserved": reading_cost,
                    "reservation_id": reservation.id,
                    "new_balance": await credit_service.get_balance(user_id)
                }
            )

        except Exception as e:
            await txn.rollback()
            logger.error(f"Reading deduction failed: {str(e)}")
            return Result(success=False, error="System error", details=str(e))
```

## Error Handling

### Failed Payments
- Transaction marked as failed
- No credits allocated
- User notified with specific error message
- Retry option provided with clear instructions
- All operations logged for audit

### Failed Readings
- Reserved credits automatically released
- Detailed error logging with context
- User notified with retry instructions
- Transaction rolled back atomically
- Error metrics updated

### Partial Failures
- Transaction rolled back completely
- System state verified and restored
- Detailed error logging for investigation
- User notified with clear status
- Support ticket created if needed

## Security Measures

### Credit Operations
- All operations are atomic and transactional
- Comprehensive audit logging
- Concurrent operation protection
- Balance verification checks
- Rate limiting enforcement

### Payment Processing
- PCI DSS compliant implementation
- Encrypted payment data handling
- Secure payment gateway integration
- Fraud detection measures
- Transaction verification

## Monitoring

### Credit System Health
- Credit operation success rate
- Transaction processing time
- Error rate monitoring
- Balance accuracy checks
- System performance metrics

### Alert Thresholds
- Error rate > 1%
- Transaction time > 2s
- Failed operations > 5/minute
- System load > 70%
- Concurrent operations > 1000/minute

## Related Documentation
- [Payment System](payment-system.md) - Detailed payment processing
- [Reading System](reading-system.md) - Reading types and costs
- [Data Management](data-management.md) - Transaction storage
- [Security](security.md) - Security measures and compliance
- [Monitoring](monitoring.md) - System monitoring and alerts