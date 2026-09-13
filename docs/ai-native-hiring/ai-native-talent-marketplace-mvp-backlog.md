# AI-Native Talent Marketplace — Solopreneur MVP Backlog

## Product objective

Build the smallest trustworthy operating system for a human-led AI talent agent:

1. Find and qualify a narrow set of high-signal candidates.
2. Maintain consented, evidence-backed candidate profiles.
3. Intake a small number of startup roles.
4. Generate explainable candidate-role matches.
5. Get candidate and employer approval before every introduction.
6. Track interviews, hires, placements, and fees.
7. Let one founder operate through an exception queue rather than a pile of messages.

**Market-analysis boundary:** Cleara is treated only as a competitor and market reference. This backlog is for a brand-neutral product.

## MVP boundary

### In scope

- Startup and scale-up knowledge-work hiring
- Engineering, product, design, operations, and technical business roles
- Candidate and employer records
- Evidence-backed profiles
- Purpose-bound consent and suppression
- Manual or semi-automated outreach
- Explainable matching
- Mutual opt-in introductions
- Interview and placement tracking
- Success-fee attribution and invoice preparation
- Founder dashboard and agent task queue

### Out of scope for MVP

- General labor marketplace
- High-volume outbound campaigns
- Autonomous cold outreach without founder approval
- Automatic candidate enrollment from public profiles
- Fully automated legal or privacy determinations
- Native iMessage or WhatsApp integrations before compliance and deliverability are proven
- Complex enterprise ATS integrations
- Candidate mobile app
- Automated payments and collections before placement attribution is reliable
- Marketplace self-service for every candidate and employer

---

# 1. Prioritized database-table backlog

## P0 — Required for a trustworthy operating loop

These tables support the first complete path:

```text
Candidate → Consent → Profile → Role → Match → Interest
→ Introduction → Interview → Hiring Outcome → Placement
```

### P0.1 `people`

**Purpose:** Canonical person identity.

**Minimum fields**

- `id`
- `name`
- `email`
- `phone`
- `timezone`
- `country`
- `status`
- `created_at`
- `updated_at`

**Acceptance criteria**

- Duplicate email and phone values are detected.
- A person can be linked to a candidate or employer contact.
- Deletion and suppression can target the canonical person.

### P0.2 `candidates`

**Purpose:** Candidate lifecycle and ownership.

**Minimum fields**

- `id`
- `person_id`
- `status`
- `source_type`
- `source_reference`
- `primary_function`
- `seniority`
- `location`
- `work_authorization`
- `availability`
- `owner_user_id`
- `created_at`
- `updated_at`

**Acceptance criteria**

- New records start as `discovered`, never as `represented`.
- Candidate status changes are audited.
- Suppressed candidates cannot be selected for outreach or matching.

### P0.3 `candidate_profiles`

**Purpose:** Versioned candidate representation.

**Minimum fields**

- `id`
- `candidate_id`
- `version`
- `headline`
- `summary`
- `skills_json`
- `work_history_json`
- `preferences_json`
- `compensation_json`
- `availability`
- `generated_by`
- `approved_at`
- `effective_from`
- `effective_to`

**Acceptance criteria**

- Profiles are versioned, not overwritten.
- Only an approved version can be shared externally.
- Every material profile field can point to evidence.

### P0.4 `evidence`

**Purpose:** Provenance for profile and role claims.

**Minimum fields**

- `id`
- `entity_type`
- `entity_id`
- `field_path`
- `source_type`
- `source_locator`
- `confidence`
- `verification_status`
- `captured_at`

**Acceptance criteria**

- A profile claim can be traced to a source.
- AI-inferred fields are labeled as inferred.
- Low-confidence claims are visible to the founder.

### P0.5 `consents`

**Purpose:** Purpose-bound permissions.

**Minimum fields**

- `id`
- `candidate_id`
- `purpose`
- `status`
- `allowed_channels`
- `allowed_roles`
- `allowed_companies`
- `notice_version`
- `captured_from`
- `captured_at`
- `expires_at`

**Acceptance criteria**

