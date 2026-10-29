
# FES-000: FighterOS Foundation Contract

- **Status:** Draft — pending review
- **Version:** 1.0
- **Owner:** FighterOS Engineering
- **Last updated:** 2026-10-29

## 1. Purpose

FighterOS is an evidence-driven training coordination and athlete operating system.

It connects training observations, performance analysis, planning, execution, outcomes, and subsequent feedback into a traceable workflow.

The system aims to help athletes and coaches make better-informed training decisions while preserving data integrity, athlete safety, and appropriate human oversight.

### 1.1 Core operating loop

The intended operating loop is:

1. Observe training activity and athlete feedback.
2. Retrieve and validate recorded data.
3. Estimate the athlete's current state from available evidence.
4. Analyze trends, workload, and relevant constraints.
5. Generate candidate plans or recommendations.
6. Validate proposed actions against defined rules and permissions.
7. Present decisions or plans for appropriate review.
8. Record execution and subsequent outcomes.
9. Use new observations to inform future decisions.

This loop is a product-level model, not a claim that every stage is implemented.

## 2. Initial product scope

The initial release will establish one reliable, testable workflow for training-session records.

The workflow must eventually support:

- Recording a training session.
- Retrieving persisted session data.
- Validating inputs and enforcing data integrity.
- Summarizing verified session information.
- Presenting useful information to an athlete or coach.
- Recording relevant feedback for subsequent analysis.

The first implementation must demonstrate a complete end-to-end workflow before additional capabilities are introduced.

## 3. Explicit non-goals

The initial milestone will not implement:

- Microservices or distributed orchestration.
- Autonomous AI-controlled training decisions.
- Complex multi-tenant authorization without a demonstrated requirement.
- Production-scale infrastructure before deployment needs are established.
- Redis, message brokers, or background workers without a concrete use case.
- Medical diagnosis or autonomous treatment recommendations.
- Advanced performance analytics before the underlying data and metrics are defined.

These capabilities may be reconsidered when product evidence and engineering requirements justify them.

## 4. Architecture boundaries

### 4.1 Application API

The API is responsible for:

- Handling requests and responses.
- Validating request data at system boundaries.
- Enforcing application workflows and business rules.
- Applying authorization checks where required.
- Coordinating persistence operations.
- Returning explicit, consistent errors.

The API must not trust client-supplied data merely because it is correctly formatted.

### 4.2 PostgreSQL and persistence

PostgreSQL is the proposed authoritative store for application records.

The persistence layer is responsible for:

- Storing and retrieving application data.
- Enforcing appropriate primary-key, foreign-key, uniqueness, and integrity constraints.
- Supporting transactional operations where consistency requires them.
- Preserving the distinction between recorded observations and derived estimates.

Derived values must not silently overwrite source observations.

### 4.3 Background workers

Workers are deferred until asynchronous processing is justified.

If introduced, workers must:

- Operate within defined application boundaries.
- Validate their inputs.
- Handle retries and failures deliberately.
- Avoid duplicate side effects where retries are possible.
- Preserve traceability for consequential operations.

Workers must not bypass business invariants or authorization requirements.

### 4.4 AI-assisted components

AI-assisted components may analyze verified data and propose structured outputs, including:

- Training summaries.
- Performance insights.
- Candidate training plans.
- Explanations of observed trends.

AI-generated output is untrusted input.

Before a proposed action can be authorized, the system must validate its structure, data references, applicable business rules, safety constraints, and permissions.

AI output alone must never authorize an operational action.

### 4.5 User interface

The user interface is responsible for presenting relevant records, analyses, and proposed plans.

It must distinguish, where applicable:

- Recorded observations.
- Calculated metrics.
- Estimated athlete state.
- AI-generated recommendations.
- Approved plans.
- Completed activities.

The interface must not imply that a recommendation has been approved or executed when it has not.

### 4.6 External integrations

External services are outside the core domain boundary.

Integrations must be introduced for a defined purpose and must not become the authoritative source for FighterOS business rules.

Credentials and sensitive integration configuration must be managed outside committed source files.

## 5. Provisional technology decisions

The following choices represent the current engineering direction. They are not all irreversible or fully validated decisions.

| Area | Proposed choice | Status |
|---|---|---|
| Backend language | TypeScript | Provisional |
| Backend framework | NestJS | Provisional |
| Primary database | PostgreSQL | Provisional |
| Initial architecture | Modular monolith | Provisional |
| Version control | Git and GitHub | Adopted |
| Current development environment | GitHub Codespaces | Adopted for current workflow |
| Cache and queue infrastructure | Redis or another suitable service | Deferred |
| Background processing | Worker process or managed service | Deferred |
| AI orchestration | To be selected when a concrete use case requires it | Unresolved |

The modular monolith is the default starting architecture because it keeps the first implementation understandable and testable without prematurely distributing the system.

Technology choices must be reviewed against the first end-to-end workflow before they are treated as stable architectural commitments.

## 6. Engineering invariants

The following invariants apply to subsequent specifications and implementation.

