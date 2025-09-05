# Allocation

## What It Is
An explicit assignment of money from a [FundingBatch](../fundingbatch.md) to a specific [Need](../need.md) with a precise amount.

## Purpose
Solves the many-to-many relationship complexity between funding sources and individual recipients by creating trackable records of exactly which money goes to which recipient.

## Why It Is Needed
Without Allocation records, the system cannot handle partial funding scenarios OR efficient bulk operations. 

For example: 
- if Jon's food [Need](../need.md) costs 400,000 KES but is funded by (A) 50,000 KES from [FundingBatch](../fundingbatch.md) StudentNutritionMarch2025 and (B) 350,000 KES from [FundingBatch](../fundingbatch.md) EmergencyResponseApril2025, there's no way to track this relationship. 
- Additionally, when a [NeedPackage](../needpackage.md) contains 50 students requiring transport, manually creating 50 individual funding links would be operationally impractical. 
- You'd end up with disconnected records like "Jon food part 1" and "Jon food part 2" with no clear connection, making it impossible to see that Jon actually received 400,000 KES total for food or which funding sources contributed what amounts. 
- Allocation records enable both complex funding scenarios and automated bulk operations while maintaining perfect traceability.

## Key Relationships
- Parent: [FundingBatch](../fundingbatch.md) (source of the money)
- Children: None
- Links to: [Need](../need.md) (recipient of the money)

## Fields

### User Input Fields
- **Amount** (Money): Specific amount allocated from batch to need
- **FundingBatch** (App): Source batch providing the money
- **Need** (App): Individual need receiving the allocation
- **Allocation Date** (Date): When allocation was created
- **Notes** (Text): Context for this allocation decision
- **Receipt Reference** (Text): Link to confirmation documentation

### Mixed Fields
- **Status** (Category): Pending | Successful | Failed | Cancelled - Can be auto-calculated based on funding confirmation but manually overridden for operational exceptions

### Strictly Automated Fields
- **Creation Method** (Category): "Manual" | "Auto-Generated"

### System Fields
- **ID** (Auto): Unique identifier
- **Created Date** (Auto): System creation timestamp
- **Modified Date** (Auto): Last update timestamp

## Rules
- Auto-allocation creates [Allocation](../allocation.md)s when [FundingBatch](../fundingbatch.md) is linked to [NeedPackage](../needpackage.md)s and amounts match exactly
- Recalculation deletes all existing Allocations for a FundingBatch and recreates them based on current linked Needs
- Auto-generated Allocations follow naming pattern "FundingBatchName_NeedName"
- When exceptions occur (amount mismatches, obsolete Needs), FundingBatch switches to manual allocation mode
- One allocation per FundingBatch-Need pairing (no duplicates)
- Allocation amount cannot exceed FundingBatch available credit
- Sum of all allocations for a Need determines Need status (Paid when sum ≥ Need amount)
- Allocation status can be updated via parent NeedPackage bulk actions (e.g., "Mark all Accepted Needs as Paid")
- FundingBatch.allocated_amount = sum of all allocation amounts regardless of status
- Money becomes fungible within FundingBatch - doesn't matter which specific transaction funded the allocation
- For audit purposes, Allocations should be marked as obsolete/cancelled rather than deleted to preserve financial history

## Examples

**Example 1 - Cross-batch funding**:
Jon's food Need costs 400,000 KES but gets funded from two sources:
1. User creates Allocation "StudentNutritionMarch2025_JonFoodNeed" linking FundingBatch StudentNutritionMarch2025 (50,000 KES) to Jon's food Need
2. Later, user creates Allocation "EmergencyResponseApril2025_JonFoodNeed" linking FundingBatch EmergencyResponseApril2025 (350,000 KES) to the same food Need  
3. Automation calculates that sum of Allocations (400,000 KES) equals Jon's food Need amount and switches the Status field in Jon's Need record to "Paid"

**Example 2 - Auto-allocation with exception handling**:
Bulk transport scenario with complications:
1. User creates [NeedPackage](../needpackage.md) "MediumDistanceTransportMarch2025" with 50 individual student [Need](../need.md)s totaling 500,000 KES
2. User creates FundingBatch "RegularOperationsMarch2025" (500,000 KES) and links it to the transport NeedPackage
3. Automation detects amounts match exactly and auto-creates 50 Allocation records like "RegularOperationsMarch2025_MaryTransportNeed", "RegularOperationsMarch2025_PeterTransportNeed"
4. User discovers one student's transport cost increased from 10,000 to 15,000 KES
5. Automation detects amount mismatch (Need total now 505,000 vs FundingBatch 500,000) and switches FundingBatch Allocation Mode field to "Manual"
6. Automation appends to System Messages field: "Need amounts (505,000) exceed FundingBatch amount (500,000) - Manual allocation required"
7. User changes Recalculate Allocations field from "None" to "Recalculate Now" after adjusting amounts
8. Automation deletes existing 50 Allocations, creates fresh ones based on current Need amounts, and switches back to Auto mode if amounts now match