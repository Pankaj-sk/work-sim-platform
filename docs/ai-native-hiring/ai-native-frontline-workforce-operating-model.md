# AI-Native Frontline Workforce Operating Model

## Segment thesis

This version applies the previous talent-marketplace model to **high-volume, operational, rules-based hiring**:

- Barbers and stylists
- Forklift and warehouse operators
- Delivery and gig-economy workers
- Drivers and field service workers
- Retail and restaurant frontline workers
- Hospitality, cleaning, security, and facilities workers
- Light manufacturing and logistics workers

The product is no longer primarily a candidate-side talent agent for startup knowledge work. It becomes an **AI-native workforce qualification and hiring-operations platform**.

**Market-analysis boundary:** Cleara is treated only as a competitor and market reference. This operating model is brand-neutral.

Its central job is:

> Convert a job’s objective requirements and a worker’s verified evidence into a fast, explainable, compliant hiring recommendation—or a clearly defined reason the worker cannot proceed.

The product can support a recruiter or manager by:

1. Defining an objective eligibility matrix.
2. Finding and inviting workers.
3. Collecting applications and evidence.
4. Verifying licenses, identity, work authorization, background requirements, and prior experience.
5. Matching workers to sites, shifts, pay, and availability.
6. Making an employer-authorized hiring recommendation.
7. Skipping the interview when policy allows.
8. Outsourcing or automating the interview when it adds value.
9. Scheduling onboarding, training, and first shift.
10. Monitoring post-hire completion and retention.

---

## Important distinction: objective rules versus employment decisions

Many frontline roles have objective requirements, but the employment decision can still create legal and fairness risk. The system should distinguish three layers:

### Layer 1 — Mechanical eligibility

Examples:

- License exists and is valid.
- Required certification is current.
- Worker is legally authorized to work.
- Worker can work the required shift.
- Worker is within the service radius.
- Worker meets minimum documented experience.
- Required safety training is complete.

These can often be evaluated through deterministic rules.

### Layer 2 — Employer policy fit

Examples:

- Employer accepts equivalent certifications.
- Employer requires six months of similar environment experience.
- Employer permits training after hire.
- Employer accepts a conditional background result.
- Employer requires a minimum availability commitment.

These rules are configurable by employer and role, but must be explicit and versioned.

### Layer 3 — Employment decision

Examples:

- Hire
- Reject
- Hold for human review
- Request additional evidence
- Make a conditional offer

The product can make an **employer-authorized recommendation** or execute a pre-approved decision policy where permitted. It should not hide the decision behind an opaque model or make an irreversible, unreviewable employment decision. The recruiter or employer remains accountable, especially for:

- Background-check adverse action
- Criminal-record interpretation
- Disability or accommodation issues
- Immigration and work authorization exceptions
- Age, location, transportation, or schedule proxies
- Safety-sensitive roles
- Local fair-chance or ban-the-box requirements

---

## Market assumptions

### 1. Jobs are repeatable and location-bound

The same role may be open at many sites with small variations:

- Great Clips barber at a specific store
- Forklift operator at a specific Walmart distribution center
- Delivery worker in a defined service area
- Restaurant worker on a defined shift

**Design consequence:** Separate the reusable `JobTemplate` from the site-specific `JobOpening`.

### 2. Hiring volume is high and time-to-fill matters

Recruiters may process hundreds or thousands of applicants for similar jobs. The economic value comes from reducing manual screening, document chasing, and scheduling.

**Design consequence:** Build bulk operations, rule evaluation, automated reminders, status queues, and reusable policy templates before sophisticated matching.

### 3. Evidence matters more than narrative

For many roles, the employer primarily needs verified facts:

- License
- Certification
- Work authorization
- Background status
- Prior environment
- Availability
- Distance
- Transportation
- Age requirement where legally applicable
- Physical or safety requirements stated in a lawful, job-related way

**Design consequence:** Use an evidence ledger and requirement matrix rather than an open-ended candidate profile.

### 4. Worker supply is fragmented

Workers may come from:

- Direct application
- Referrals
- Staffing agencies
- Job boards
- Community groups
- Workforce programs
- Gig platforms
- Employer rehire pools
- SMS or messaging campaigns

**Design consequence:** Source provenance, duplicate resolution, portable worker profiles, and channel-specific consent matter.

### 5. Worker trust and accessibility are critical

Many workers will use a phone, have limited time, and may not be comfortable with complex forms. They may also be skeptical of automated outreach or background checks.

