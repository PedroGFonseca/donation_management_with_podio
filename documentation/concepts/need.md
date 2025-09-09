# Need

## What It Is
An individual-level funding requirement for a specific recipient with a precise amount, representing the atomic unit of funding accountability within the foundation system.

## Purpose
To provide granular tracking of individual funding requirements while maintaining perfect accountability for every recipient and amount within group requests or individual applications.

## Why it is needed 
The [Need](need.md) entity enables both clean UI management (through [NeedPackage](needpackage.md)s) and detailed individual accountability. It ensures every recipient and amount is precisely tracked while supporting explicit funding allocation through the [Allocation](allocation.md) model.

## Key Relationships
- **References**: Person/School/Group (recipient field), NeedPackage (needpackage field)
- **Referenced by**: Allocation records (need field in Allocation app)

## Fields

### User Input Fields
- **Amount** (Money): Exact amount needed for this specific recipient
- **Recipient** ([Person](person.md)/[School](school.md)/[Group](group.md)): The specific individual or entity who will receive this funding
- **Priority** (Single Select): Low, Medium, High, Urgent
- **Type** (Single Select): Education, Medical, Food, Transport, Emergency, Infrastructure, Other
- **Title** (Text): Descriptive title for this specific Need (auto-generated if from template)
- **Description** (Text Area): Detailed description of the specific requirement
- **Currency** (Single Select): Currency for the amount (defaults to foundation's default currency)
- **Deadline** (Date): When funding is needed by
- **Notes** (Text Area): Additional context, special circumstances, or operational notes
- **Relevant Links** (Link): Supporting documentation, photos, or external references
- **Covered By** ([Person](person.md)/Organization): Specific donor attribution when funding is donor-designated

### Mixed Fields
- **Status** (Single Select): New, Accepted, Declined, Allocated, Paid, Failed, Cancelled
  - **Automation**: "Allocated" when allocations exist, "Paid" when sum(successful allocations) ≥ Need amount
  - **Manual override**: User can change to "Declined" (recipient unavailable), "Cancelled" (circumstances changed), or "Failed" (funding unsuccessful)
  - **Override scenarios**: Real-world exceptions where calculation doesn't reflect operational reality
  - **Constraint**: Cannot manually set to "Paid" unless allocations actually cover the amount

### Strictly Automated Fields
- **Allocated Amount** (Calculated Money): Sum of all successful [Allocation](allocation.md) amounts for this Need
- **Remaining Amount** (Calculated Money): Amount - Allocated Amount

### System Fields
- **System Messages** (Text Area): Automation log for status changes, allocation impacts, and amount change effects
  - Example: "2025-09-06 16:20: Status change - Allocation sum (15,000) reached Need amount, status changed to Paid"
  - Example: "2025-09-06 17:10: Amount change - Need amount increased from 10,000 to 15,000, FundingBatch switched to Manual mode"
  - Example: "2025-09-06 18:30: Manual override - Status changed to Declined due to recipient unavailability"
- **NeedPackage ID** (Relationship): Links to parent [NeedPackage](needpackage.md)
- **Template Reference** (Relationship): Links to [NeedTemplate](needtemplate.md) if generated from template
- **Created Date** (Date): When this Need was created
- **Last Updated** (Date): Most recent modification

## Rules
- [Need](need.md) must have exactly one specific recipient ([Person](person.md), [School](school.md), or [Group](group.md))
- Need must have exactly one specific Type - no mixed reasons at the individual recipient level (e.g., if John needs both school fees and uniforms, create two separate Needs: "John, Education, 50,000 KES" and "John, Transport, 15,000 KES")
- Amount must be positive and in valid currency
- Status automatically calculated based on allocation status: "Allocated" when allocations exist, "Paid" when sum(successful allocations) >= Need amount
- Status can also be updated via parent NeedPackage bulk actions (e.g., "Mark all Accepted Needs as Paid")
- Recipient type must match NeedPackage distribution rules
- Currency must match parent NeedPackage currency
- One [Need](need.md) can have multiple [Allocation](allocation.md)s from different [FundingBatch](fundingbatch.md)es (cross-batch funding)
- Allocation amounts cannot exceed Need amount
- For audit purposes, Needs should be marked as obsolete/cancelled rather than deleted to preserve financial history

## Examples
1. **Individual Student Transport**: Amount: 4,500 KES, Recipient: John Mwangi ([Person](person.md)), Description: "Holiday transport from school to home village", Status: New, Type: Transport, Parent: "30 Students Holiday Transport [NeedPackage](needpackage.md)"

2. **School Infrastructure**: Amount: 250,000 KES, Recipient: St. Andrews Primary ([School](school.md)), Description: "Repair of classroom roof damaged in recent storms", Status: Allocated, Type: Infrastructure, Covered By: Anonymous Donor, Priority: High