- Consent for contact is separate from consent for employer sharing.
- Every outbound message and profile share performs a live consent check.
- Withdrawal immediately blocks future activity.

### P0.6 `suppressions`

**Purpose:** Hard block on contact or sharing.

**Minimum fields**

- `id`
- `person_id`
- `scope`
- `channels`
- `reason`
- `created_at`
- `expires_at`

**Acceptance criteria**

- Suppression is checked in the same transaction or command path as send/share.
- A candidate who opts out cannot be re-enqueued by an agent.
- Repeat-contact violations are detectable.

### P0.7 `companies`

**Purpose:** Employer account and relationship record.

**Minimum fields**

- `id`
- `name`
- `domain`
- `website`
- `description`
- `stage`
- `sector`
- `verification_status`
- `account_status`
- `owner_user_id`

**Acceptance criteria**

- Employer contacts can be associated with a company.
- Company status supports `prospect`, `qualified`, `active`, `paused`, `blocked`.
- The founder can flag a company as not suitable for candidate presentation.

### P0.8 `employer_contacts`

**Purpose:** Employer-side identity and authority.

**Minimum fields**

- `id`
- `company_id`
- `person_id`
- `role`
- `permissions`
- `verified_at`
- `status`

**Acceptance criteria**

- Only authorized contacts can approve role requirements or candidate interest.
- Every employer action is attributable to a person.

### P0.9 `roles`

**Purpose:** Active employer demand.

**Minimum fields**

- `id`
- `company_id`
- `title`
- `function`
- `seniority`
- `description`
- `location_policy`
- `locations`
- `compensation_json`
- `visibility`
- `status`
- `fee_agreement_id`
- `activated_at`
- `closed_at`

**Acceptance criteria**

- A role cannot be activated without required fields and founder approval.
- Closed roles cannot generate new matches.
- Confidential roles have controlled disclosure.

### P0.10 `role_requirements`

**Purpose:** Structured role criteria.

**Minimum fields**

- `id`
- `role_id`
- `category`
- `field`
- `operator`
- `value`
- `weight`
- `hard_constraint`
- `human_verified`

**Acceptance criteria**

- Must-haves are distinguishable from preferences.
- The founder can edit AI-generated requirements.
- Match explanations reference these requirements.

### P0.11 `matches`

**Purpose:** Candidate-role fit record.

**Minimum fields**

- `id`
- `candidate_id`
- `role_id`
- `status`
- `score`
- `score_version`
- `hard_constraint_result`
- `reasons_json`
- `risks_json`
- `model_run_id`
- `created_at`
- `expires_at`

**Acceptance criteria**

- Every match has an explanation, not only a score.
- Consent and representation scope are checked before a match becomes actionable.
- The founder can approve, reject, snooze, or request more evidence.

### P0.12 `interest_signals`

**Purpose:** Independent candidate and employer decisions.

**Minimum fields**

- `id`
- `match_id`
- `actor_type`
- `actor_id`
- `signal`
- `reason`
- `captured_at`

**Acceptance criteria**

- Silence never counts as interest.
- Candidate and employer signals are stored separately.
- An introduction requires two positive signals.

### P0.13 `introductions`

**Purpose:** Commercial and relationship boundary.

**Minimum fields**

- `id`
- `match_id`
- `candidate_id`
- `company_id`
- `role_id`
- `candidate_interest_signal_id`
- `employer_interest_signal_id`
- `method`
- `status`
- `sent_at`
- `accepted_at`

**Acceptance criteria**

- Introduction cannot be sent without both interest signals.
- The exact candidate packet and role version are recorded.
- The founder can dispute or cancel an introduction.

### P0.14 `interviews`

**Purpose:** Track post-introduction progress.

**Minimum fields**

- `id`
- `introduction_id`
- `stage`
- `status`
- `scheduled_at`
- `candidate_feedback`
- `employer_feedback`
- `feedback_captured_at`

**Acceptance criteria**

- A stalled interview appears in the founder queue.
- Candidate and employer feedback are labeled by source.
- AI summaries cannot overwrite original feedback.

