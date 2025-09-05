# Group Distribution and Membership Snapshot Decision Document

*Decision Date: Friday, September 5, 2025*

## Problem Statement

Groups serve two fundamentally different purposes in donation tracking: (1) final recipients that receive funds as a collective entity, and (2) convenience tools for creating individual recipient records. The system needs to handle both patterns while avoiding complex forwarding logic and membership change complications.

## Current Challenge Example

**Scenario**: NeedPackage created for "Class B Girls" transport needs
1. User selects group "Class B Girls" (10 members) for equal distribution
2. System creates 10 individual Needs (1,000 KES each)
3. Two months later: Alice transfers to different school, leaves "Class B Girls" group
4. **Problem**: Alice's Need still references a group she no longer belongs to

## Decision: Group Property + Membership Snapshot

### Core Design Principles

#### **1. Groups Have Distribution Property**
Each Group has a "Final Receiver" property:
- **True**: Group always receives as collective entity (families, organizations)
- **False**: Group used for individual distribution only (student classes, member lists)

#### **2. Snapshot Membership for Individual Distribution**
When Group with "Final Receiver = False" used for equal distribution:
- System captures group membership at moment of NeedPackage creation
- Creates individual Need records for each member at that snapshot
- Group membership can change independently afterward
- No ongoing relationship between Needs and Group

#### **3. Audit Trail Preservation**
System records the source group information for transparency:
- Need record includes note: "Created via group 'Class B Girls' on 2025-09-05"
- Enables audit trail even when group membership changes
- Maintains accountability without complex versioning

### Implementation Logic

#### **Group Types and Behavior**
```
Family Groups (Final Receiver = True):
- "Smith Family" → Always creates single Need for family unit
- Funding goes to family collectively
- No individual member breakdown

Student Groups (Final Receiver = False):
- "Class B Girls" → Creates individual Needs for each member
- Equal distribution: amount/member_count per person
- Members tracked individually for accountability
```

#### **NeedPackage Creation Workflow**
```
1. User selects distribution type and recipient
2. If "Individual-Equal" + Group selected:
   - Check Group.final_receiver property
   - If False: Snapshot current members, create individual Needs
   - If True: Error - cannot use final receiver group for individual distribution
3. If "Group-Final" + Group selected:
   - Check Group.final_receiver property
   - If True: Create single Need for group
   - If False: Error - cannot use individual group for final distribution
```

#### **Automation Process**
```python
def create_needs_from_group(needpackage, group, distribution_type, total_amount):
    if distribution_type == "Individual-Equal":
        if not group.final_receiver:
            # Snapshot current membership
            members = get_group_members(group.id)
            snapshot_date = datetime.now()
            amount_per_member = total_amount / len(members)
            
            for member in members:
                need = create_need(
                    needpackage_id=needpackage.id,
                    recipient=member,
                    amount=amount_per_member
                )
                add_audit_note(need.id, 
                    f"Created via group '{group.name}' on {snapshot_date}")
        else:
            raise ValueError("Cannot use final receiver group for individual distribution")
    
    elif distribution_type == "Group-Final":
        if group.final_receiver:
            create_need(
                needpackage_id=needpackage.id,
                recipient=group,
                amount=total_amount
            )
        else:
            raise ValueError("Cannot use individual group for final distribution")
```

### Distribution Type Matrix

| Group Type | Individual-Equal | Individual-Custom | Group-Final |
|------------|------------------|-------------------|-------------|
| Final Receiver = True | ❌ Error | ❌ Error | ✅ Single Need |
| Final Receiver = False | ✅ Individual Needs | N/A (use spreadsheet) | ❌ Error |

### Real-World Examples

#### **Example 1: Student Transport (Individual Distribution)**
```
Group: "Class B Girls" (Final Receiver = False)
Members at creation: Alice, Bob, Carol
NeedPackage: "March Transport" - Individual-Equal distribution
Result: 
- Need: Alice Transport (1,000 KES) - "Created via group 'Class B Girls' on 2025-09-05"
- Need: Bob Transport (1,000 KES) - "Created via group 'Class B Girls' on 2025-09-05"  
- Need: Carol Transport (1,000 KES) - "Created via group 'Class B Girls' on 2025-09-05"

Later: Alice transfers schools, leaves group
Impact: Alice's Need record unchanged, audit trail preserved
```

#### **Example 2: Family House Repair (Final Distribution)**
```
Group: "Smith Family" (Final Receiver = True)  
NeedPackage: "House Roof Repair" - Group-Final distribution
Result:
- Need: Smith Family House Repair (50,000 KES)

Later: Family member moves away
Impact: Need remains with family unit, no individual tracking affected
```

#### **Example 3: Mixed Use Group**
```
Group: "Class B Girls" (Final Receiver = False)
Use Case 1: Equal transport costs → Individual Needs created
Use Case 2: Varied book costs → Use spreadsheet (bypass group entirely)
Use Case 3: Shared classroom equipment → Cannot use this group (would need different group with Final Receiver = True)
```

## Benefits of This Approach

### **Operational Clarity**
- Clear distinction between "group as entity" vs "group as convenience tool"
- No ambiguous forwarding or container status inheritance
- User intent declared explicitly at NeedPackage creation

### **Data Integrity**
- No broken references when group membership changes
- Perfect audit trail through snapshot notes
- Individual accountability maintained regardless of group changes

### **System Simplicity**
- No complex versioning or membership tracking
- Groups remain simple reference entities
- Need records are self-contained with clear recipient links

## Alternative Approaches Rejected

### **Container/Forwarding Model**
Groups forward funds to members with dynamic status inheritance
**Rejected**: Complex status calculations, broken links when membership changes

### **Group Versioning**
Track membership changes over time with version history
**Rejected**: Implementation complexity in Podio, unclear version references

### **Context-Dependent Distribution**
Distribution type determined per NeedPackage regardless of group properties  
**Rejected**: Inconsistent group behavior, user confusion about group purpose

## Implementation Considerations

### **User Interface Requirements**
- Clear indicators of group "Final Receiver" property during selection
- Validation messages when incompatible group/distribution combinations attempted
- Visible audit notes showing group source for individual Needs

### **Data Migration**
- Existing groups need "Final Receiver" property assignment
- Historical group-based Needs may need audit note backfilling
- Review current group usage patterns to assign appropriate properties

### **User Training**
- Education on when to use final vs individual distribution groups
- Best practices for group creation and naming
- Understanding of snapshot behavior and membership independence

## Success Criteria

- Users can efficiently create both collective and individual recipient patterns
- Group membership changes do not break existing Need records
- Complete audit trail maintained for all distribution decisions
- Clear, predictable group behavior eliminates user confusion about distribution outcomes

This approach transforms groups from complex containers into simple, predictable tools that serve specific distribution patterns while maintaining data integrity and operational efficiency.