**Design consequence:** Use mobile-first, multilingual, low-bandwidth flows with plain-language explanations, save-and-resume, voice or SMS options where permitted, and a human escalation path.

### 6. Interviews may be low-value for some roles

For a licensed barber or forklift operator, a validated credential and skills check may be more predictive than a conversational interview. For customer-facing roles, a short structured interview may still matter.

**Design consequence:** Model interview as an optional branch:

- No interview
- Structured asynchronous interview
- AI-led interview with disclosure
- Third-party interview
- Employer interview
- Skills demonstration or paid trial, where lawful and appropriate

### 7. Hiring decisions are often conditional

A worker may be eligible subject to:

- Background completion
- License verification
- Drug or safety screening where lawful
- Orientation
- Training
- Site-specific paperwork

**Design consequence:** Support `conditional_offer`, `pending_clearance`, and `onboarding_required` states instead of only hired/rejected.

---

# 1. List of Data Schemas

## 1.1 `Employer`

**Purpose:** Organization that hires workers across one or more locations.

**Core fields**

- `employer_id`
- `legal_name`
- `display_name`
- `industry`
- `size_band`
- `billing_account_id`
- `policy_pack_id`
- `status`
- `created_at`

**Design decision:** Employer-wide hiring policies are separate from individual job openings so they can be versioned and reused.

## 1.2 `Site`

**Purpose:** Physical or operational location where work occurs.

**Core fields**

- `site_id`
- `employer_id`
- `name`
- `address`
- `latitude`
- `longitude`
- `timezone`
- `operating_hours`
- `contact_person_id`
- `safety_requirements`
- `status`

**Design decision:** Site fit is a first-class constraint for location-bound work.

## 1.3 `HiringUser`

**Purpose:** Recruiter, manager, franchise owner, or operations leader.

**Core fields**

- `hiring_user_id`
- `employer_id`
- `site_ids[]`
- `role`
- `permissions[]`
- `decision_authority`
- `status`

**Design decision:** An AI agent can act for a recruiter only within explicitly delegated authority.

## 1.4 `JobTemplate`

**Purpose:** Reusable definition of a recurring frontline job.

**Core fields**

- `job_template_id`
- `employer_id`
- `title`
- `job_family`
- `description`
- `standard_pay_range`
- `employment_types[]`
- `standard_requirements[]`
- `standard_interview_policy`
- `standard_background_policy`
- `standard_training_path`
- `version`
- `status`

**Examples**

- `barber`
- `forklift_operator`
- `delivery_driver`
- `warehouse_associate`
- `restaurant_shift_worker`

**Design decision:** Templates let a solo operator or employer configure a policy once and reuse it safely.

## 1.5 `JobOpening`

**Purpose:** Specific hiring need at a site, shift, or service area.

**Core fields**

- `job_opening_id`
- `job_template_id`
- `site_id`
- `title_override`
- `headcount_needed`
- `shift_definition`
- `start_date`
- `pay_override`
- `location_radius`
- `requirements_override[]`
- `interview_policy_override`
- `status`
- `opened_at`
- `closed_at`

**Design decision:** A worker may qualify for the job template but not for a specific opening because of shift, distance, pay, or start date.

## 1.6 `Requirement`

**Purpose:** Atomic requirement used by the eligibility engine.

**Core fields**

- `requirement_id`
- `job_template_id` or `job_opening_id`
- `category`: `license | certification | identity | authorization | background | experience | availability | location | transport | skill | language | training`
- `field`
- `operator`
- `expected_value`
- `required`
- `waivable`
- `waiver_authority`
- `evidence_type`
- `effective_date`
- `version`

**Examples**

```text
license.type = cosmetology
license.status = valid
experience.environment = warehouse
experience.duration_months >= 6
availability.overlap_hours >= 24
distance.miles <= 20
background.policy_status = eligible
```

**Design decision:** Requirements are atomic, typed, versioned, and explainable. Avoid hiding requirements in free text.

## 1.7 `Worker`

**Purpose:** Person who may apply, qualify, work, or be hired.

**Core fields**

- `worker_id`
- `person_id`
- `phone`
- `email`
- `preferred_language`
- `home_location`
- `transportation_options[]`
- `work_authorization_status`
- `communication_preferences`
- `lifecycle_status`
- `source`
- `created_at`

**Design decision:** Use `Worker` rather than only `Candidate` because the relationship continues after hiring into onboarding, shifts, and rehire eligibility.

## 1.8 `WorkerPreference`

**Purpose:** Worker constraints and preferences.

**Core fields**