### P0.15 `hiring_outcomes`

**Purpose:** Capture the result of an introduction.

**Minimum fields**

- `id`
- `introduction_id`
- `status`
- `start_date`
- `compensation_json`
- `reported_by`
- `reported_at`
- `evidence_id`

**Acceptance criteria**

- Hires are not inferred solely from a message or profile update.
- Conflicting reports create an exception.
- A confirmed hire can create a placement recommendation.

### P0.16 `fee_agreements`

**Purpose:** Commercial terms for each employer.

**Minimum fields**

- `id`
- `company_id`
- `fee_type`
- `percentage_of_salary`
- `flat_amount`
- `replacement_window_days`
- `terms_version`
- `accepted_at`

**Acceptance criteria**

- Every active role links to an accepted fee agreement.
- Terms are versioned.
- The calculation basis is explicit.

### P0.17 `placements`

**Purpose:** Attributable hiring revenue.

**Minimum fields**

- `id`
- `hiring_outcome_id`
- `candidate_id`
- `company_id`
- `role_id`
- `fee_agreement_id`
- `attribution_status`
- `guarantee_ends_at`

**Acceptance criteria**

- Placement traces to a specific introduction.
- Placement remains provisional during the guarantee window.
- Disputed attribution is visible and blocks automatic invoicing.

### P0.18 `invoices`

**Purpose:** Invoice preparation and status.

**Minimum fields**

- `id`
- `placement_id`
- `amount`
- `currency`
- `status`
- `issued_at`
- `due_at`
- `paid_at`

**Acceptance criteria**

- Invoice cannot be generated without a valid placement and fee agreement.
- Fee calculation is reproducible.
- Disputed placements cannot be auto-issued.

### P0.19 `audit_events`

**Purpose:** Immutable operational history.

**Minimum fields**

- `id`
- `actor_type`
- `actor_id`
- `action`
- `entity_type`
- `entity_id`
- `before_json`
- `after_json`
- `reason`
- `occurred_at`

**Acceptance criteria**

- Consent, profile approval, external send, external share, introduction, placement, and invoice actions are logged.
- Agent actions identify the agent and version.
- Founder overrides include a reason.

## P1 — Required once the first loop works

### P1.1 `agent_definitions`

Stores agent purpose, version, tools, policies, allowed actions, and approval rules.

### P1.2 `agent_tasks`

Stores queued work, recommendations, confidence, risk flags, human decisions, and execution status.

### P1.3 `agent_runs`

Stores model execution metadata, input references, output references, latency, cost, and error status.

### P1.4 `agent_identities`

Stores sender identity, disclosure text, verified domains, channels, and active/inactive state.

### P1.5 `outreach_campaigns`

Stores audience definition, purpose, source policy, consent policy, template version, cadence, and founder approval.

### P1.6 `outreach`

Stores immutable message-level activity, destination, purpose, consent reference, delivery, reply, bounce, and opt-out.

### P1.7 `conversations`

Stores candidate and employer relationship threads across email, portal, and later messaging channels.

### P1.8 `messages`

Stores sender type, AI status, disclosure status, body, template, delivery status, and timestamps.

### P1.9 `candidate_packets`

Stores role-specific candidate versions, redactions, fields shared, candidate approval, and expiration.

### P1.10 `privacy_requests`

Stores access, correction, deletion, restriction, and opt-out requests.

### P1.11 `trust_events`

Stores spam reports, privacy complaints, identity concerns, consent mismatches, abuse reports, and bounce clusters.

### P1.12 `notifications`

Stores founder alerts, candidate reminders, employer reminders, and escalation state.

## P2 — Defer until repeatable demand exists

- `referrals`
- `candidate_reviews`
- `employer_reviews`
- `marketplace_taxonomy`
- `integrations`
- `external_sync_cursors`
- `payment_transactions`
- `replacement_claims`
- `candidate_community_memberships`
- `experiments`
- `model_evaluations`
- `data_retention_jobs`
- `jurisdiction_rules`

Do not build P2 tables until the founder has enough real workflow volume to validate their shape.

