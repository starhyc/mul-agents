---
name: PRD Optimization
description: Optimize an existing Product Requirements Document (PRD) through structured client dialogue to identify missing features, problematic requirements, and areas for improvement.
version: 0.1.0
---

# PRD Optimization

## Purpose

Review and optimize existing Product Requirements Documents through targeted client dialogue. Identify gaps in functionality, conflicting requirements, unclear specifications, and potential issues before implementation begins.

## When to Use This Skill

Use this skill when:
- An existing PRD needs review and refinement
- Client wants to validate completeness of requirements
- Need to identify missing edge cases or scenarios
- Requirements seem unclear or conflicting
- Before starting implementation to catch issues early

## Optimization Framework

### 1. Initial PRD Analysis

Read and analyze the existing PRD to understand:
- Product vision and goals
- Target users and use cases
- Core features and functionality
- Technical architecture
- Success metrics

**Identify potential issues:**
- Missing user scenarios
- Unclear or ambiguous requirements
- Conflicting specifications
- Missing non-functional requirements
- Incomplete user flows
- Missing error handling scenarios
- Security or privacy gaps
- Scalability concerns not addressed

### 2. Structured Dialogue Areas

#### 2.1 Functionality Gaps

**Questions to explore:**
- "I see the PRD covers [feature X]. What should happen when [edge case Y]?"
- "For [user role], I notice we haven't defined how they handle [scenario Z]. Can you clarify?"
- "What happens if [external dependency] fails or is unavailable?"
- "Are there any batch operations or bulk actions users might need?"
- "How should the system handle concurrent operations or race conditions?"

**Common missing features:**
- Error handling and recovery
- Undo/redo functionality
- Bulk operations
- Import/export capabilities
- Audit logging
- Notification preferences
- Search and filtering
- Data validation rules
- Rate limiting
- Offline mode or degraded functionality

#### 2.2 User Experience Gaps

**Questions to explore:**
- "How should users be notified when [event X] occurs?"
- "What feedback does the user get during long-running operations?"
- "How do users recover from errors or mistakes?"
- "What happens on first-time user experience vs returning users?"
- "Are there any accessibility requirements we should consider?"

**Common UX gaps:**
- Loading states and progress indicators
- Empty states (no data scenarios)
- Error messages and guidance
- Confirmation dialogs for destructive actions
- Keyboard shortcuts or accessibility features
- Mobile responsiveness requirements
- Internationalization needs

#### 2.3 Data and Integration Gaps

**Questions to explore:**
- "How should data be migrated from existing systems?"
- "What data retention policies should we implement?"
- "Are there any data export or backup requirements?"
- "How should the system integrate with [external system]?"
- "What happens to data when users delete their accounts?"

**Common data gaps:**
- Data migration strategy
- Backup and disaster recovery
- Data retention and archival
- GDPR/privacy compliance
- Data validation and sanitization
- API versioning strategy

#### 2.4 Security and Permissions

**Questions to explore:**
- "Who can access [sensitive feature/data]?"
- "How should we handle authentication failures or suspicious activity?"
- "Are there any compliance requirements (GDPR, HIPAA, etc.)?"
- "What audit logging is needed for security or compliance?"
- "How should API keys or secrets be managed?"

**Common security gaps:**
- Role-based access control details
- API authentication and authorization
- Rate limiting and abuse prevention
- Sensitive data encryption
- Session management
- Password policies
- Two-factor authentication

#### 2.5 Performance and Scalability

**Questions to explore:**
- "What's the expected data volume in 1 year? 5 years?"
- "How many concurrent users should the system support?"
- "Are there any specific performance requirements for [critical operation]?"
- "What happens when the system is under heavy load?"
- "Are there any caching strategies we should consider?"

**Common performance gaps:**
- Specific response time requirements
- Concurrent user limits
- Data volume projections
- Caching strategy
- Database indexing needs
- CDN requirements

#### 2.6 Operations and Maintenance

**Questions to explore:**
- "How will the system be monitored in production?"
- "What alerts should be configured for critical issues?"
- "How will database migrations be handled?"
- "What's the deployment and rollback strategy?"
- "How will configuration changes be managed?"

**Common operational gaps:**
- Monitoring and alerting requirements
- Logging strategy
- Deployment process
- Database migration strategy
- Configuration management
- Backup and recovery procedures

### 3. Conflict and Inconsistency Detection

**Look for:**
- Features that contradict each other
- Inconsistent terminology or naming
- Conflicting user flows
- Technical choices that don't align with requirements
- Timeline expectations vs scope reality
- Resource constraints vs feature complexity