- `worker_preference_id`
- `worker_id`
- `preferred_job_families[]`
- `preferred_sites[]`
- `max_commute_minutes`
- `available_days[]`
- `available_time_windows[]`
- `minimum_pay`
- `employment_types[]`
- `start_date`
- `transportation_constraints`
- `communication_preferences`
- `effective_from`

**Design decision:** Scheduling and location fit are part of matching, not late-stage recruiter notes.

## 1.9 `WorkerSkillRecord`

**Purpose:** Structured evidence of a worker’s skills and experience.

**Core fields**

- `skill_record_id`
- `worker_id`
- `skill_code`
- `skill_level`
- `environment_type`
- `duration_months`
- `last_used_at`
- `source`
- `verification_status`
- `evidence_id`

**Examples**

- `forklift_counterbalance`
- `haircut_clip_guard`
- `point_of_sale`
- `route_navigation`
- `warehouse_picking`
- `food_safety`

**Design decision:** Use controlled skill codes and environment tags to support objective rules and equivalent-experience policies.

## 1.10 `Credential`

**Purpose:** License, certification, permit, or training record.

**Core fields**

- `credential_id`
- `worker_id`
- `credential_type`
- `issuing_authority`
- `credential_number_token`
- `jurisdiction`
- `issued_at`
- `expires_at`
- `status`
- `verification_method`
- `verified_at`
- `evidence_id`

**Design decision:** Credential status should be independently verifiable and time-aware.

## 1.11 `IdentityAndAuthorization`

**Purpose:** Identity, work authorization, and required employment documents.

**Core fields**

- `record_id`
- `worker_id`
- `document_type`
- `verification_status`
- `verified_at`
- `expires_at`
- `verification_provider`
- `failure_reason_code`
- `evidence_id`

**Design decision:** Store verification status and limited result codes rather than unnecessary sensitive document content.

## 1.12 `BackgroundCheck`

**Purpose:** Track a lawful, authorized background-check process.

**Core fields**

- `background_check_id`
- `worker_id`
- `employer_id`
- `job_opening_id`
- `provider`
- `consent_at`
- `status`
- `result_category`
- `dispute_status`
- `adverse_action_stage`
- `completed_at`
- `retention_until`

**Important rule:** Do not store raw criminal-history details in the core worker profile. Store provider, authorization, status, policy result, dispute status, and retention metadata.

**Design decision:** Background-check decisions require a jurisdiction-aware policy and human/legal review path; they must not be treated as a simple AI score.

## 1.13 `Availability`

**Purpose:** Worker availability for specific shifts, dates, or recurring schedules.

**Core fields**

- `availability_id`
- `worker_id`
- `effective_from`
- `effective_to`
- `days_of_week`
- `start_time`
- `end_time`
- `timezone`
- `recurrence`
- `source`
- `last_confirmed_at`

**Design decision:** Availability should be machine-evaluable and periodically reconfirmed.

## 1.14 `Application`

**Purpose:** Worker’s application to a specific opening.

**Core fields**

- `application_id`
- `worker_id`
- `job_opening_id`
- `source`
- `status`
- `applied_at`
- `withdrawn_at`
- `current_stage`
- `owner_agent_id`
- `owner_human_id`

**Design decision:** One worker can have different eligibility outcomes across openings.

## 1.15 `Evidence`

**Purpose:** Provenance for a worker, credential, requirement, or decision.

**Core fields**

- `evidence_id`
- `entity_type`
- `entity_id`
- `evidence_type`
- `source`
- `source_locator`
- `captured_at`
- `confidence`
- `verification_status`
- `expires_at`

**Examples**

- Credential registry response
- Employer reference
- Prior payroll or work record
- Skills assessment
- Worker-provided document
- Structured interview response
- Training completion record

**Design decision:** Every eligibility result should be explainable through evidence.

## 1.16 `Consent`

**Purpose:** Record worker permission by purpose.

**Core fields**

- `consent_id`
- `worker_id`
- `purpose`
- `employer_id`
- `job_opening_id`
- `channel`
- `status`
- `notice_version`
- `captured_at`
- `withdrawn_at`

**Purposes**

- `contact_worker`
- `create_worker_record`
- `verify_credential`
- `run_background_check`
- `share_application`
- `conduct_interview`
- `share_interview_with_employer`
- `retain_for_future_roles`
- `send_shift_or_gig_opportunities`

**Design decision:** Optional interviews and outsourced interviews require their own clear consent and disclosure.

## 1.17 `EligibilityEvaluation`

**Purpose:** Evaluate a worker against a job opening.

**Core fields**