---

# 2. Prioritized AI-agent backlog

## P0 agents — Build these first

## A0. Founder Operations Agent

**Mission:** Turn all system activity into a ranked exception queue.

**Reads**

- Candidate, role, match, interview, placement, trust, and agent-task data

**Produces**

- Daily operating brief
- “Needs founder decision” queue
- Stalled workflow alerts
- Revenue-at-risk alerts
- Trust-risk alerts

**May act autonomously**

- Summarize
- Prioritize
- Deduplicate tasks
- Remind the founder

**Requires founder approval**

- Any external message
- Any role activation
- Any candidate representation decision
- Any placement or invoice exception

**MVP success metric:** Founder can identify the five highest-value decisions in under ten minutes.

## A1. Candidate Research Agent

**Mission:** Find, normalize, enrich, and prioritize candidate leads from approved sources.

**Inputs**

- Search results
- Referrals
- Resumes
- Public professional profiles
- Existing candidate records

**Outputs**

- Candidate records
- Evidence records
- Duplicate suggestions
- Fit summary
- Missing-information tasks

**Hard limits**

- Cannot contact a person.
- Cannot mark a person represented.
- Cannot infer protected characteristics.
- Cannot silently merge identities.

**MVP success metric:** 80% of candidate records are reviewable without manual reformatting.

## A2. Candidate Profile Agent

**Mission:** Draft and maintain evidence-backed candidate profiles.

**Inputs**

- Candidate sources
- Resume or portfolio
- Candidate conversation
- Existing approved profile

**Outputs**

- Profile draft
- Evidence links
- Confidence scores
- Contradiction alerts
- Candidate review request

**Hard limits**

- Cannot publish an unapproved profile externally.
- Cannot invent achievements.
- Cannot convert an inference into a fact.

**MVP success metric:** Candidate can approve or correct a profile in one short review session.

## A3. Role Intake Agent

**Mission:** Convert a job description and founder conversation into a structured role.

**Inputs**

- Job description
- Employer intake notes
- Company research
- Compensation and location information

**Outputs**

- Role draft
- Must-have requirements
- Nice-to-have requirements
- Missing-question checklist
- Candidate-facing summary

**Hard limits**

- Cannot activate a role without founder approval.
- Cannot decide ambiguous requirements without escalation.

**MVP success metric:** Founder can turn an unstructured role into an active brief in under 15 minutes.

## A4. Matching Agent

**Mission:** Produce explainable candidate-role opportunities.

**Inputs**

- Approved candidate profile
- Candidate preferences
- Active role requirements
- Representation scope
- Consent state

**Outputs**

- Match score
- Hard-constraint result
- Reasons
- Risks
- Candidate-facing opportunity draft

**Hard limits**

- Cannot share a candidate with an employer.
- Cannot bypass consent.
- Cannot use hidden sensitive attributes.

**MVP success metric:** Every recommended match has at least three evidence-backed reasons or is routed to review.

## A5. Relationship and Follow-up Agent

**Mission:** Keep candidates and employers moving through approved workflows.

**Inputs**

- Conversations
- Interest signals
- Interview state
- Reminder policies

**Outputs**

- Follow-up drafts
- Scheduling requests
- Missing-feedback tasks
- Escalation flags

**Hard limits**

- External send requires approval in P0.
- Cannot send after opt-out or suppression.
- Cannot impersonate a human.

**MVP success metric:** No active opportunity is idle without a next action or explicit pause reason.

## P1 agents — Add after the core loop is reliable

### A6. Employer Research Agent

Researches companies, hiring signals, contacts, market context, and account risk.

### A7. Candidate Packet Agent

Creates role-specific, redacted, candidate-approved employer packets.

### A8. Interview Coordinator Agent

Schedules, reminds, collects feedback, and detects stalled processes.

### A9. Placement Attribution Agent

Detects likely hires, requests confirmation, checks attribution, and prepares a placement recommendation.

### A10. Billing Agent

Calculates fees, prepares invoices, monitors due dates, and escalates disputes.

