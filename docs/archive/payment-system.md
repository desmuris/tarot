# Payment System

## Overview
The payment system handles all financial transactions, integrating with the credit system to manage purchases, subscriptions, and refunds. This system ensures secure, reliable payment processing while maintaining compliance with financial regulations.

## Pricing Structure

### Reading Prices
```python
class ReadingPrices:
    """Defines comprehensive reading price structure"""
    
    READING_COSTS = {
        # Basic Readings
        "single": {
            "credits": 5,
            "description": "Quick insight reading"
        },
        "three": {
            "credits": 12,
            "description": "Past-Present-Future spread"
        },
        "celtic": {
            "credits": 15,
            "description": "Comprehensive situation spread"
        },
        
        # Advanced Readings
        "relationship": {
            "credits": 12,
            "description": "Relationship guidance spread"
        },
        "career": {
            "credits": 12,
            "description": "Career path guidance spread"
        },
        "week_ahead": {
            "credits": 20,
            "description": "Weekly guidance spread"
        },
        
        # Premium Readings
        "monthly_guidance": {
            "credits": 100,
            "description": "Monthly overview spread"
        },
        "yearly_path": {
            "credits": 500,
            "description": "Yearly overview spread"
        },
        "life_path": {
            "credits": 1000,
            "description": "Complete life path analysis"
        }
    }
```

### Credit Packages
```python
class CreditPackages:
    """Defines credit package pricing and details"""
    
    PACKAGES = {
        "starter": {
            "credits": 50,
            "price": 10.00,
            "value_ratio": 0.20  # $/credit
        },
        "basic": {
            "credits": 120,
            "price": 20.00,
            "value_ratio": 0.17
        },
        "premium": {
            "credits": 300,
            "price": 45.00,
            "value_ratio": 0.15
        },
        "professional": {
            "credits": 1000,
            "price": 140.00,
            "value_ratio": 0.14
        },
        "unlimited": {
            "credits": 5000,
            "price": 600.00,
            "value_ratio": 0.12
        }
    }
```

### Subscription Tiers
```python
class SubscriptionTiers:
    """Defines subscription tier benefits and pricing"""
    
    TIERS = {
        "basic": {
            "price": 15.00,
            "monthly_credits": 150,
            "benefits": {
                "bonus_credits": 0.10,  # 10% bonus
                "allowed_readings": ["single", "three", "celtic"],
                "rate_limit": 100,  # requests per minute
                "priority": "standard"
            }
        },
        "premium": {
            "price": 45.00,
            "monthly_credits": 500,
            "benefits": {
                "bonus_credits": 0.15,  # 15% bonus
                "allowed_readings": ["*", "!life_path"],  # all except life_path
                "rate_limit": 300,
                "priority": "high"
            }
        },
        "professional": {
            "price": 100.00,
            "monthly_credits": 1000,
            "benefits": {
                "bonus_credits": 0.20,  # 20% bonus
                "allowed_readings": ["*"],  # all readings
                "rate_limit": 500,
                "priority": "highest"
            }
        }
    }
```

## Payment Processing

### Payment Handler
```python
class PaymentHandler:
    """Handles secure payment processing and integration"""
    
    async def process_payment(
        self,
        user_id: int,
        amount: float,
        payment_method: str,
        description: str,
        idempotency_key: str = None
    ) -> PaymentResult:
        """
        Process payment with comprehensive error handling and security measures
        
        Args:
            user_id: Unique identifier for the user
            amount: Payment amount in USD
            payment_method: Payment method identifier
            description: Payment description
            idempotency_key: Unique key to prevent duplicate processing
            
        Returns:
            PaymentResult object containing transaction details
        """
        try:
            # 1. Validate payment method
            if not await self.validate_payment_method(user_id, payment_method):
                return PaymentResult(success=False, error="Invalid payment method")

            # 2. Prepare payment intent
            payment_intent = await self.payment_gateway.create_intent(
                amount=amount,
                currency="USD",
                payment_method=payment_method,
                description=description,
                metadata={
                    "user_id": user_id,
                    "idempotency_key": idempotency_key
                }
            )

            # 3. Process payment
            result = await self.payment_gateway.process_payment(payment_intent.id)
            
            if result.success:
                # 4. Record successful transaction
                await self.record_transaction(
                    user_id=user_id,
                    amount=amount,
                    payment_ref=result.transaction_id,
                    description=description
                )
                
                return PaymentResult(
                    success=True,
                    transaction_id=result.transaction_id,
                    reference=result.reference
                )
            
            # 5. Handle payment failure
            await self.handle_payment_failure(
                user_id=user_id,
                error=result.error,
                payment_intent=payment_intent
            )
            
            return PaymentResult(
                success=False,
                error=result.error,
                details=result.error_details
            )

        except Exception as e:
            logger.error(f"Payment processing failed: {str(e)}")
            return PaymentResult(
                success=False,
                error="Payment processing failed",
                details=str(e)
            )
```

## Transaction Management

### Credit Purchase Flow
1. User selects credit package
2. System validates user account and payment method
3. Payment intent created with idempotency key
4. Payment processed through secure gateway
5. Credits allocated upon successful payment
6. Transaction recorded with full audit trail

### Subscription Management
1. User subscribes to tier
2. Initial payment processed
3. Subscription record created
4. Monthly credits allocated
5. Recurring payment scheduled
6. Benefits activated immediately

### Refund Processing
1. Refund request validated
2. Credits frozen if applicable
3. Payment refund processed
4. Transaction records updated
5. User notified of refund status
6. Audit trail maintained

## Error Handling

### Payment Failures
- Invalid payment method
- Insufficient funds
- Payment provider errors
- Network timeouts
- Transaction conflicts

### Transaction Errors
- Database errors
- Concurrency issues
- Integration failures
- System timeouts
- State inconsistencies

## Security Measures

### Payment Processing
- PCI DSS compliance
- Secure payment gateway integration
- Encrypted sensitive data
- Fraud detection systems
- Transaction monitoring

### Data Protection
- Encrypted payment data
- Secure credential storage
- Access control enforcement
- Audit logging
- Regular security audits

## Monitoring

### Transaction Monitoring
- Success/failure rates
- Processing times
- Error patterns
- Volume metrics
- Revenue analytics

### Alert Thresholds
- Payment failure rate > 1%
- Processing time > 2s
- Error rate > 5%
- Unusual patterns
- System degradation

## Related Documentation
- [Credit System](credit-system.md) - Credit management and operations
- [Security](security.md) - Security measures and compliance
- [Monitoring](monitoring.md) - System monitoring and alerts
- [Data Management](data-management.md) - Transaction data handling
- [Error Handling](error-handling.md) - Error management procedures