- `evaluation_id`
- `application_id`
- `job_opening_id`
- `worker_id`
- `policy_version`
- `status`: `eligible | ineligible | conditional | needs_evidence | human_review`
- `requirement_results[]`
- `missing_evidence[]`
- `disqualifying_requirements[]`
- `evaluated_by_agent_id`
- `evaluated_at`
- `human_reviewed_at`
- `decision_reason_code`

**Design decision:** Store every requirement result individually. Never store only “qualified: true.”

## 1.18 `InterviewPolicy`

**Purpose:** Define whether and how an interview is used.

**Core fields**

- `interview_policy_id`
- `job_template_id` or `job_opening_id`
- `mode`: `none | employer | async_structured | ai_led | third_party`
- `required_for`
- `disqualifying_answers`
- `scoring_rubric`
- `human_review_required`
- `worker_disclosure_text`
- `version`

**Design decision:** Interview is a policy branch, not a mandatory universal stage.

## 1.19 `InterviewAssignment`

**Purpose:** Track an interview owned by an employer, AI agent, or external provider.

**Core fields**

- `interview_assignment_id`
- `application_id`
- `mode`
- `provider`
- `consent_id`
- `status`
- `scheduled_at`
- `completed_at`
- `result_status`
- `scorecard_id`
- `review_required`

**Design decision:** Outsourced interviews are treated as a service and evidence source with a provider, SLA, cost, and data-sharing boundary.

## 1.20 `Scorecard`

**Purpose:** Structured assessment for interview or skill verification.

**Core fields**

- `scorecard_id`
- `application_id`
- `rubric_version`
- `criteria[]`
- `scores[]`
- `evidence[]`
- `recommendation`
- `created_by`
- `reviewed_by`

**Design decision:** Use structured, job-related criteria; do not use vague “culture fit” scoring.

## 1.21 `HiringDecisionPolicy`

**Purpose:** Employer-authorized rules for recommendations and decisions.

**Core fields**

- `policy_id`
- `employer_id`
- `job_template_id` or `job_opening_id`
- `rule_set`
- `automatic_actions_allowed[]`
- `human_review_triggers[]`
- `conditional_offer_rules[]`
- `adverse_action_workflow`
- `version`
- `approved_by`
- `approved_at`

**Examples of allowed actions**

- Request missing evidence
- Move to eligible
- Invite to application
- Make conditional-offer recommendation
- Route to human review
- Reject for a documented job-related reason

**Design decision:** The employer configures the policy; the AI evaluates against it and exposes the result.

## 1.22 `HiringRecommendation`

**Purpose:** AI-generated recommendation to the authorized recruiter or employer.

**Core fields**

- `recommendation_id`
- `application_id`
- `recommendation`: `advance | hold | request_evidence | conditional_offer | reject | human_review`
- `reasons[]`
- `evidence_ids[]`
- `policy_version`
- `confidence`
- `risk_flags[]`
- `generated_by_agent_id`
- `human_decision`
- `human_decided_at`

**Design decision:** Separate recommendation from final employment action so the system remains explainable and reviewable.

## 1.23 `Offer`

**Purpose:** Formal offer or conditional offer.

**Core fields**

- `offer_id`
- `application_id`
- `offer_type`: `conditional | final`
- `pay`
- `schedule`
- `start_date`
- `conditions[]`
- `expires_at`
- `status`
- `sent_at`
- `accepted_at`

**Design decision:** Conditions must be explicit and tied to evidence or onboarding tasks.

## 1.24 `OnboardingTask`

**Purpose:** Pre-start and first-shift completion.

**Core fields**

- `task_id`
- `worker_id`
- `job_opening_id`
- `task_type`
- `required`
- `status`
- `due_at`
- `completed_at`
- `evidence_id`
- `owner_agent_id`

**Examples**

- Complete orientation
- Submit payroll information
- Finish safety training
- Confirm first shift
- Complete site access paperwork
- Obtain uniform or equipment

**Design decision:** A successful hiring product must own the transition from “selected” to “ready to work.”

## 1.25 `Placement` and `WorkOutcome`

**Purpose:** Record actual start, retention, and operational outcome.

**Core fields**

`Placement`

- `placement_id`
- `application_id`
- `worker_id`
- `job_opening_id`
- `start_date`
- `status`
- `source`

`WorkOutcome`

- `work_outcome_id`
- `placement_id`
- `first_shift_completed`
- `days_7_status`
- `days_30_status`
- `attendance_status`
- `employer_feedback`
- `worker_feedback`
- `ended_at`
- `end_reason_code`

**Design decision:** Optimize for successful starts and early retention, not only offers.