### A11. Trust Monitoring Agent

Detects complaints, consent mismatches, unusual bounce rates, repeat contacts, and risky campaigns.

### A12. Market Intelligence Agent

Tracks role demand, candidate supply, competitor movement, and changes to the chosen niche.

---

# 3. Prioritized business-process backlog

## P0 process 1 — Founder defines the niche and policies

**Trigger:** New market or quarterly review.

**AI work**

- Summarize demand and supply.
- Recommend role segments and target companies.
- Identify policy questions and risk areas.

**Human lead**

- Choose the market.
- Set inclusion and exclusion rules.
- Approve compensation, geography, and outreach policies.

**System output**

- Market policy
- Role taxonomy
- Candidate qualification rubric
- Agent permissions

**Done when**

- An agent can determine whether a candidate, company, or role is inside the initial business scope.

## P0 process 2 — Candidate discovery

**Trigger:** Approved source, referral, or manual lead.

**AI work**

- Create or update candidate.
- Normalize identity.
- Extract evidence.
- Estimate fit and priority.

**Human lead**

- Approve source strategy.
- Review ambiguous identities and high-value candidates.

**System output**

- Candidate record
- Evidence records
- Discovery task

**Done when**

- Every candidate has a source, status, owner, and next action.

## P0 process 3 — Candidate consented onboarding

**Trigger:** Candidate replies, signs up, or accepts an invitation.

**AI work**

- Explain the platform’s service.
- Answer routine questions.
- Capture purpose-specific consent.
- Create a representation agreement draft.

**Human lead**

- Handle questions outside policy.
- Approve exceptions and high-value relationships.

**System output**

- Consent records
- Suppression if declined
- Active representation if approved

**Done when**

- The system can answer “what may the platform do with this candidate’s data right now?”

## P0 process 4 — Candidate profile creation

**Trigger:** Active representation or candidate request.

**AI work**

- Draft profile.
- Link evidence.
- Detect contradictions.
- Request candidate review.

**Human lead**

- Review sensitive claims and strategic candidates.

**External decision**

- Candidate approves or corrects the profile.

**System output**

- Approved profile version
- Open profile tasks

**Done when**

- No unapproved profile can be shared externally.

## P0 process 5 — Employer and role intake

**Trigger:** Employer referral, founder outreach, or inbound role.

**AI work**

- Research company.
- Structure role.
- Identify missing requirements.
- Draft candidate-facing role summary.

**Human lead**

- Qualify employer.
- Confirm hiring authority.
- Agree fee terms.
- Activate role.

**System output**

- Company
- Employer contact
- Role
- Role requirements
- Fee agreement

**Done when**

- Every active role has a verified employer, structured requirements, owner, status, and commercial terms.

## P0 process 6 — Candidate-role matching

**Trigger:** Active role or material profile update.

**AI work**

- Apply hard constraints.
- Score fit.
- Generate evidence-backed reasons and risks.
- Queue candidate opportunity.

**Human lead**

- Review top matches and exceptions.
- Override or reject poor recommendations.

**System output**

- Match
- Match explanation
- Candidate opportunity task

**Done when**

- The founder can understand and approve a match without rereading every source document.

## P0 process 7 — Candidate interest capture

**Trigger:** Approved match.

**AI work**

- Present opportunity.
- Explain fit.
- Answer routine questions.
- Capture interest, decline, or request for information.

**Human lead**

- Handle career advice, sensitive information, and negotiation.

**System output**

- Interest signal
- Follow-up task
- Updated preference or suppression record if relevant

**Done when**

- No employer sees the candidate until candidate interest is explicit.

## P0 process 8 — Employer interest capture

**Trigger:** Candidate interest.

**AI work**

- Prepare role-specific candidate packet.
- Summarize candidate evidence.
- Ask employer whether to proceed.

**Human lead**

- Approve sensitive packet content.
- Handle objections and positioning.

**System output**

- Employer interest signal
- Packet-share audit event

**Done when**

- Employer receives only candidate-approved, role-relevant information.

## P0 process 9 — Mutual introduction

