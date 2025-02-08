# Payment System

## Pricing Structure

### Reading Prices
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
```

### Credit Packages
```python
    CREDIT_PACKAGES = {
        "starter": {"credits": 50, "price": 10.00},
        "basic": {"credits": 120, "price": 20.00},
        "premium": {"credits": 300, "price": 45.00},
        "professional": {"credits": 1000, "price": 140.00},
        "unlimited": {"credits": 5000, "price": 600.00}
    }
```

### Subscription Tiers
```python
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

## Credit Management

### Purchase Processing
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

## Transaction Flow

### Credit Purchase Flow
1. User selects credit package
2. System validates user account
3. Credits are reserved
4. Payment is processed
5. Credits are confirmed
6. Transaction is recorded

### Reading Credit Deduction
1. User requests reading
2. System verifies credit balance
3. Credits are reserved
4. Reading is generated
5. Credits are deducted
6. Transaction is recorded

### Subscription Management
1. User subscribes to tier
2. Monthly payment is processed
3. Monthly credits are allocated
4. Subscription status is updated
5. Benefits are activated

## Error Handling

### Payment Errors
- Invalid payment method
- Insufficient funds
- Payment provider errors
- Network timeouts
- Transaction conflicts

### Credit Operation Errors
- Insufficient credits
- Invalid package selection
- Reservation conflicts
- Database errors
- Concurrency issues

## Security Measures

### Payment Processing
- Secure payment gateway integration
- PCI DSS compliance
- Encryption of sensitive data
- Fraud detection mechanisms
- Transaction logging

### Credit Management
- Atomic transactions
- Audit logging
- Balance verification
- Concurrency control
- Error recovery

## Related Documentation
- [Data Management](data-management.md) - Database schema and operations
- [System Architecture](architecture.md) - System integration points
- [Development Guide](development.md) - Implementation details