## 1.26 `Communication` and `Suppression`

**Purpose:** Manage worker and employer messages safely.

**Core fields**

- `communication_id`
- `recipient_type`
- `recipient_id`
- `channel`
- `purpose`
- `agent_identity_id`
- `consent_id`
- `message_template_version`
- `status`
- `sent_at`
- `reply_at`
- `opt_out_at`

`Suppression` includes:

- `person_id`
- `channels`
- `scope`
- `reason`
- `created_at`

**Design decision:** Worker communication must be mobile-first and easy to stop.

## 1.27 `Agent`, `AgentTask`, and `AuditEvent`

**Purpose:** Operate the fully AI-native company with human accountability.

**Agent fields**

- `agent_id`
- `purpose`
- `version`
- `allowed_actions[]`
- `approval_policy`
- `tool_permissions[]`
- `active`

**Agent task fields**

- `task_id`
- `agent_id`
- `entity_type`
- `entity_id`
- `status`
- `confidence`
- `risk_flags[]`
- `recommendation`
- `human_decision`
- `created_at`

**Audit fields**

- `audit_event_id`
- `actor_type`
- `actor_id`
- `action`
- `entity_type`
- `entity_id`
- `policy_version`
- `reason`
- `occurred_at`

**Design decision:** Any automated hiring recommendation must be attributable to an agent version, policy version, evidence set, and human decision where required.

---

# 2. List of Business Processes

## 2.1 Define the job template and hiring policy

**Purpose:** Convert a recurring frontline role into objective, lawful, structured rules.

**AI agent**

- Drafts the role definition from employer materials.
- Extracts skills, credentials, schedule, site, and experience requirements.
- Identifies ambiguous or potentially risky requirements.
- Suggests equivalent-experience rules.
- Generates a candidate-facing explanation.

**Human lead**

- Employer or authorized recruiter approves job requirements.
- Founder or implementation operator checks that requirements are job-related and operationally realistic.
- The system administrator approves the decision policy and human-review triggers.

**System output**

- `JobTemplate`
- `Requirement[]`
- `InterviewPolicy`
- `HiringDecisionPolicy`

**Example: forklift operator**

```text
Required:
  - valid forklift certification or employer-approved training path
  - ability to work the required shift
  - work authorization
  - safety policy clearance

Preferred:
  - 6+ months warehouse experience
  - experience in high-volume distribution environment
```

**Done when:** Every requirement has a type, evidence source, rule, version, and escalation path.

## 2.2 Create a site-specific job opening

**Purpose:** Apply a reusable template to a site, shift, and headcount need.

**AI agent**

- Creates opening from template.
- Applies location, pay, shift, start date, and headcount.
- Checks whether requirements are missing.
- Drafts the job post and worker-facing summary.

**Human lead**

- Confirms headcount, pay, schedule, site conditions, and decision authority.
- Activates the opening.

**System output**

- `JobOpening`
- Site-specific requirements
- Recruiting campaign task

**Done when:** A worker can determine whether the job is feasible before applying.

## 2.3 Source and invite workers

**Purpose:** Build a qualified applicant pool.

**AI agent**

- Searches approved worker sources.
- Deduplicates worker records.
- Identifies likely matches.
- Sends approved invitations or routes workers to the application flow.
- Personalizes messages using only verified, relevant information.

**Human lead**

- Approves source mix and outreach policy.
- Reviews campaigns and unusual candidate segments.

**System controls**

- Consent and suppression checks before every contact.
- Clear employer, role, pay, shift, location, and AI disclosure.
- No automatic enrollment without a valid basis.

**Done when:** Outreach produces applications without creating an untraceable worker database.

## 2.4 Worker application and mobile intake

**Purpose:** Collect enough information to evaluate the worker quickly.

**AI agent**

- Runs a mobile-first conversational intake.
- Asks only questions needed for the opening.
- Captures availability, location, experience, credentials, transportation, and preferences.
- Detects missing or contradictory answers.
- Offers language or accessibility support.

**Human lead**

- Handles workers who need help or challenge a question.
- Resolves unusual evidence.

**System output**

- `Worker`
- `Application`
- `WorkerPreference`
- Initial evidence requests

**Done when:** A worker can complete the first screen without a resume for roles where a resume adds little value.

## 2.5 Credential and document verification

**Purpose:** Confirm licenses, certifications, identity, work authorization, and required documents.

**AI agent**

- Requests documents or registry details.
- Connects to approved verification providers.
- Checks expiration and jurisdiction.
- Matches the document to the worker.
- Routes failures and ambiguity.

