# PRD Optimization Question Bank

Organized questions to help identify gaps and issues in PRDs.

## Functionality & Features

### Core Features
- "What are the 3 most critical features users absolutely need?"
- "If we could only build one feature for launch, which would it be?"
- "What happens if [feature X] fails or is unavailable?"
- "How should the system behave when [edge case]?"

### Missing Features
- "Do users need to perform bulk operations? (e.g., bulk delete, bulk update)"
- "Should users be able to undo actions? Which actions?"
- "Do users need to import/export data? In what formats?"
- "Is there a need for scheduled or automated tasks?"
- "Should users be able to save drafts or work-in-progress?"
- "Do users need to share or collaborate on [entity]?"
- "Is there a need for templates or presets?"
- "Should users be able to customize their view or preferences?"

### Error Handling
- "What happens when [external service] is down?"
- "How should the system handle network failures?"
- "What if a user loses connection mid-operation?"
- "How do users recover from mistakes?"
- "What error messages should users see for [scenario]?"

## User Experience

### Feedback & States
- "What does the user see while [long operation] is processing?"
- "What should users see when there's no data yet?"
- "How are users notified about [important event]?"
- "What confirmation do users get after [action]?"
- "Should there be confirmation dialogs for [destructive action]?"

### Navigation & Flow
- "How does a first-time user get started?"
- "What's different for a returning user vs new user?"
- "How do users navigate from [screen A] to [screen B]?"
- "Can users access [feature] from multiple places?"
- "What happens if a user navigates away mid-process?"

### Accessibility & Usability
- "Are there keyboard shortcuts for common actions?"
- "How does this work on mobile devices?"
- "What accessibility features are needed?"
- "Should the interface support multiple languages?"
- "How do color-blind users distinguish [elements]?"

## Data & Integration

### Data Management
- "How long should we retain [data type]?"
- "What happens to user data when they delete their account?"
- "Can users export their data? In what format?"
- "How is data backed up and recovered?"
- "What data validation rules are needed?"
- "How do we handle data migration from [old system]?"

### Integrations
- "What external systems need to integrate with this?"
- "What happens if [external API] changes or breaks?"
- "How often should we sync with [external system]?"
- "What data do we send to/receive from [integration]?"
- "Are there webhook or callback requirements?"

### Data Consistency
- "What happens if two users edit the same [entity] simultaneously?"
- "How do we handle conflicts in [data]?"
- "What's the source of truth for [data type]?"
- "How do we ensure data consistency across [services]?"

## Security & Permissions

### Authentication & Authorization
- "Who can access [feature/data]?"
- "How do users authenticate? (password, SSO, 2FA?)"
- "What happens after multiple failed login attempts?"
- "How long should sessions last?"
- "Can users have different roles? What are they?"

### Data Security
- "What data is considered sensitive?"
- "How should sensitive data be encrypted?"
- "Who can view [sensitive information]?"
- "Are there audit log requirements?"
- "What compliance requirements apply? (GDPR, HIPAA, etc.)"

### API Security
- "How are API requests authenticated?"
- "What rate limits should be applied?"
- "How do we prevent abuse or attacks?"
- "Should API keys be rotatable?"
- "What IP restrictions are needed?"

## Performance & Scale

### Performance Requirements
- "What's an acceptable response time for [operation]?"
- "How many concurrent users should the system support?"
- "What's the expected data volume in 1 year? 5 years?"
- "Which operations are most performance-critical?"
- "What happens under heavy load?"

### Scalability
- "How will the system scale as users grow?"
- "What's the caching strategy?"
- "Are there any batch processing needs?"
- "Should we implement pagination? Where?"
- "What database optimization is needed?"

### Resource Management
- "Are there file size limits for uploads?"
- "How many [entities] can a user create?"
- "What happens when storage limits are reached?"
- "Are there any quotas or throttling needed?"

## Operations & Monitoring

### Monitoring & Alerts
- "What metrics should we monitor?"
- "What alerts are critical for operations?"
- "How do we know if the system is healthy?"
- "What SLAs or uptime requirements exist?"
- "What should be logged for debugging?"

### Deployment & Maintenance
- "How will updates be deployed?"
- "What's the rollback strategy if deployment fails?"
- "How are database migrations handled?"
- "Can the system be updated without downtime?"
- "How are configuration changes managed?"

### Support & Troubleshooting
- "What information do support teams need to debug issues?"
- "How can we trace a user's actions?"
- "What logs are needed for troubleshooting?"
- "How do we reproduce reported issues?"

## Business & Compliance

### Business Rules
- "What business rules govern [feature]?"
- "Are there any regulatory requirements?"
- "What happens during [business event like end of month]?"
- "Are there any seasonal or time-based behaviors?"
- "What's the business logic for [calculation]?"

### Compliance & Legal
- "What data privacy laws apply?"
- "Are there data residency requirements?"
- "What terms of service or legal agreements are needed?"
- "How do we handle GDPR requests (data export, deletion)?"
- "Are there age restrictions or parental consent needs?"

### Billing & Monetization
- "How is usage tracked for billing?"
- "What happens when payment fails?"
- "Can users upgrade/downgrade plans?"
- "How are refunds handled?"
- "What happens to data when subscription ends?"

## Edge Cases & Scenarios

### Concurrent Operations
- "What if two users try to [action] at the same time?"
- "How do we handle race conditions in [scenario]?"
- "What if a user opens multiple tabs/sessions?"

### Failure Scenarios
- "What if [external dependency] is down?"
- "How do we handle partial failures?"
- "What if the operation times out?"
- "How do we recover from crashes?"

### Boundary Conditions
- "What's the maximum [number/size/length] allowed?"
- "What happens with zero or empty [data]?"
- "How do we handle very old data?"
- "What if a user has thousands of [entities]?"

### Time-Based Scenarios
- "What happens across time zones?"
- "How do we handle daylight saving time?"
- "What if a scheduled task fails?"
- "How long do temporary items persist?"

## Validation Questions

### Requirement Clarity
- "When you say '[vague term]', what specifically do you mean?"
- "Can you give me a concrete example of [scenario]?"
- "What does success look like for [feature]?"
- "How will we measure if [feature] is working well?"

### Priority Validation
- "Is [feature] truly required for MVP or could it wait?"
- "If we had to cut one P0 feature, which would it be?"
- "What's the impact if we don't include [feature]?"
- "Which features depend on each other?"

### Conflict Resolution
- "I see [requirement A] says X but [requirement B] says Y. Which is correct?"
- "The timeline shows [X weeks] but scope includes [large set]. Should we adjust?"
- "This architecture choice conflicts with [requirement]. How should we resolve?"
- "These two features seem to contradict each other. Can you clarify?"
