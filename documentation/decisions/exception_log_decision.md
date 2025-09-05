# Exception Log Field Decision Document

*Decision Date: September 2025*

## Problem Statement

Automation in the donation tracking system needs to communicate why it stopped working, switched modes, or detected problems. Users need visibility into system decisions to understand when manual intervention is required and why automated processes failed or changed behavior.

## Current Gaps

Without systematic exception logging:
- **Silent failures**: User doesn't know why auto-allocation stopped working
- **Lost context**: No record of what automation detected or when
- **Debugging difficulty**: Cannot trace automation decision history
- **User confusion**: System switches to manual mode without explanation

## Decision: Universal Exception Log Field

### Field Specification

#### **Field Name**: "System Messages"
- **Type**: Text (multi-line)
- **Behavior**: Append-only logging
- **Visibility**: Read-only for users, write-only for automation
- **Scope**: Added to all Apps that have automation logic

#### **Message Format**
```
YYYY-MM-DD HH:MM: [Context] - [Issue Description] - [Action Taken]
```

#### **Example Messages**
```
2025-09-15 14:30: Auto-allocation - Need amounts (520,000) exceed FundingBatch amount (500,000) - Switched to Manual mode
2025-09-16 09:15: Status inheritance - Need 'JonTransport' marked obsolete - Deallocated 15,000 from FundingBatch
2025-09-16 11:45: Bulk operation - New Need added to linked NeedPackage after auto-allocation - Manual review required
```

### Implementation Across Apps

#### **Apps Requiring Exception Log**
- **FundingBatch**: Auto-allocation exceptions, credit violations
- **NeedPackage**: Status calculation issues, template generation problems  
- **Need**: Amount changes affecting allocations, status inheritance failures
- **Allocation**: Credit limit violations, duplicate allocation attempts
- **Transaction**: Banking status conflicts, amount mismatches

#### **Apps Not Requiring Exception Log**
- **People/Schools/Groups**: Static reference data, minimal automation
- **NeedTemplate**: Template definitions, no real-time automation

### Automation Logging Rules

#### **When to Log**
1. **Mode switches**: Auto → Manual transitions
2. **Exception detection**: Constraint violations, data inconsistencies
3. **Failed operations**: Automation attempts that cannot complete
4. **Manual overrides**: User actions that affect automated processes

#### **What Not to Log**
- **Successful operations**: Normal automation flow
- **User-initiated actions**: Manual operations by design
- **Informational updates**: Routine status changes

#### **Message Content Standards**
- **Timestamp**: Always include precise date/time
- **Context**: What automation process was running
- **Issue**: Specific problem detected
- **Action**: What the system did in response
- **User impact**: What user needs to do (if anything)

### User Experience Design

#### **Field Visibility**
- **Prominent placement**: Near status fields where relevant
- **Read-only display**: Users cannot edit system messages
- **Clear labeling**: "System Messages" or "Automation Log"
- **Chronological order**: Most recent messages at top

#### **Message Clarity**
- **Plain language**: Avoid technical jargon
- **Actionable information**: Tell user what to do next
- **Context preservation**: Explain why automation made decisions
- **Severity indication**: Use consistent language for different issue levels

### Implementation Benefits

#### **Operational Transparency**
- **User understanding**: Clear communication about system behavior
- **Trust building**: Users see system is actively monitoring for problems
- **Learning opportunity**: Users understand system logic and constraints

#### **Debugging and Maintenance**
- **Issue tracking**: Historical record of automation problems
- **Pattern recognition**: Identify recurring issues for system improvement
- **Support efficiency**: Users can reference specific error messages

#### **Audit and Compliance**
- **Decision trail**: Record of why system made specific choices
- **Change documentation**: Track automation behavior over time
- **Accountability**: Clear record of system vs user actions

## Implementation Considerations

### Technical Requirements
- **Append-only mechanism**: Automation can add but not modify existing messages
- **Character limits**: Podio text fields have limits, may need message rotation
- **Webhook integration**: System messages added via API when automation detects exceptions
- **Timestamp consistency**: Use server-side timestamps, not client-side

### User Training Requirements
- **Message interpretation**: Users need to understand common message types
- **Response procedures**: What to do when specific exceptions occur
- **Escalation paths**: When to contact technical support vs handle manually

### System Performance
- **Log rotation**: Consider archiving old messages to prevent field overflow
- **Query efficiency**: Ensure logging doesn't impact system performance
- **Storage planning**: Text fields with extensive logging may grow large

## Alternative Considered: Separate Logging App

**Rejected because**: Creates additional complexity with separate App for logs, reduces visibility (users would need to navigate to different App), and complicates the user experience. In-line logging in the relevant App provides immediate context and visibility.

## Success Metrics

- **Reduced support requests**: Users can self-diagnose automation issues
- **Faster issue resolution**: Clear error messages enable quicker problem solving
- **User confidence**: Transparent system behavior increases user trust
- **System reliability**: Better tracking of automation failures enables improvements

This universal logging approach provides systematic visibility into automation decisions while maintaining clean user experience and operational transparency.