**Human lead**

- Reviews exceptions.
- Handles worker disputes.
- Approves equivalent evidence or training pathways where policy permits.

**System controls**

- Store minimal necessary document data.
- Tokenize credential numbers.
- Track expiration and re-verification.
- Keep raw background information out of the worker profile.

**Done when:** Each requirement has `verified`, `unverified`, `expired`, `not_applicable`, or `needs_human_review`.

## 2.6 Experience and environment verification

**Purpose:** Determine whether the worker has done similar work in a relevant environment.

**AI agent**

- Extracts prior employers, job types, environment, duration, and recency.
- Maps free-text experience to controlled skill and environment codes.
- Requests references, proof, or structured attestations.
- Identifies equivalent experience.

**Human lead**

- Defines accepted equivalents.
- Reviews conflicting or unusual experience.

**System output**

- `WorkerSkillRecord`
- `Evidence`
- Environment-fit result

**Done when:** “Similar experience” has a defined rule rather than a subjective recruiter impression.

## 2.7 Schedule, site, and transport matching

**Purpose:** Determine whether the worker can realistically work the opening.

**AI agent**

- Compares availability to shift requirements.
- Calculates travel distance or estimated commute.
- Checks transportation constraints.
- Proposes alternative sites or shifts.

**Human lead**

- Approves lawful, job-related transportation rules.
- Handles exceptions and relocation cases.

**System output**

- Availability fit
- Site fit
- Recommended opening alternatives

**Done when:** The system can reject a mismatch before a recruiter spends time interviewing or scheduling.

## 2.8 Background-check workflow

**Purpose:** Complete an employer-authorized screening process.

**AI agent**

- Obtains required consent.
- Initiates the provider workflow.
- Tracks status.
- Notifies the worker of next steps.
- Routes report status to the appropriate policy workflow.

**Human lead**

- Handles disputes, adverse-action notices, and jurisdiction-specific exceptions.
- Does not ask the AI to independently interpret raw criminal history.

**System controls**

- Separate authorization and disclosure.
- Apply jurisdiction-aware policy.
- Provide pre-adverse and adverse-action workflow where required.
- Keep background status separate from general worker profile.

**Done when:** Background status can move an application to clear, pending, dispute, or human review without exposing unnecessary personal history.

## 2.9 Deterministic eligibility evaluation

**Purpose:** Evaluate the application against the job’s requirement matrix.

**AI agent**

- Executes typed rules.
- Produces one result per requirement.
- Identifies missing evidence and conflicts.
- Creates `EligibilityEvaluation`.

**Human lead**

- Reviews exceptions, waivers, and ambiguous evidence.
- Approves policy changes.

**System output**

- `eligible`
- `ineligible`
- `conditional`
- `needs_evidence`
- `human_review`

**Example result**

```text
License: verified
Work authorization: verified
Shift availability: pass
Distance: pass
Prior warehouse environment: pass
Background policy: pending
Overall: conditional
```

**Done when:** The founder or recruiter can explain the result requirement by requirement.

## 2.10 Choose the interview path

**Purpose:** Decide whether an interview is necessary and which party owns it.

**Decision inputs**

- Job template
- Employer policy
- Role risk
- Worker evidence completeness
- Customer-facing requirements
- Safety sensitivity
- Local policy
- Worker preference

**Available branches**

1. `no_interview`
2. `employer_interview`
3. `async_structured_interview`
4. `ai_led_interview`
5. `third_party_interview`
6. `skills_assessment`
7. `paid_trial_or_work_sample`, where lawful and appropriate

**AI agent**

- Recommends the lowest-friction valid path.
- Explains why an interview is or is not needed.
- Schedules the chosen path.

**Human lead**

- Employer approves interview policy.
- Founder or recruiter handles exceptions.

**Done when:** Interview is a deliberate policy choice, not an accidental default.

## 2.11 No-interview hiring path

**Purpose:** Hire workers based on verified objective criteria when the employer policy permits.

**AI agent**

- Confirms all required evidence.
- Checks decision policy.
- Generates a hiring recommendation or conditional-offer recommendation.
- Produces a plain-language decision explanation.
- Routes to employer approval if required.

**Human lead**

- Recruiter or authorized employer approves the final action unless the employer has explicitly delegated automated execution where permitted.

**System controls**

- Store policy and version.
- Record evidence used.
- Prevent use of prohibited or hidden features.
- Create a review path for worker challenge or correction.

**Output**

- `advance`
- `conditional_offer`
- `hold`
- `request_evidence`
- `human_review`
- `reject`

