# Transaction Import Automation

*Technical Documentation - September 2025*

## Overview

The system automatically imports TransferWise transactions via webhook integration, creating incomplete Transaction records that users then complete with business context. This eliminates data entry errors while preserving human control over business logic assignments.

## Technical Architecture

### TransferWise Webhook Integration

**Endpoint**: `/api/webhooks/transferwise/transaction-complete`
**Events**: `transfer_state_change` (when transfer status becomes "outgoing_payment_sent")
**Authentication**: HMAC signature validation

### Manual Transaction Support

**Creation Templates**: Pre-configured templates for cash, M-Pesa, and custom transactions
**User Interface**: Standard Podio record creation with template selection

## Core Automation Logic

### TransferWise Transaction Import

When TransferWise webhook indicates a completed transfer, the system creates an incomplete Transaction record.

#### Auto-Populated Fields
```python
{
    "title": f"TW-{transfer_id} - {source_currency} to Kenya",
    "foundation_account": derive_account_from_profile_id(webhook_data['profile_id']),
    "amount_sent": webhook_data['source_amount'],
    "currency_sent": webhook_data['source_currency'], 
    "amount_received": webhook_data['target_amount'],
    "currency_received": webhook_data['target_currency'],
    "date_sent": webhook_data['created_at'],
    "date_received": webhook_data['finished_at'],
    "transfer_service": "TransferWise",
    "transfer_fee": calculate_fee(source_amount, target_amount),
    "transferwise_id": webhook_data['transfer_id'],
    "creation_method": "API Import",
    "status": "Incomplete",
    "paid_by": foundation_account.paid_by  # Auto-populated from account
}
```

#### Foundation Account Mapping
```python
TRANSFERWISE_PROFILE_MAPPING = {
    "12345": "TransferWise EUR Account",
    "67890": "TransferWise USD Account"
}
```

### Transaction Completion Process

When user completes required fields:

```python
def complete_transaction(transaction_id, user_inputs):
    transaction = Transaction.get(transaction_id)
    
    # Update user-provided fields
    transaction.update({
        'fundingbatch': user_inputs['fundingbatch'],
        'intermediary': user_inputs.get('intermediary'),
        'description': user_inputs['description'],
        'title': user_inputs.get('title', transaction.title),
        'status': 'Complete'
    })
    
    # Update FundingBatch credit
    funding_batch = transaction.fundingbatch
    funding_batch.funded_amount += transaction.amount_received
    
    # Trigger auto-allocation if applicable
    if should_trigger_auto_allocation(funding_batch):
        trigger_auto_allocation(funding_batch)
    
    # Log completion
    add_system_message(transaction, 
        f"User completed business fields - Status changed to Complete")
```

## User Interaction Points

### Pending Transactions Dashboard

**Real-time notification** sent to users when new incomplete transactions are created:
- Email notification with transaction summary
- In-app notification badge
- Dashboard showing pending transactions count

**Dashboard Interface** displays:
- List of all transactions with Status = "Incomplete"
- Quick completion form for each transaction
- Bulk assignment tools for common patterns

**User Actions Required:**
- **FundingBatch**: User must assign to appropriate funding batch
- **Local Intermediary**: User specifies who receives the money (optional)
- **Description**: User adds business context
- **Title**: User can customize auto-generated title

**User Interface Elements:**
- **Completion Form**: Quick input fields for required business context
- **Bulk Assignment**: Tools for applying common patterns to multiple transactions
- **Status Indicators**: Visual feedback on completion progress

## Error Handling

### Webhook Failures

**Duplicate Prevention**: Check TransferWise ID before creating new transaction
**Retry Logic**: Failed webhook processing retried with exponential backoff
**Manual Fallback**: Failed imports can be created manually with TransferWise ID

### Data Validation

**Amount Validation**: Verify amounts are positive and currencies are supported
**Account Validation**: Ensure Foundation Account exists and is active
**Date Validation**: Ensure dates are reasonable and Date Received >= Date Sent

### Exception Logging

All automation events logged to Transaction.system_messages:
```
2025-09-15 16:45: Auto-imported from TransferWise API - TW-98765
2025-09-15 16:45: Validation passed - Foundation Account: TransferWise EUR
2025-09-15 17:20: User completed business fields - Status changed to Complete
2025-09-15 17:25: FundingBatch credit updated - Added 65,750 KES to Student Transport March 2025
```

### Manual Transaction Creation

**Creation Methods** for non-TransferWise transactions:
- **"Cash Transaction" template**: Pre-fills Transfer Service = "Cash", Transfer Fee = 0
- **"M-Pesa Direct" template**: Pre-fills Transfer Service = "M-Pesa Direct"
- **"Custom Transaction"**: Full manual entry

**User Actions Required:**
- Select appropriate template
- Fill all transaction details manually
- Assign to FundingBatch immediately (no incomplete status)

## Integration Points

### FundingBatch System
- Completed transactions automatically update FundingBatch.funded_amount
- Auto-allocation triggers when FundingBatch amounts match linked NeedPackages

### Operational Cost Tracking
- Transaction fees tracked at transaction level
- Currency conversion costs excluded (handled by separate OperationalCost entity)

### SystemDiffs Integration
- All transaction field changes logged for audit trail
- User completion actions attributed to specific users

## Performance Requirements

### Webhook Processing
- Response time: < 5 seconds for TransferWise webhook acknowledgment
- Throughput: Handle up to 100 transactions per hour
- User notification: Real-time alerts within 30 seconds of transaction import

### Batch Operations
- Bulk completion: Support completion of 50+ transactions simultaneously
- Pattern recognition: Suggest FundingBatch assignments based on historical patterns
- User interface: Sub-second response times for completion forms

## Security Requirements

### Webhook Validation
- HMAC signature verification required for all TransferWise webhooks
- IP address whitelist enforcement for webhook sources
- Request rate limiting to prevent abuse

### Data Protection
- Secure credential storage for API access
- Encrypted data transmission and storage
- Comprehensive access logging for all transaction modifications

## Success Criteria

### Expected Behaviors
- 100% of TransferWise transactions automatically imported without data loss
- Users can complete transaction business fields within 2 minutes average
- Zero duplicate transactions created from same TransferWise source
- Seamless integration with FundingBatch credit updates

### Quality Metrics
- Transaction import success rate: > 99.5%
- Average completion time: < 2 minutes per transaction
- User completion rate: > 95% within 24 hours of import
- Data accuracy: 100% match with TransferWise transaction details

## Configuration Requirements

### TransferWise Integration
```
Foundation Account Mapping:
- TransferWise Profile ID → Foundation Account relationship
- Webhook endpoint URL and authentication keys
- Supported currency pairs and account configurations
```

### Transaction Templates
```
Template Definitions:
- Cash transactions: Zero fees, same-currency transfers
- M-Pesa Direct: KES destination currency, standard fee structure
- Custom transactions: Full manual field specification
```

---

*This automation bridges TransferWise's real-time transaction data with the foundation's business logic requirements, ensuring complete data capture while preserving human control over funding assignments.*