**Trigger:** Candidate and employer both signal interest.

**AI work**

- Verify both signals.
- Draft introduction.
- Coordinate logistics.
- Track acceptance.

**Human lead**

- Make strategic introductions.
- Resolve ownership, confidentiality, or reputation issues.

**System output**

- Introduction
- Communication history
- Next-action task

**Done when**

- Every introduction is attributable to one candidate, one company, one role, two interest signals, and one fee agreement.

## P0 process 10 — Interview progress

**Trigger:** Introduction accepted.

**AI work**

- Schedule and remind.
- Request feedback.
- Detect stalled stages.
- Draft next-step communications.

**Human lead**

- Coach, mediate, and intervene in sensitive situations.

**System output**

- Interview records
- Feedback
- Escalation tasks

**Done when**

- Every active introduction has a current stage and next action.

## P0 process 11 — Hire and placement attribution

**Trigger:** Offer signal, employer update, candidate update, or follow-up deadline.

**AI work**

- Detect likely hire.
- Request confirmation.
- Compare evidence.
- Prepare placement recommendation.

**Human lead**

- Confirm disputed outcomes.
- Approve attribution and replacement terms.

**System output**

- Hiring outcome
- Provisional or disputed placement

**Done when**

- A placement can be traced back to a specific introduction and fee agreement.

## P0 process 12 — Invoice preparation

**Trigger:** Confirmed placement or approved invoice condition.

**AI work**

- Calculate fee.
- Draft invoice.
- Track due dates.
- Prepare reminders.

**Human lead**

- Approve invoice.
- Handle discounts, disputes, replacements, and strategic exceptions.

**System output**

- Invoice
- Revenue forecast
- Collection task

**Done when**

- The founder can produce an accurate invoice without reconstructing the hiring history manually.

## P0 process 13 — Privacy, opt-out, and complaint handling

**Trigger:** Candidate request, spam report, bounce cluster, or consent mismatch.

**AI work**

- Classify request.
- Freeze workflows.
- Resolve identity.
- Generate response and execution checklist.
- Monitor for repeat contact.

**Human lead**

- Own serious complaints and legal/reputational decisions.

**System output**

- Privacy request
- Suppression
- Trust event
- Audit trail

**Done when**

- A candidate can be reliably removed from future outreach and employer sharing.

## P0 process 14 — Daily founder operations

**Trigger:** Daily schedule or material event.

**AI work**

- Summarize funnel health.
- Rank tasks by expected value and risk.
- Identify stalled candidates, roles, interviews, placements, and invoices.
- Highlight trust anomalies.

**Human lead**

- Make decisions from the queue.
- Change policies and agent permissions.

**System output**

- Daily brief
- Prioritized queue
- Completed decision log

**Done when**

- The founder can run the business from one screen and does not need to inspect every successful agent action.

---

# 4. Founder dashboard backlog

The dashboard should be organized around decisions, not departments or raw activity.

## Dashboard 1 — Today

**Purpose:** What needs human attention now?

**Cards**

- High-value candidate decisions
- Roles at risk of going stale
- Matches awaiting approval
- Candidates awaiting response
- Employers awaiting response
- Interviews needing intervention
- Placements requiring confirmation
- Invoices requiring approval
- Trust or privacy alerts

**Primary action:** Approve, reject, snooze, delegate, or escalate.

## Dashboard 2 — Marketplace funnel

**Purpose:** Where is the system losing value?

**Views**

- Candidates: discovered → contacted → consented → represented
- Matches: scored → candidate interested → employer interested
- Introductions: sent → accepted → interviewed → hired
- Roles: activated → first match → first introduction → filled

**Filters**

- Function
- Seniority
- Geography
- Company stage
- Agent
- Source
- Campaign

## Dashboard 3 — Trust and compliance

**Purpose:** Detect damage before it becomes a brand problem.

**Views**

- Opt-out rate
- Complaint rate per 1,000 contacts
- Consent completeness
- Repeat-contact violations
- Unverified profile claims
- Sender-domain performance
- Bounces by campaign
- Open privacy requests
- Candidates shared without complete provenance