**Done when:** The employer can hire without an interview and still explain the decision.

## 2.12 AI-led structured interview path

**Purpose:** Run a consistent, optional interview when the role needs additional evidence.

**AI agent**

- Discloses that the interview is AI-led.
- Asks only approved, job-related questions.
- Supports multiple languages and accessible formats.
- Transcribes responses where permitted.
- Scores only against the approved rubric.
- Identifies uncertainty and routes to human review.

**Human lead**

- Approves questions and scoring rubric.
- Reviews borderline outcomes.
- Handles accommodations and complaints.

**System controls**

- Worker consent before recording or processing.
- No personality or emotion inference.
- No unapproved facial, voice, accent, or sentiment scoring.
- Retain original answers and scoring evidence.

**Done when:** The interview adds job-related evidence without becoming an opaque personality filter.

## 2.13 Outsourced interview path

**Purpose:** Use a third-party interview or assessment provider.

**AI agent**

- Selects an approved provider based on role and policy.
- Sends worker consent and instructions.
- Tracks SLA and completion.
- Imports structured result and evidence.
- Flags provider quality issues.

**Human lead**

- Approves provider and commercial terms.
- Reviews exceptions and disputed results.

**System controls**

- Store provider, rubric version, consent, data-sharing scope, and retention period.
- Do not silently pass data to a third party.
- Support worker correction or dispute.

**Done when:** A third-party result can be treated as one evidence source—not an unquestionable truth.

## 2.14 Employer-authorized hiring recommendation

**Purpose:** Let the product recommend or prepare the hiring decision.

**AI agent**

- Combines eligibility, evidence, interview or assessment result, schedule fit, and employer policy.
- Generates `HiringRecommendation`.
- States the exact reasons and missing conditions.
- Prepares offer or rejection communication.

**Human lead**

- Recruiter or hiring manager approves the decision.
- Founder handles unusual cases, policy disputes, and escalations.

**System controls**

- Separate recommendation from final decision.
- Require human review for defined triggers:
  - conflicting evidence;
  - adverse background result;
  - accommodation request;
  - policy waiver;
  - missing consent;
  - low confidence;
  - worker dispute;
  - safety-critical role.

**Done when:** The employer sees a decision packet rather than a black-box score.

## 2.15 Conditional offer and clearance

**Purpose:** Move an eligible worker toward a start date while remaining transparent about remaining conditions.

**AI agent**

- Drafts conditional offer.
- Lists outstanding conditions.
- Sends reminders.
- Tracks completion and expiration.

**Human lead**

- Approves terms and conditions.
- Handles negotiation and exceptions.

**System output**

- `Offer`
- Clearance tasks
- Start-date readiness score

**Done when:** Worker knows exactly what remains before starting.

## 2.16 Onboarding and first-shift readiness

**Purpose:** Convert a hire into a worker who actually starts.

**AI agent**

- Collects payroll and onboarding information.
- Schedules orientation.
- Sends location, uniform, equipment, and shift instructions.
- Confirms first-shift attendance.
- Escalates no-response or confusion.

**Human lead**

- Handles site-specific problems, accommodations, and no-show recovery.

**System output**

- `OnboardingTask`
- Readiness status
- First-shift confirmation

**Done when:** The system knows whether the worker is ready, not merely hired.

## 2.17 Post-hire retention and rehire

**Purpose:** Learn whether the hiring decision created a successful placement.

**AI agent**

- Checks first shift, day 7, and day 30 outcomes.
- Collects worker and employer feedback.
- Detects early attrition patterns.
- Recommends rehire eligibility or retraining.

**Human lead**

- Reviews systemic issues.
- Changes job requirements, source policy, or onboarding.

**System output**

- `WorkOutcome`
- Retention metrics
- Updated worker eligibility and preferences

**Done when:** Hiring quality is measured by successful starts and retention, not only selection speed.

## 2.18 Worker correction, appeal, and adverse-action handling

**Purpose:** Give workers a clear path to challenge incorrect or incomplete information.

**AI agent**

- Explains the decision in plain language.
- Identifies the requirement that failed.
- Accepts corrected evidence.
- Routes disputes to the correct human.

**Human lead**

- Reviews disputes and adverse-action steps.
- Applies employer and jurisdiction policy.

**System controls**

- Preserve original decision evidence.
- Do not erase the reason for a decision after correction.
- Pause irreversible rejection where policy requires review.

**Done when:** A worker can understand, correct, and challenge an eligibility or hiring result.

## 2.19 Daily workforce operations

