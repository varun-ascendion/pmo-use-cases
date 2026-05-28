# Jira Epic Template

## 1.	Epic Description
Problem Statement
- What technical limitation, risk, or inefficiency exists today?
- Why does it matter now (scale, reliability, security, cost, velocity)?
Goal / Outcome
- What measurable or observable improvement will this epic deliver?
- Focus on system behavior, not tasks.
In Scope
- Explicitly list what this epic includes.
Out of Scope
- Explicit exclusions to prevent scope creep.

## 2.	Business / Technical Value
- Reliability, scalability, security, developer velocity, cost reduction, compliance, etc.
- If applicable, quantify (e.g., latency ↓ 30%, incidents ↓, cloud cost ↓).

## 3.	Success Metrics (Acceptance Criteria at Epic Level)
- Clear, testable signals of completion
Examples:
o	99.9% auth service uptime over 30 days
o	P95 latency < 200ms
o	Zero Sev-1 incidents related to auth flows
o	All consumers migrated to new API

## 4.	Dependencies & Risks
Dependencies
- External teams, vendors, infra changes, data migrations, approvals.
Risks
- Migration risk, backward compatibility, performance regressions, unknown legacy behaviour.

## 5.	Assumptions
- Known constraints or expectations (traffic levels, infra availability, timelines).

## 6.	Non-Functional Requirements (NFRs)
- Performance:
- Security:
- Compliance:
- Observability:
- Availability / DR:

## 7.	Rollout / Release Strategy
- Feature flags, canary, phased rollout, dual-run, rollback plan.

## 8.	Linked Issues
- Child user stories
- Spikes
- Bugs
- Tech debt items

---


# Jira User Story Template

## 1.	User Story Statement

For technical stories, avoid fake personas. Use system-oriented framing:
As a platform / system / service
I want to [capability or change]
So that [technical or business outcome]


Example:
As the authentication platform
I want to issue short-lived JWTs
So that internal services can authenticate securely without session state

## 2.	Description / Context
- Current behaviour
- Desired behaviour
- Why this change is required
- Relevant architectural context

## 3.	Acceptance Criteria (Required)
Use Given / When / Then where possible.

Functional
- Given X, when Y, then Z happens
Non-Functional
- Performance thresholds
- Security constraints
- Backward compatibility
- Logging / metrics emitted

Example:
- Given a valid client credential, when requesting a token, then a JWT is returned
- Tokens expire after 15 minutes
- Token validation latency < 50ms
- Invalid tokens return 401 with standard error format

## 4.	Technical Notes / Implementation Details
- Design constraints
- Libraries / frameworks
- API changes
- Data model changes
- Infra implications

## 5.	Dependencies
- Blocked by other stories, infra changes, approvals, or migrations.

## 6.	Testing Notes
- Unit tests required
- Integration tests required
- Load / security testing expectations

## 7.	Definition of Done
- Code merged to main
- Tests passing
- Documentation updated
- Metrics/logs verified
- Deployed to required environment(s)

## 8.	Estimation
- Story points / t-shirt size
- Complexity notes (optional)

