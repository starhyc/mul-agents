# PRD Review Checklist

Use this checklist to systematically review a PRD and identify gaps.

## 1. Product Vision & Goals

- [ ] Clear problem statement defined
- [ ] Target users clearly identified
- [ ] Success metrics defined and measurable
- [ ] Product scope is realistic for timeline
- [ ] Business value articulated

## 2. User Scenarios & Flows

- [ ] Primary user flows documented
- [ ] Edge cases considered
- [ ] Error scenarios defined
- [ ] First-time user experience described
- [ ] Power user scenarios covered
- [ ] Multi-user/concurrent scenarios addressed

## 3. Functional Requirements

### Core Features
- [ ] All must-have features clearly defined
- [ ] Feature priorities justified
- [ ] Dependencies between features identified
- [ ] Acceptance criteria for each feature

### Missing Common Features
- [ ] Error handling and recovery
- [ ] Undo/redo functionality
- [ ] Bulk operations
- [ ] Import/export
- [ ] Search and filtering
- [ ] Notifications
- [ ] Audit logging
- [ ] Data validation

## 4. User Experience

- [ ] Loading states defined
- [ ] Empty states defined
- [ ] Error messages specified
- [ ] Confirmation dialogs for destructive actions
- [ ] Accessibility requirements
- [ ] Mobile/responsive requirements
- [ ] Internationalization needs

## 5. Data & Integration

- [ ] Data model defined
- [ ] Data migration strategy
- [ ] Data retention policy
- [ ] Backup and recovery plan
- [ ] External integrations specified
- [ ] API contracts defined
- [ ] Data validation rules

## 6. Security & Privacy

- [ ] Authentication method defined
- [ ] Authorization/permissions model
- [ ] Sensitive data handling
- [ ] Compliance requirements (GDPR, etc.)
- [ ] API security
- [ ] Rate limiting
- [ ] Audit logging for security events

## 7. Performance & Scalability

- [ ] Response time requirements
- [ ] Concurrent user targets
- [ ] Data volume projections
- [ ] Caching strategy
- [ ] Database optimization needs
- [ ] Load handling strategy

## 8. Operations & Maintenance

- [ ] Monitoring requirements
- [ ] Alerting strategy
- [ ] Logging requirements
- [ ] Deployment process
- [ ] Rollback strategy
- [ ] Configuration management
- [ ] Database migration strategy

## 9. Technical Architecture

- [ ] Technology choices justified
- [ ] Architecture supports requirements
- [ ] Scalability considerations
- [ ] Third-party dependencies identified
- [ ] Technical risks identified

## 10. Documentation & Clarity

- [ ] Requirements are specific, not vague
- [ ] Terminology is consistent
- [ ] No conflicting requirements
- [ ] Mockups/wireframes for UI features
- [ ] API specifications for integrations
- [ ] User documentation needs identified

## Red Flags to Watch For

- ⚠️ Vague terms without specific metrics ("fast", "scalable", "user-friendly")
- ⚠️ Only happy path defined, no error handling
- ⚠️ Unrealistic scope for timeline
- ⚠️ Missing non-functional requirements
- ⚠️ No success metrics or KPIs
- ⚠️ Conflicting requirements
- ⚠️ Technology chosen before requirements understood
- ⚠️ No consideration of edge cases
- ⚠️ Missing security or privacy requirements
- ⚠️ No data migration or integration strategy