**Purpose:** Give the recruiter, manager, or solopreneur a decision-oriented operating view.

**AI agent**

- Prioritizes applicants by action needed.
- Groups missing evidence.
- Detects expiring credentials.
- Highlights roles at risk of not filling.
- Identifies no-shows and stalled onboarding.
- Monitors agent errors and fairness-risk signals.

**Human lead**

- Approves exceptions.
- Contacts high-value or at-risk workers.
- Changes policy and staffing priorities.

**System output**

- Exception queue
- Role fill forecast
- Clearance queue
- Interview queue
- Onboarding queue
- Trust and compliance alerts

---

# Founder and recruiter dashboards

## Dashboard 1 — Hiring command center

- Open roles by urgency
- Applicants awaiting eligibility
- Workers missing one document
- Conditional offers at risk
- Interviews awaiting completion
- First shifts tomorrow
- No-shows and stalled onboarding
- Human decisions required

## Dashboard 2 — Rule evaluation

- Pass / fail / conditional by requirement
- Most common failure reasons
- Expiring licenses and certifications
- Missing evidence by source
- Waivers and overrides
- Requirements producing the most disputes

## Dashboard 3 — Interview operations

- No-interview hires
- AI interview completion
- Third-party interview SLA
- Employer interview backlog
- Borderline scores
- Worker interview opt-outs
- Accommodation and review requests

## Dashboard 4 — Hiring quality

- Time from application to decision
- Time from decision to start
- First-shift completion
- Day 7 and day 30 retention
- No-show rate
- Early attrition by source, site, shift, and requirement set
- Worker and employer satisfaction

## Dashboard 5 — Trust and fairness

- Consent completion
- Opt-out rate
- Complaint rate
- Repeat-contact violations
- Decision overrides
- Appeals and corrections
- Adverse-action workflow status
- Outcome differences by job-relevant segments approved for monitoring

Do not expose protected attributes to the decision agent simply because they are available for analytics. Any fairness monitoring should be designed with appropriate legal and privacy review.

---

# Recommended MVP for this segment

## Phase 1 — Qualification engine

Build:

- `Employer`
- `Site`
- `JobTemplate`
- `JobOpening`
- `Requirement`
- `Worker`
- `WorkerPreference`
- `Credential`
- `Availability`
- `Application`
- `Evidence`
- `Consent`
- `EligibilityEvaluation`
- `Suppression`
- `AuditEvent`

Agents:

- Job Definition Agent
- Worker Intake Agent
- Evidence Collection Agent
- Eligibility Evaluation Agent
- Founder Operations Agent

Product experience:

```text
Job opening
  → worker application
  → evidence collection
  → deterministic evaluation
  → eligible / conditional / needs evidence / review
```

## Phase 2 — Decision and interview branching

Build:

- `InterviewPolicy`
- `InterviewAssignment`
- `Scorecard`
- `HiringDecisionPolicy`
- `HiringRecommendation`
- `Offer`

Agents:

- Interview Orchestration Agent
- Structured Interview Agent
- Hiring Recommendation Agent

Product experience:

```text
Eligible
  ├─ no interview → hiring recommendation
  ├─ AI interview → scorecard → hiring recommendation
  ├─ third-party interview → imported evidence → hiring recommendation
  └─ employer interview → recruiter decision
```

## Phase 3 — Start and retention

Build:

- `OnboardingTask`
- `Placement`
- `WorkOutcome`
- First-shift and day-30 dashboards

Agents:

- Offer and Onboarding Agent
- First-Shift Readiness Agent
- Retention Agent

## Suggested initial pilot

- One job family
- One employer or franchise network
- One or two sites
- One geography
- One approved verification provider
- One no-interview policy
- One optional interview policy
- 50–200 applications
- Human approval for every hiring recommendation

Start with a role such as **forklift operator or licensed barber**, where the requirements can be made concrete. Avoid starting with a broad gig marketplace; the operational variance will hide whether the qualification engine is actually working.

---

# Core product conclusion

For this market, the winning product is not an AI recruiter that writes better messages. It is a **qualification, decision, and readiness engine**:

```text
Job policy
  → evidence collection
  → rule evaluation
  → optional assessment/interview
  → employer-authorized hiring recommendation
  → conditional offer
  → onboarding
  → successful first shift
  → retention and rehire
```

The strongest defensibility comes from:

- A reusable ontology of frontline skills and environments
- Verified credential and evidence workflows
- Employer-specific decision policies
- Low-friction worker communication
- Explainable, appealable decisions
- Better prediction of first-shift completion and early retention
