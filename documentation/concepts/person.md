# Person

## What It Is
An individual person who interacts with the foundation as a recipient, family member, staff member, or other stakeholder, storing only current-state information.

## Purpose
To maintain current demographic and role information for individuals involved with the foundation, enabling accurate recipient tracking, relationship management, and operational coordination.

## Why it is needed 
Person records serve as the foundation for recipient identification in the allocation system while maintaining clean current-state data. Historical tracking is handled through the SystemDiffs infrastructure, allowing the Person app to focus on operational needs without time-bound field clutter.

## Key Relationships
- Parent: None (atomic entity)
- Children: None (atomic entity)
- Links to: Need records (as recipients), Allocation records (as recipients), School records (students/staff), Group records (family members), SystemDiffs (historical changes)

## Fields

### User Input Fields
- **Name** (Text): Full name of the person
- **Type** (Single Select): Student, Teacher, Non-teaching staff, Contractor, Caregiver, Other
- **Grade/Year** (Single Select): Current academic level - PP1, PP2, Grade 1, Grade 2, Grade 3, Grade 4, Grade 5, Grade 6, Grade 7, Grade 8, Grade 9, Grade 10, Grade 11, Grade 12, University Year 1, University Year 2, University Year 3, University Year 4, TVET Year 1, TVET Year 2, TVET Year 3, Completed/Working, Other (relevant for students only)
- **Priority** (Single Select): High, Medium, Low
- **School** (Relationship): Current school affiliation (schools app) 
- **Gender** (Single Select): M, F, Other/NA
- **Date of Birth** (Date): Birth date
- **Description** (Text Area): General notes and context about the person
- **Family** (Relationship): Family group affiliation (groups app) 
- **Phone Number** (Text): Current contact number
- **Requires Boarding** (Single Select): Yes, No (relevant for students only)
- **Yearly Expense** (Money): Estimated annual cost for this person's needs

### Mixed Fields
- **Status** (Single Select): In touch, Lost touch
  - **Automation**: Calculated based on recent Need creation, Allocation activity, or direct contact logging within last 6 months
  - **Manual override**: User can change status when operational knowledge contradicts automation (e.g., recent phone contact not captured in system)
  - **Override logging**: Changes recorded in System Messages for audit trail
  - **Constraints**: Cannot set to "In touch" if no activity in last 12 months without providing justification in override

### Strictly Automated Fields
None in Person app - financial and historical calculations handled by other apps

### Buttons/Bulk Actions
None required for Person app

### System Fields
- **Created Date** (Date): When this person record was created
- **Last Updated** (Date): Most recent modification
- **System Messages** (Text Area): Automated log of system actions, status changes, and manual overrides
  - **Automation examples**: "Status auto-updated to 'Lost touch' - no activity for 6+ months", "Status auto-updated to 'In touch' - new Need created"
  - **Manual override examples**: "Status manually changed from 'Lost touch' to 'In touch' by [User] - Reason: Recent phone contact confirmed", "Priority manually updated from Medium to High by [User] - Reason: Urgent medical need identified"

## Rules
- Person Name should be unique when provided to prevent duplicate records
- Type determines which fields are relevant (Grade/Year only for students, Requires Boarding only for students)
- All fields are optional to allow gradual data completion
- Historical data (support received, previous grades, status changes) tracked through SystemDiffs, not stored in Person app
- For audit purposes, Person records should be marked as obsolete/cancelled rather than deleted to preserve financial history
- Status can be auto-calculated based on recent Need/Allocation activity but manually overrideable
- Schema changes (adding/removing fields, changing choice options) require careful coordination with SystemDiffs tracking (see Schema Evolution decision document)

## Examples
1. **Current Student**: Name: "John Mwangi", Type: Student, Grade/Year: Form 3, School: St. Andrews Primary, Status: In touch, Priority: High, Requires Boarding: Yes, Yearly Expense: 45,000 KES

2. **Teacher**: Name: "Mary Wanjiku", Type: Teacher, School: St. Andrews Primary, Status: In touch, Priority: Medium, Phone Number: "+254 700 123456", Description: "Head teacher, primary contact for school coordination"