**Escalation rule:** Any repeat-contact violation or consent mismatch pauses the relevant campaign automatically.

## Dashboard 4 — Revenue

**Purpose:** Understand future cash, not just activity.

**Views**

- Active fee agreements
- Provisional placements
- Confirmed placements
- Guarantee windows ending
- Invoices awaiting approval
- Overdue invoices
- Disputed attribution
- Revenue by role, company, and source

## Dashboard 5 — Agent control room

**Purpose:** Ensure automation is useful, bounded, and improving.

**Views**

- Agent tasks by status
- Human escalation rate
- Approval and rejection rate
- Agent error rate
- Average task latency
- External messages drafted versus sent
- Agent actions reversed by founder
- Cost per completed workflow
- Performance by prompt or policy version

---

# 5. Delivery sequence

## Phase 0 — Operating foundation

**Build**

- Market policy
- Founder account and permissions
- People, candidates, companies, roles
- Audit events
- Basic founder dashboard

**Do not build**

- Autonomous outreach
- General marketplace discovery
- Multiple messaging channels

**Exit condition**

- Founder can create a candidate, company, role, and auditable next action.

## Phase 1 — Trusted candidate and role loop

**Build**

- Candidate profile versions
- Evidence
- Consent
- Suppression
- Role requirements
- Candidate and employer interest signals
- Manual introduction flow

**Agents**

- Candidate Research Agent
- Candidate Profile Agent
- Role Intake Agent
- Matching Agent

**Exit condition**

- One candidate and one employer can move from consented profile to mutual introduction without spreadsheet reconstruction.

## Phase 2 — Founder leverage

**Build**

- Agent registry
- Agent tasks
- Founder exception queue
- Relationship follow-up
- Interview tracking
- Daily operating brief

**Agents**

- Founder Operations Agent
- Relationship and Follow-up Agent

**Exit condition**

- Founder spends time on decisions and relationships rather than formatting, chasing, and copying data.

## Phase 3 — Revenue loop

**Build**

- Hiring outcomes
- Placements
- Fee agreements
- Invoice preparation
- Guarantee tracking
- Revenue dashboard

**Agents**

- Placement Attribution Agent
- Billing Agent

**Exit condition**

- Every paid placement can be traced from invoice back to the original candidate consent, match, and introduction.

## Phase 4 — Controlled scale

**Build**

- Employer research
- Candidate packet generation
- Trust monitoring
- Approved email automation
- Integrations
- Campaign analytics

**Exit condition**

- Outreach volume can grow without reducing consent quality, candidate experience, or founder visibility.

---

# 6. MVP acceptance checklist

The MVP is ready for real use when all of the following are true:

- [ ] Founder can define the target market and excluded categories.
- [ ] Every candidate has a source and lifecycle status.
- [ ] Every material profile claim has evidence or is labeled as an inference.
- [ ] Candidate contact, representation, and employer sharing have separate permissions.
- [ ] Opt-out blocks future outreach immediately.
- [ ] Every role has structured requirements and a fee agreement.
- [ ] Every match includes reasons, risks, and score version.
- [ ] Candidate and employer interest are recorded independently.
- [ ] Introduction cannot be sent without mutual interest.
- [ ] Interview status and next action are visible.
- [ ] Hiring outcome requires evidence or human confirmation.
- [ ] Placement attribution is traceable.
- [ ] Invoice calculation is reproducible.
- [ ] Founder sees exceptions, trust alerts, and revenue risk in one dashboard.
- [ ] Agent actions are versioned and auditable.
- [ ] No AI agent can silently impersonate a human or bypass suppression.

## Recommended first build

Build **Phase 0 and Phase 1 together**, using a narrow founder-controlled pilot:

- 20–50 candidates
- 5–10 employers
- 5–15 active roles
- One approved outbound channel
- Manual founder approval for every external message and every employer share

The goal of the first pilot is not volume. It is to validate that the chain from **consent → profile → match → mutual introduction → interview** creates enough signal to justify automating the revenue and follow-up layers.
