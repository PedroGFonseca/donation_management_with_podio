# Auto-Allocation Decision Document

*Decision Date: September 2025*

## Problem Statement

When a [FundingBatch](../concepts/fundingbatch.md) is created for a [NeedPackage](../concepts/needpackage.md) containing many individual [Need](../concepts/need.md)s (e.g., 50 students requiring transport with varying amounts), the current system would require users to manually create 50 separate [Allocation](../concepts/allocation.md) records. This defeats the operational efficiency gained from spreadsheet-based [Need](../concepts/need.md) creation.

## Motivation Example

**Scenario**: Medium-distance student transport
1. User creates spreadsheet with 50 students, each with different transport costs ranging from 8,000-12,000 KES, totaling 500,000 KES
2. Automation reads spreadsheet and creates:
   - [NeedPackage](../concepts/needpackage.md) "MediumDistanceTransportMarch2025" 
   - 50 individual [Need](../concepts/need.md) records with specific amounts for each student
3. User creates [FundingBatch](../concepts/fundingbatch.md) "RegularOperationsMarch2025" (500,000 KES) and links it to the transport [NeedPackage](../concepts/needpackage.md)
4. **Current problem**: User must now manually create 50 [Allocation](../concepts/allocation.md) records linking the [FundingBatch](../concepts/fundingbatch.md) to each individual [Need](../concepts/need.md)

**Without auto-allocation**: User performs 50+ manual operations, negating spreadsheet efficiency
**With auto-allocation**: System automatically creates all 50 Allocations, user reviews and confirms

## Decision: Auto-Allocation with Manual Override

### Core Logic
**When**: [FundingBatch](../concepts/fundingbatch.md) is linked to [NeedPackage](../concepts/needpackage.md)(s) AND total amounts match
**Action**: Automation automatically creates [Allocation](../concepts/allocation.md) records for all linked [Need](../concepts/need.md)s
**Safeguard**: Switch to manual mode when exceptions occur

### Detailed Workflow

#### Auto-Allocation Trigger
1. User creates FundingBatch and links to one or more NeedPackages
2. Automation calculates: `sum(linked Need amounts) vs FundingBatch amount`
3. **If amounts match exactly**: Auto-create Allocation records proportionally
4. **If amounts don't match**: Leave in manual mode, require user decision

#### Allocation Creation
- **Allocation naming**: Follow pattern "FundingBatchName_NeedName"
- **Amount assignment**: Each Allocation gets the full amount of its corresponding Need
- **Status**: All created Allocations start as "Pending"

#### Exception Triggers (Switch to Manual Mode)
Auto-allocation mode switches to "Manual" when:
1. **Need amount changes** after Allocations created
2. **Need marked obsolete** or status changed to exclude from funding
3. **New Need added** to linked NeedPackage after auto-allocation
4. **User manually modifies** any auto-created Allocation

#### Manual Mode Behavior
- **Existing Allocations**: Remain but can be freely modified by user
- **New Allocations**: Must be created manually
- **Status indicator**: FundingBatch shows "Manual Allocation Mode" 
- **Re-enable auto**: User can reset to auto-allocation (clears existing Allocations)

### Implementation Specifics

#### FundingBatch Fields
- **Allocation Mode** (Category): "Auto" | "Manual"
- **Linked NeedPackages** (App): Multiple references to NeedPackages
- **Auto-Allocation Status** (Calculated): "Complete" | "Pending" | "Exception"

#### Automation Triggers
- **FundingBatch creation/update**: Check if auto-allocation conditions met
- **Need amount changes**: Check if linked FundingBatch needs manual review
- **Need status changes**: Check if obsolete Needs affect allocation

#### User Controls
- **Force manual mode**: User can override auto-allocation at any time
- **Recalculate allocations**: User can trigger fresh auto-allocation (replaces existing)
- **Review exceptions**: Clear indication when manual intervention needed

## Benefits

### Operational Efficiency
- **Eliminates repetitive work**: No manual creation of 50+ similar Allocation records
- **Preserves spreadsheet workflow**: Maintains efficiency of bulk Need creation
- **Reduces errors**: Automated calculation prevents manual allocation mistakes

### Maintains Control
- **Exception detection**: System flags when auto-allocation assumptions break
- **Manual override**: User can take control when business logic requires it
- **Audit trail**: Clear record of automatic vs manual allocation decisions

### Financial Integrity
- **Amount consistency**: Auto-allocation ensures FundingBatch amount equals sum of Allocations
- **No overcommitment**: System prevents allocating more than available credit
- **Exception safety**: Manual mode triggers prevent invalid states

## Risks and Mitigations

### Risk: Premature Auto-Allocation
**Scenario**: User creates FundingBatch before all Needs are finalized
**Mitigation**: Auto-allocation only triggers when user explicitly links NeedPackages

### Risk: Complex Exception Handling
**Scenario**: Multiple Needs change simultaneously, creating cascading exceptions
**Mitigation**: Switch to manual mode preserves all existing work, user resolves at their pace

### Risk: User Confusion About Mode
**Scenario**: User doesn't understand why system switched to manual mode
**Mitigation**: Clear status indicators and explanatory messages for mode changes

## Alternative Considered: Always Manual Allocation

**Rejected because**: Creates operational burden that scales poorly with NeedPackage size. A 100-student transport NeedPackage would require 100+ manual Allocation entries, making the system impractical for bulk operations.

## Implementation Priority

**Phase 1**: Basic auto-allocation when amounts match exactly
**Phase 2**: Exception detection and manual mode switching  
**Phase 3**: User controls for overriding and recalculating allocations

This decision enables efficient bulk operations while maintaining the precision and control required for financial accountability.