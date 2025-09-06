# Reallocation Mechanism Decision Document

*Decision Date: Friday, September 5, 2025*

## Problem Statement

When [FundingBatch](../concepts/fundingbatch.md) auto-allocation creates exceptions ([Need](../concepts/need.md) amounts change, [Need](../concepts/need.md)s marked obsolete, etc.), users need a way to trigger fresh allocation calculations without losing existing work. The system must provide a clear mechanism for users to recalculate allocations while working within Podio's UI constraints.

## Podio UI Constraints

Podio does not provide true "button" functionality. Available trigger mechanisms:

- **Category/Dropdown fields**: Change triggers webhook, can auto-reset
- **Checkbox fields**: Check/uncheck triggers webhook  
- **Text fields**: Less intuitive, prone to user error
- **No direct button functionality**: Must simulate button behavior

## Decision: Dropdown-Based Recalculation Trigger

### Implementation Design

#### **FundingBatch Fields**
```
- Allocation Mode (Category): "Auto" | "Manual"
- Recalculate Allocations (Category): "None" | "Recalculate Now" 
- System Messages (Text): Append-only exception log
- Linked NeedPackages (App): Multiple references to NeedPackages
```

#### **User Workflow**
1. User encounters allocation exception (via System Messages field)
2. User changes "Recalculate Allocations" from "None" to "Recalculate Now"
3. Automation processes request and provides feedback
4. Field automatically resets to "None" for future use

### Detailed Automation Logic

#### **Webhook Trigger Process**
```python
def handle_recalculate_request(funding_batch_id):
    # Step 1: Delete existing allocations
    existing_allocations = get_allocations_for_batch(funding_batch_id)
    for allocation in existing_allocations:
        delete_allocation(allocation.id)
    
    # Step 2: Get current linked needs
    linked_needs = get_needs_from_linked_packages(funding_batch_id)
    
    # Step 3: Validate amounts
    total_need_amount = sum(need.amount for need in linked_needs)
    batch_amount = get_funding_batch_amount(funding_batch_id)
    
    # Step 4: Execute appropriate action
    if total_need_amount == batch_amount:
        # Create new auto-allocations
        for need in linked_needs:
            create_allocation(funding_batch_id, need.id, need.amount)
        
        set_allocation_mode(funding_batch_id, "Auto")
        append_system_message(funding_batch_id, 
            f"Recalculated {len(linked_needs)} allocations - amounts match exactly")
    
    else:
        # Cannot auto-allocate
        set_allocation_mode(funding_batch_id, "Manual")
        append_system_message(funding_batch_id,
            f"Cannot auto-allocate - Need total ({total_need_amount:,}) vs Batch amount ({batch_amount:,}) - Manual allocation required")
    
    # Step 5: Reset trigger field
    update_field(funding_batch_id, "recalculate_allocations", "None")
```

#### **Recalculation Scenarios**

**Scenario 1: Amounts Now Match**
- Original: 50 Needs totaling 520K, FundingBatch 500K → Manual mode
- User adjusts Need amounts to total 500K exactly
- User triggers recalculation
- Automation creates 50 new Allocations, switches to Auto mode

**Scenario 2: Amounts Still Don't Match**  
- Need total: 480K, FundingBatch: 500K
- User triggers recalculation hoping for auto-allocation
- Automation detects mismatch, stays in Manual mode
- System Messages explains the remaining 20K difference

**Scenario 3: Needs Structure Changed**
- Original 50 Needs, now 48 Needs (2 marked obsolete)
- User triggers recalculation
- Automation creates 48 new Allocations for remaining valid Needs
- 20K credit now available for other uses

### User Experience Features

#### **Clear Feedback Loop**
- **Immediate response**: System Messages updates within seconds
- **Status visibility**: Allocation Mode shows current state
- **Action availability**: Recalculate option always available
- **Reset behavior**: Field returns to "None" for repeated use

#### **Error Prevention**
- **Destructive action warning**: Could include confirmation step for large batches
- **Undo capability**: Previous allocations are deleted, but operation can be repeated
- **Manual override**: User can always switch to Manual mode to avoid auto-allocation

#### **System Messages Examples**
```
2025-09-05 15:30: Recalculation requested - Deleted 50 existing allocations, created 48 new allocations for valid Needs - 20,000 KES credit now available
2025-09-05 15:45: Recalculation failed - Need total (480,000) vs Batch amount (500,000) - Manual allocation required for 20,000 difference
2025-09-05 16:15: Recalculation successful - All 45 Needs allocated exactly, switched to Auto mode
```

## Alternative Approaches Considered

### **Option 1: Mode Toggle Recalculation**
Switch Allocation Mode from "Manual" back to "Auto" triggers recalculation
**Rejected**: Confuses mode state with action triggers, less clear user intent

### **Option 2: Checkbox Trigger**
"Recalculate Allocations" checkbox that auto-unchecks after processing
**Rejected**: Less intuitive than dropdown, checkbox semantics unclear

### **Option 3: Separate Recalculation App**
Create separate App for allocation management with dedicated recalculation records
**Rejected**: Adds unnecessary complexity, reduces user workflow efficiency

## Implementation Considerations

### **Technical Requirements**
- **Webhook reliability**: Critical that field reset happens after processing
- **Error handling**: If automation fails, field should not reset to avoid losing user intent
- **Concurrency**: Handle multiple simultaneous recalculation requests safely

### **User Training**
- **Destructive nature**: Users need to understand existing allocations will be deleted
- **Timing considerations**: Best practices for when to recalculate vs manual adjustment
- **Exception interpretation**: How to read System Messages and respond appropriately

### **Performance Impact**
- **Large batches**: Deleting and recreating 100+ allocations may take time
- **Rate limits**: Bulk operations must respect Podio API constraints
- **User feedback**: Progress indication for long-running recalculations

## Success Criteria

- **User empowerment**: Users can resolve allocation exceptions without technical support
- **Clear communication**: System provides actionable feedback about recalculation results
- **Operational efficiency**: Recalculation significantly faster than manual recreation
- **Data integrity**: No orphaned allocations or amount inconsistencies after recalculation

This dropdown-based approach provides the most intuitive user experience within Podio's constraints while maintaining clear audit trails and proper exception handling for allocation management.