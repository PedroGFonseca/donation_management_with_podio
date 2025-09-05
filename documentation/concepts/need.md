# Need

## What It Is
An individual-level funding requirement for a specific recipient with a precise amount, representing the atomic unit of funding accountability within the foundation system.

## Purpose
To provide granular tracking of individual funding requirements while maintaining perfect accountability for every recipient and amount within group requests or individual applications.

## Why it is needed 
The Need entity enables both clean UI management (through NeedPackages) and detailed individual accountability. It ensures every recipient and amount is precisely tracked while supporting explicit funding allocation through the Allocation model.

## Key Relationships
- Parent: NeedPackage (the group request that contains this individual need)
- Children: None (atomic unit)
- Links to: Allocation records (explicit funding assignments), Person/School/Group (the specific recipient), NeedTemplate (if generated from template)

## Fields

### User Input Fields
- **Amount** (Money): Exact amount needed for this specific recipient
- **Recipient** (Person/School/Group): The specific individual or entity who will receive this funding
- **Priority** (Single Select): Low, Medium, High, Urgent
- **Type** (Single Select): Education, Medical, Food, Transport, Emergency, Infrastructure, Other
- **Title** (Text): Descriptive title for this specific need (auto-generated if from template)
- **Description** (Text Area): Detailed description of the specific requirement
- **Currency** (Single Select): Currency for the amount (defaults to foundation's default currency)
- **Deadline** (Date): When funding is needed by
- **Notes** (Text Area): Additional context, special circumstances, or operational notes
- **Relevant Links** (Link): Supporting documentation, photos, or external references
- **Covered By** (Person/Organization): Specific donor attribution when funding is donor-designated

### Mixed Fields
- **Status** (Single Select): New, Accepted, Declined, Allocated, Paid, Failed, Cancelled - Auto-calculated based on allocation status: "Allocated" when allocations exist, "Paid" when sum(successful allocations) >= need amount, but can be manually overridden for operational exceptions

### Strictly Automated Fields
- **Allocated Amount** (Calculated Money): Sum of all successful Allocation amounts for this need
- **Remaining Amount** (Calculated Money): Amount - Allocated Amount

### System Fields
- **NeedPackage ID** (Relationship): Links to parent NeedPackage
- **Template Reference** (Relationship): Links to NeedTemplate if generated from template
- **Created Date** (Date): When this need was created
- **Last Updated** (Date): Most recent modification

## Rules
- Need must have exactly one specific recipient (Person, School, or Group)
- Need must have exactly one specific Type - no mixed reasons at the individual recipient level (e.g., if John needs both school fees and uniforms, create two separate Needs: "John, Education, 50,000 KES" and "John, Transport, 15,000 KES")
- Amount must be positive and in valid currency
- Status automatically calculated based on allocation status: "Allocated" when allocations exist, "Paid" when sum(successful allocations) >= need amount
- Status can also be updated via parent NeedPackage bulk actions (e.g., "Mark all Accepted Needs as Paid")
- Recipient type must match NeedPackage distribution rules
- Currency must match parent NeedPackage currency
- One Need can have multiple Allocations from different FundingBatches (cross-batch funding)
- Allocation amounts cannot exceed Need amount
- For audit purposes, Needs should be marked as obsolete/cancelled rather than deleted to preserve financial history

## Examples
1. **Individual Student Transport**: Amount: 4,500 KES, Recipient: John Mwangi (Person), Description: "Holiday transport from school to home village", Status: New, Type: Transport, Parent: "30 Students Holiday Transport Package"

2. **School Infrastructure**: Amount: 250,000 KES, Recipient: St. Andrews Primary (School), Description: "Repair of classroom roof damaged in recent storms", Status: Allocated, Type: Infrastructure, Covered By: Anonymous Donor, Priority: High