### 6.1 Input validation

- Validate external inputs at system boundaries.
- Reject invalid data with explicit errors.
- Do not treat type correctness as proof of business validity.

### 6.2 Data integrity

- Persisted records are the source of truth for recorded events.
- Use database constraints and transactions where appropriate.
- Preserve source observations separately from derived values.
- Represent missing, invalid, or uncertain information explicitly.

### 6.3 Authorization and action control

- Validate proposed actions against deterministic business rules.
- Enforce applicable permissions before authorizing actions.
- Do not allow AI-generated content to bypass validation or authorization.
- Require human review for consequential decisions where the product's safety policy demands it.

### 6.4 Traceability

- Record relevant state changes and consequential decisions.
- Maintain a clear distinction between a proposal, an approval, and an executed action.
- Preserve enough context to explain how a result was produced.

### 6.5 Correctness and testing

- Test business rules independently of framework implementation.
- Test valid inputs, invalid inputs, and important edge cases.
- Verify calculations against known inputs and expected outputs.
- Test persistence and API behavior at integration boundaries.
- Do not claim that a feature is verified until the relevant checks have actually run.

### 6.6 Security and privacy

- Never commit credentials, API keys, or database passwords.
- Avoid collecting athlete data that is not needed for a defined purpose.
- Apply appropriate access controls before exposing sensitive records.
- Do not use real private athlete data in tests unless explicitly justified and protected.

## 7. Athlete safety and evidence

FighterOS supports training coordination and performance analysis. It is not automatically a medical diagnostic system.

The system should:

- Distinguish measured values from calculated and estimated values.
- Identify relevant data limitations.
- Avoid presenting uncertain conclusions as facts.
- Avoid recommending training loads that contradict known constraints.
- Require appropriate human review for consequential training changes.
- Avoid inferring medical readiness from incomplete or unreliable data.

Safety rules must be made concrete in the relevant feature specifications and tests rather than remaining only general principles.

## 8. Initial implementation strategy

Implementation will proceed in small, verifiable milestones.

### Milestone 1: Foundation contract

- Review and approve this document.
- Resolve only the architectural questions needed for the first feature.
- Record material decisions and their rationale.

### Milestone 2: Application scaffold

- Establish the TypeScript and NestJS project structure.
- Configure development and test commands.
- Establish environment-variable conventions.
- Prepare PostgreSQL development configuration.
- Verify that the application starts successfully.

### Milestone 3: First vertical slice

Implement one training-session workflow:

1. Define the session data contract.
2. Define required fields and validation rules.
3. Implement a validated API operation.
4. Persist and retrieve a session using PostgreSQL.
5. Add automated unit and integration tests.
6. Verify the workflow against explicit acceptance criteria.

### Milestone 4: Review and extend

- Review implementation against the foundation contract.
- Identify correctness gaps and operational risks.
- Record unresolved decisions.
- Expand only after the current milestone passes its acceptance criteria.

Authentication, deployment hardening, and data-access policies must be addressed before exposing sensitive data or operating a production service.

## 9. Open decisions

The following decisions remain unresolved and must not be silently treated as confirmed:

1. Exact fields and validation rules for the initial training-session record.
2. Athlete and coach ownership relationships.
3. Authentication and authorization requirements for the initial release.
4. Which performance metrics are supported by sufficiently reliable evidence.
5. Deployment target and operational requirements.
6. Whether and when asynchronous processing is necessary.
7. Retention, export, and deletion requirements for athlete data.
8. Which AI-assisted capabilities, if any, belong in the initial product.

Resolve each question when it becomes necessary for a defined implementation task.

For each material decision, document the selected option, rationale, relevant alternatives, and consequences.

## 10. Acceptance criteria

This foundation contract is ready for approval when all the following checks pass:

- [ ] Purpose and intended outcome are explicit.
- [ ] Initial scope and non-goals are defined.
- [ ] Responsibilities of the API, persistence, workers, AI, and UI are separated.
- [ ] Technology decisions are classified as adopted, provisional, deferred, or unresolved.
- [ ] Validation, integrity, authorization, and traceability invariants are documented.
- [ ] Athlete safety and privacy principles are explicit.
- [ ] Open decisions are recorded without being presented as confirmed requirements.
- [ ] The initial implementation sequence is bounded and testable.
- [ ] The document contains no contradictory requirements or accidental shell commands.
- [ ] The document has been reviewed before approval.

## 11. Change control

This document remains a draft until reviewed and approved.

Changes to scope, architecture boundaries, or engineering invariants must include a rationale.

Feature specifications and implementation tasks must be consistent with this contract. If a genuine requirement conflicts with it, document the conflict and revise the contract explicitly rather than silently violating it.

## 12. Current status

FighterOS has an initialized GitHub repository and a working cloud development environment.

The foundation contract is the first engineering specification. Its presence does not imply that the application scaffold, database integration, validation logic, or end-to-end workflow has been implemented or tested.

The next action is to review this document, commit the approved version, and then establish the application scaffold.