**Questions to ask:**
- "I notice [requirement A] says X, but [requirement B] implies Y. Which is correct?"
- "The architecture shows [technology A], but the requirements need [capability B] which isn't supported. How should we handle this?"
- "The timeline shows [X weeks] but the scope includes [large feature set]. Should we prioritize or extend timeline?"

### 4. Clarity and Specificity Review

**Identify vague requirements:**
- "Fast response time" → What's the specific SLA?
- "User-friendly interface" → What are the specific UX requirements?
- "Scalable system" → What are the specific scale targets?
- "Secure" → What are the specific security requirements?

**Questions to ask:**
- "When you say [vague term], can you provide specific metrics or examples?"
- "What does success look like for [feature]? How will we measure it?"
- "Can you describe a specific scenario where [feature] would be used?"

### 5. Priority and Scope Validation

**Questions to explore:**
- "I see [feature X] is marked P0. Is this truly required for MVP or could it be P1?"
- "Features [A, B, C] are all P0, but they seem to have dependencies. What's the implementation order?"
- "The MVP scope seems large. If we had to cut one feature, which would it be?"
- "Are there any features marked as 'nice to have' that are actually critical?"

### 6. Documentation and Handoff

**Check for:**
- Clear acceptance criteria for each feature
- Mockups or wireframes for UI features
- API specifications for integrations
- Data models and schemas
- Error codes and messages
- User documentation needs

**Questions to ask:**
- "Do we have mockups for [UI feature]?"
- "What should the API contract look like for [integration]?"
- "What error messages should users see for [error scenario]?"
- "What documentation will users need?"

## Dialogue Techniques

### Progressive Questioning

Start broad, then drill into specifics:
1. "Tell me about the [feature area]"
2. "What happens when [specific scenario]?"
3. "How should the system handle [edge case]?"
4. "What if [failure condition]?"

### Scenario-Based Validation

Present concrete scenarios:
- "Imagine a user who [scenario]. Walk me through what happens step by step."
- "What if a user tries to [unexpected action]?"
- "Consider a power user who needs to [advanced scenario]. How would they accomplish this?"

### Challenge Assumptions

Respectfully question assumptions:
- "I notice we're assuming [X]. What if that's not true?"
- "This design works well for [scenario A], but what about [scenario B]?"
- "Have we considered users who [different context]?"

### Prioritization Forcing

Help clarify priorities:
- "If we could only build 3 features for launch, which would they be?"
- "What's the minimum functionality needed for users to get value?"
- "Which features would cause the most pain if missing?"

## Output Format

After dialogue, provide a structured summary:

### Issues Identified

**Missing Features:**
- [Feature description] - [Why it's needed] - [Suggested priority]

**Unclear Requirements:**
- [Requirement] - [What's unclear] - [Suggested clarification]

**Conflicts:**
- [Conflict description] - [Impact] - [Suggested resolution]

**Gaps:**
- [Gap area] - [What's missing] - [Recommendation]

### Recommended Changes

**High Priority:**
- [Change 1] - [Rationale]
- [Change 2] - [Rationale]

**Medium Priority:**
- [Change 3] - [Rationale]

**Low Priority / Future Consideration:**
- [Change 4] - [Rationale]

### Questions Still Outstanding

- [Question 1]
- [Question 2]

### Next Steps

- [Action item 1]
- [Action item 2]

## Best Practices

1. **Be thorough but focused** - Cover all areas but don't overwhelm with too many questions at once
2. **Use concrete examples** - Abstract questions get abstract answers
3. **Challenge respectfully** - Point out issues without being confrontational
4. **Prioritize ruthlessly** - Help client distinguish must-have from nice-to-have
5. **Document decisions** - Capture the "why" behind requirements
6. **Think like an implementer** - Ask questions developers will need answered
7. **Consider the user** - Always bring it back to user value and experience
8. **Be realistic** - Help client understand trade-offs and constraints

## Common PRD Anti-Patterns to Watch For

- **Feature creep** - Scope expanding beyond original vision
- **Gold plating** - Over-engineering for hypothetical future needs
- **Vague success metrics** - No clear way to measure success
- **Missing error handling** - Only happy path defined
- **Unrealistic timelines** - Scope doesn't match timeline
- **Technology-first thinking** - Choosing tech before understanding requirements
- **Missing user research** - Assumptions about users not validated
- **Ignoring constraints** - Not considering budget, resources, or technical limitations
