# AI-Native Talent Marketplace — Operating Model

## Operating premise

The product is modeled here as a **solopreneur-led, AI-native talent agency and two-sided marketplace**.

- The founder is the accountable human for judgment, relationships, reputation, legal/commercial decisions, and exceptions.
- Every recurring business process has an AI agent that prepares, executes, monitors, or recommends work.
- Agents may act autonomously inside explicit policy limits, but consequential external actions remain approval-gated.
- The system is designed around evidence, consent, explainability, and an auditable chain from candidate source to paid placement.

The product is therefore not just “AI matching.” It is a **human-led operating system for candidate representation, employer recruiting, introductions, and success-fee revenue**.

**Market-analysis boundary:** Cleara is treated only as a competitor and market reference. The operating model itself is brand-neutral.

---

## Underlying market assumptions

These assumptions are inferred from market research and materially shape the design. They are not claims about any competitor’s internal business.

### A. Kind of jobs

The model assumes the product primarily serves:

- Startup and scale-up knowledge-work roles
- Engineering, product, design, operations, growth, and technical business roles
- Roles where a strong candidate profile and warm introduction matter more than raw application volume
- Roles with meaningful compensation and a success fee large enough to support human-led recruiting economics
- Roles that can be evaluated through work history, skills, projects, seniority, motivation, and founder/team fit

It does **not** assume the initial product is optimized for:

- High-volume hourly hiring
- Seasonal or shift-based labor
- Government roles with formal procurement workflows
- Highly credentialed professions where licensing is the primary filter
- Large enterprise recruiting with complex ATS, procurement, and multi-stakeholder approvals

### B. Kind of candidates

The model assumes candidates are:

- Experienced or high-signal individual contributors, managers, and early executives
- Comfortable with digital communication and asynchronous workflows
- Discoverable through professional profiles, referrals, resumes, portfolios, or prior conversations
- Interested in selective opportunities rather than applying indiscriminately
- Willing to share enough context for representation if trust is established

The product is less naturally suited to candidates who:

- Need intensive career coaching before they can be represented
- Have sparse or difficult-to-verify digital work history
- Need local, in-person, or regulated placement support
- Require high-touch accessibility or language support that a solo operator cannot provide without additional service design

### C. Kind of job market

The model assumes a market with these characteristics:

- Startup-heavy and founder-led
- Competitive for strong technical and product talent
- Geographically distributed, especially US, UK, EU, and remote-friendly hubs
- Fast-moving enough that speed-to-introduction is valuable
- Trust-sensitive because unsolicited recruiting outreach is easily confused with scams
- Willing to transact on contingency or success-fee terms

### D. Commercial assumptions

The model assumes:

- Candidates are free to use the service.
- Employers pay only when a placement occurs, or under a closely related success-fee arrangement.
- The founder needs to maximize placements per unit of human time.
- Revenue is lumpy, delayed, and dependent on accurate attribution.
- One founder cannot personally manage every candidate, employer, message, interview, and invoice.

### E. Design consequences

These assumptions lead to the following design decisions:

| Assumption | Design decision |
|---|---|
| High-signal startup roles | Use structured evidence and narrative fit, not keyword matching alone. |
| Selective candidates | Make representation scope, preferences, and consent first-class data. |
| Fast-moving market | Use event-driven workflows, queues, reminders, and agent follow-up. |
| Scam-sensitive outreach | Make sender identity, AI disclosure, source, and opt-out visible in every contact. |
| Success-fee economics | Track attribution from match to introduction to hire to invoice. |
| One human operator | Agents batch routine work; the founder sees exceptions and decisions, not raw activity. |
| Small initial market | Support manual overrides and flexible schemas before building a generalized marketplace. |
| Cross-border candidates | Store jurisdiction, work authorization, privacy basis, currency, time zone, and data residency signals. |
| Trust as the product moat | Measure complaints, opt-outs, response quality, and consent completeness alongside revenue. |

---

# 1. List of Data Schemas

The schemas below are the minimum coherent data model for the assumed business. Each schema includes its purpose, core fields, and the AI-native design implication.

## 1.1 `Person`

**Purpose:** Canonical human identity shared across candidate, employer, and operator contexts.

**Core fields**

- `person_id`
- `name`
- `email_addresses[]`
- `phone_numbers[]`
- `timezone`
- `country`
- `preferred_language`
- `identity_status`
- `created_at`
- `updated_at`

**AI-native decision:** Identity resolution must be deterministic before any agent sends a message or merges records. Agents may suggest duplicates; the founder or a verified rule approves merges.

## 1.2 `Candidate`

**Purpose:** A person who may be discoverable, engaged, represented, or suppressed.

**Core fields**

- `candidate_id`
- `person_id`
- `lifecycle_status`: `discovered | contacted | engaged | consented | represented | paused | suppressed | deleted`
- `source_type`
- `source_reference`
- `candidate_type`: `individual_contributor | manager | executive | founder | contractor`
- `seniority_band`
- `primary_function`
- `location`
- `work_authorization`
- `availability`
- `data_quality_status`
- `owner_agent_id`
- `human_owner_id`

**AI-native decision:** A discovered person is not automatically an active candidate. The schema separates discovery from consented representation.

## 1.3 `CandidateProfile`

**Purpose:** Versioned professional representation used for matching and employer presentation.

**Core fields**

- `profile_id`
- `candidate_id`
- `version`
- `headline`
- `summary`
- `skills[]`
- `work_history[]`
- `education[]`
- `projects[]`
- `achievements[]`
- `role_preferences[]`
- `company_preferences[]`
- `location_preferences[]`
- `compensation_expectation`
- `availability`
- `profile_visibility`
- `generated_by`: `candidate | operator | ai | imported`
- `candidate_approved_at`
- `effective_from`
- `effective_to`

**AI-native decision:** Agents draft and refresh profiles, but important claims need source evidence and candidate approval before external sharing.

## 1.4 `Evidence`

**Purpose:** Source-level support for a candidate, role, company, or outcome claim.

**Core fields**

- `evidence_id`
- `entity_type`
- `entity_id`
- `field_path`
- `source_type`: `candidate | resume | public_profile | portfolio | employer | operator | ai_inference`
- `source_locator`
- `captured_at`
- `confidence`
- `verification_status`
- `retention_until`

**AI-native decision:** The AI cannot turn an inference into a fact without preserving the source and confidence behind it.

## 1.5 `CandidatePreference`

**Purpose:** Explicit candidate preferences that constrain matching.

**Core fields**

- `preference_id`
- `candidate_id`
- `preferred_functions[]`
- `preferred_seniority`
- `preferred_company_stages[]`
- `preferred_industries[]`
- `location_policy`
- `location_values[]`
- `compensation_floor`
- `employment_types[]`
- `visa_or_authorization_constraints`
- `deal_breakers[]`
- `communication_preferences`
- `effective_from`
- `effective_to`

**AI-native decision:** Preferences are structured constraints, not only free-text conversation memory.

## 1.6 `RepresentationAgreement`

**Purpose:** Defines whether and how the platform may represent a candidate.

**Core fields**

- `representation_id`
- `candidate_id`
- `scope`: `general_marketplace | specific_role | specific_company`
- `status`: `pending | active | paused | revoked | expired`
- `terms_version`
- `start_at`
- `end_at`
- `revoked_at`
- `revocation_reason`
- `approved_by_candidate_at`

**AI-native decision:** Agents cannot submit or represent a candidate outside the active scope.

## 1.7 `ConsentRecord`

**Purpose:** Purpose-bound permission and communication preferences.

**Core fields**

- `consent_id`
- `candidate_id`
- `purpose`: `contact | create_profile | represent | share_profile | specific_opportunity | research_public_profile`
- `legal_basis`
- `status`: `granted | denied | withdrawn | expired | needs_review`
- `allowed_channels[]`
- `allowed_employers[]`
- `allowed_roles[]`
- `notice_version`
- `captured_at`
- `captured_from`
- `evidence_id`
- `expires_at`

**AI-native decision:** Consent is checked immediately before every external contact and every employer share. A single boolean is insufficient.

## 1.8 `Company`

**Purpose:** Employer organization that may submit roles and pay for placements.

**Core fields**

- `company_id`
- `organization_name`
- `domain`
- `website`
- `description`
- `sector`
- `company_stage`
- `headcount_band`
- `locations`
- `investor_or_network_tags[]`
- `verification_status`
- `account_status`
- `relationship_owner_agent_id`
- `human_owner_id`

**AI-native decision:** The company record supports automated research, but the founder approves commercial relationships and reputation-sensitive exceptions.

## 1.9 `EmployerUser`

**Purpose:** Individual employer contact and permission boundary.

**Core fields**

- `employer_user_id`
- `company_id`
- `person_id`
- `role`: `founder | hiring_manager | recruiter | advisor | viewer`
- `permissions[]`
- `verified_at`
- `status`

**AI-native decision:** Agents can communicate with a company but cannot assume every contact can approve a hire or fee.

## 1.10 `Role`

**Purpose:** Open hiring requirement submitted or maintained by an employer.

**Core fields**

- `role_id`
- `company_id`
- `title`
- `function`
- `seniority`
- `description`
- `employment_type`
- `location_policy`
- `locations[]`
- `compensation_range`
- `target_start_date`
- `visibility`: `named | confidential | anonymized`
- `status`: `draft | active | paused | filled | closed`
- `hiring_process`
- `fee_agreement_id`
- `created_by`
- `activated_at`

**AI-native decision:** The agent converts unstructured job descriptions into a structured brief, but a human verifies the requirements that determine candidate fit.

## 1.11 `RoleRequirement`

**Purpose:** Machine-readable requirements used for matching and explanation.

**Core fields**

- `requirement_id`
- `role_id`
- `category`: `must_have | nice_to_have | constraint | signal`
- `field`
- `operator`
- `value`
- `weight`
- `human_verified`
- `evidence_required`

**AI-native decision:** Hard constraints and soft signals are separated so an agent does not hide a disqualifier inside a blended score.

## 1.12 `Agent`

**Purpose:** Registry of AI workers that perform bounded business responsibilities.

**Core fields**

- `agent_id`
- `name`
- `purpose`
- `agent_type`
- `version`
- `allowed_actions[]`
- `prohibited_actions[]`
- `approval_policy`
- `tool_permissions[]`
- `prompt_or_policy_version`
- `active`
- `human_owner_id`

**Example agents**

- `CandidateResearchAgent`
- `ProfileBuilderAgent`
- `OutreachAgent`
- `RoleIntakeAgent`
- `MatchingAgent`
- `CandidateRelationshipAgent`
- `EmployerRelationshipAgent`
- `InterviewFollowupAgent`
- `PlacementAttributionAgent`
- `BillingAgent`
- `TrustMonitoringAgent`

**AI-native decision:** Every agent has an explicit action boundary. “AI-generated” is not a sufficient operating model.

## 1.13 `AgentTask`

**Purpose:** Unit of work assigned to an AI agent or escalated to the founder.

**Core fields**

- `task_id`
- `agent_id`
- `task_type`
- `entity_type`
- `entity_id`
- `priority`
- `status`: `queued | running | awaiting_human | approved | rejected | completed | failed`
- `inputs`
- `recommendation`
- `confidence`
- `risk_flags[]`
- `human_decision`
- `human_decided_by`
- `human_decided_at`
- `created_at`
- `completed_at`

**AI-native decision:** The founder operates from a prioritized exception queue instead of reviewing every successful automated action.

## 1.14 `AgentIdentity`

**Purpose:** Defines who or what appears to communicate externally.

**Core fields**

- `agent_identity_id`
- `display_name`
- `identity_type`: `human | ai_disclosed | ai_assisted_human`
- `organization_id`
- `channels[]`
- `sender_domains[]`
- `disclosure_text`
- `verification_status`
- `active`

**AI-native decision:** Sender identity, AI disclosure, and domain ownership are data, not presentation details.

## 1.15 `OutreachCampaign`

**Purpose:** Defines a bounded audience, purpose, message policy, and sender.

**Core fields**

- `campaign_id`
- `purpose`
- `audience_definition`
- `source_policy`
- `consent_policy`
- `agent_identity_id`
- `template_version`
- `frequency_limit`
- `opt_out_behavior`
- `status`
- `approved_by_human`

**AI-native decision:** Agents may personalize messages only within an approved campaign policy.

## 1.16 `Outreach`

**Purpose:** Immutable record of each candidate or employer contact.

**Core fields**

- `outreach_id`
- `campaign_id`
- `candidate_id`
- `company_id`
- `role_id`
- `channel`
- `destination`
- `purpose`
- `consent_record_id`
- `agent_identity_id`
- `message_id`
- `status`
- `sent_at`
- `delivered_at`
- `replied_at`
- `opted_out_at`

**AI-native decision:** Every message is attributable to a purpose, agent, campaign, sender, and permission check.

## 1.17 `Conversation` and `Message`

**Purpose:** Shared communication history across candidate and employer relationships.

**Core fields**

- `conversation_id`
- `participant_ids[]`
- `candidate_id`
- `company_id`
- `role_id`
- `channel`
- `state`
- `assigned_agent_id`
- `human_escalation_status`
- `last_message_at`

Each `Message` includes:

- `sender_type`
- `agent_identity_id`
- `ai_generated`
- `disclosure_shown`
- `body`
- `template_version`
- `delivery_status`
- `sent_at`

**AI-native decision:** The conversation is the agent’s working memory, but material decisions are also written into structured records.

## 1.18 `Match`

**Purpose:** Candidate-role compatibility assessment.

**Core fields**

- `match_id`
- `candidate_id`
- `role_id`
- `status`
- `score`
- `score_version`
- `hard_constraint_result`
- `match_reasons[]`
- `match_risks[]`
- `model_run_id`
- `candidate_visibility`
- `expires_at`

**AI-native decision:** The match stores explanations and risks, not only a numeric score.

## 1.19 `CandidatePacket`

**Purpose:** Controlled representation of a candidate shared for a specific opportunity.

**Core fields**

- `packet_id`
- `match_id`
- `candidate_id`
- `role_id`
- `version`
- `fields_shared`
- `redactions`
- `visibility`
- `candidate_approved_at`
- `shared_at`
- `expires_at`

**AI-native decision:** The system generates a role-specific packet rather than exposing the entire candidate profile.

## 1.20 `InterestSignal`

**Purpose:** Records candidate and employer interest independently.

**Core fields**

- `interest_signal_id`
- `match_id`
- `actor_type`
- `actor_id`
- `signal`: `interested | not_interested | needs_more_info | snooze`
- `reason`
- `captured_at`

**AI-native decision:** The system cannot infer mutual interest from silence, profile views, or a match score.

## 1.21 `Introduction`

**Purpose:** Formal record of a mutually approved introduction.

**Core fields**

- `introduction_id`
- `match_id`
- `candidate_id`
- `company_id`
- `role_id`
- `candidate_interest_signal_id`
- `employer_interest_signal_id`
- `method`
- `status`
- `intro_message`
- `sent_at`
- `accepted_at`

**AI-native decision:** The introduction is the commercial attribution boundary.

## 1.22 `Interview` and `HiringOutcome`

**Purpose:** Track progression and feedback after an introduction.

**Core fields**

- `interview_id`
- `introduction_id`
- `stage`
- `scheduled_at`
- `status`
- `candidate_feedback`
- `employer_feedback`

`HiringOutcome` includes:

- `outcome_id`
- `introduction_id`
- `status`: `unknown | rejected | withdrew | offer | hired | contracted`
- `start_date`
- `compensation`
- `reported_by`
- `reported_at`
- `evidence_id`

**AI-native decision:** Follow-up agents collect status, but the founder handles conflicting reports and sensitive feedback.

## 1.23 `FeeAgreement`, `Placement`, and `Invoice`

**Purpose:** Convert hiring outcomes into attributable revenue.

**Core fields**

`FeeAgreement`

- `fee_agreement_id`
- `company_id`
- `fee_type`
- `percentage_of_salary`
- `flat_amount`
- `replacement_window_days`
- `terms_version`
- `accepted_at`

`Placement`

- `placement_id`
- `hiring_outcome_id`
- `candidate_id`
- `company_id`
- `role_id`
- `attribution_status`
- `guarantee_ends_at`
- `fee_agreement_id`

`Invoice`

- `invoice_id`
- `placement_id`
- `amount`
- `currency`
- `status`
- `issued_at`
- `due_at`
- `paid_at`

**AI-native decision:** Billing is triggered by evidence-backed outcomes, not by an agent’s assumption that a candidate was hired.

## 1.24 `ConsentEvent`, `PrivacyRequest`, and `Suppression`

**Purpose:** Enforce candidate rights and prevent repeat contact.

**Core fields**

`PrivacyRequest`

- `privacy_request_id`
- `candidate_id`
- `request_type`: `access | correction | deletion | restriction | opt_out`
- `status`
- `received_at`
- `completed_at`
- `blocking_reason`

`Suppression`

- `suppression_id`
- `person_id`
- `channels[]`
- `scope`
- `reason`
- `created_at`
- `expires_at`

`ConsentEvent`

- `consent_event_id`
- `consent_id`
- `event_type`
- `actor`
- `captured_at`
- `evidence_id`

**AI-native decision:** Suppression is checked synchronously immediately before every send and share.

## 1.25 `TrustEvent` and `AuditEvent`

**Purpose:** Monitor reputation risk and prove what happened.

**Core fields**

`TrustEvent`

- `trust_event_id`
- `event_type`: `spam_report | privacy_complaint | identity_concern | consent_mismatch | abuse_report | bounce`
- `severity`
- `related_entity`
- `description`
- `status`
- `created_at`
- `resolved_at`

`AuditEvent`

- `audit_event_id`
- `actor_type`
- `actor_id`
- `action`
- `entity_type`
- `entity_id`
- `before`
- `after`
- `reason`
- `occurred_at`

**AI-native decision:** Trust metrics are operating metrics. The system should report complaint rate, opt-out rate, consent completeness, and repeat-contact violations next to placements and revenue.

---

# 2. List of Business Processes

Every process has four roles:

- **AI agent:** performs research, drafting, classification, routing, monitoring, or follow-up.
- **Solopreneur:** owns judgment, relationships, exceptions, reputation, and legally/commercially consequential decisions.
- **System:** enforces constraints and records events.
- **External party:** candidate or employer provides the necessary decision or signal.

## 2.1 Define market focus and operating policy

**Purpose:** Decide which jobs, candidates, geographies, and companies the platform will serve.

**AI agent**

- Researches market segments and role demand.
- Summarizes company and role patterns.
- Identifies attractive candidate pools.
- Recommends a narrow initial niche.

**Solopreneur lead**

- Chooses the market wedge.
- Sets prohibited categories and geographies.
- Approves the fee model and service promise.

**System outputs**

- Market thesis
- Role ontology
- Candidate segments
- Geographic and jurisdiction policy
- Outreach and representation rules

**Design implication:** A solo operator should start with a narrow, high-value niche such as startup engineering/product talent rather than attempt a general labor marketplace.

## 2.2 Discover candidate leads

**Purpose:** Identify people who may fit the chosen market.

**AI agent**

- Searches approved sources.
- Extracts structured signals from resumes, public profiles, portfolios, referrals, and prior conversations.
- Deduplicates identities.
- Scores source quality and potential fit.
- Creates `Candidate` records in `discovered` status.

**Solopreneur lead**

- Approves source policies.
- Reviews unusual, sensitive, or low-confidence candidates.
- Handles referrals and high-value personal outreach.

**System controls**

- Store `Evidence` and source provenance.
- Do not treat public visibility as consent.
- Run suppression and jurisdiction checks before outreach.

**Output:** Candidate lead queue.

## 2.3 Qualify and prioritize candidate leads

**Purpose:** Decide which discovered people deserve attention.

**AI agent**

- Enriches candidate records.
- Estimates fit for the target market.
- Identifies missing information.
- Ranks leads by likely placement value, responsiveness, and confidence.

**Solopreneur lead**

- Approves the priority queue.
- Overrides ranking for strategic or relationship reasons.
- Rejects candidates who should not be contacted.

**System controls**

- Do not use protected characteristics or inferred sensitive attributes.
- Record why a candidate was prioritized.

**Output:** Prioritized candidate task queue.

## 2.4 Contact and onboard a candidate

**Purpose:** Convert a lead into an informed relationship.

**AI agent**

- Selects an approved campaign and channel.
- Personalizes an outreach draft from verified evidence.
- Discloses the platform’s identity and AI involvement.
- Sends or queues the message within frequency limits.
- Handles routine replies.
- Captures opt-outs and routes questions.

**Solopreneur lead**

- Approves campaign policy and high-risk templates.
- Handles skepticism, complaints, high-value candidates, and unusual requests.
- Approves any non-standard outreach.

**System controls**

- Verify `AgentIdentity`.
- Check `ConsentRecord` and `Suppression` immediately before send.
- Create `Outreach` and `AuditEvent`.

**Output:** Engaged candidate, consent request, or suppression record.

## 2.5 Obtain consent and representation

**Purpose:** Establish what the platform may do on the candidate’s behalf.

**AI agent**

- Explains the service in plain language.
- Answers routine questions from approved knowledge.
- Presents purpose-specific permissions.
- Captures consent evidence.
- Identifies ambiguity or disagreement.

**Solopreneur lead**

- Resolves nuanced consent questions.
- Approves terms and exceptions.
- Handles any candidate who requests unusual representation scope.

**System controls**

- Separate consent for contact, profile creation, representation, and employer sharing.
- Store notice version, timestamp, channel, and evidence.
- Activate `RepresentationAgreement` only after the required permission exists.

**Output:** Active or declined representation relationship.

## 2.6 Build and maintain the candidate profile

**Purpose:** Produce an accurate, compelling, and current representation.

**AI agent**

- Parses source material.
- Drafts profile summaries and achievement narratives.
- Extracts skills and evidence.
- Detects stale or contradictory information.
- Proposes profile updates.

**Solopreneur lead**

- Reviews high-value profiles.
- Resolves contradictions and sensitive claims.
- Approves the final narrative for strategic candidates.

**External candidate**

- Corrects, approves, or rejects material claims.
- Maintains preferences and availability.

**System controls**

- Version every profile.
- Preserve source evidence.
- Do not share unapproved material claims.

**Output:** Candidate-approved `CandidateProfile`.

## 2.7 Acquire and qualify employer accounts

**Purpose:** Build a supply of credible companies and hiring relationships.

**AI agent**

- Researches target startups and hiring signals.
- Finds relevant founders and hiring managers.
- Drafts account briefs and outreach.
- Monitors company changes and new roles.
- Scores account fit and payment likelihood.

**Solopreneur lead**

- Approves target accounts.
- Establishes the relationship.
- Qualifies culture, hiring quality, decision authority, and willingness to pay.

**System controls**

- Verify company and contact identity.
- Record employer permissions.
- Flag reputation, fraud, or payment-risk signals.

**Output:** Qualified `Company` and `EmployerUser`.

## 2.8 Intake and structure a job

**Purpose:** Convert employer demand into a matchable role.

**AI agent**

- Reads the job description.
- Drafts the structured requirements.
- Identifies missing constraints.
- Proposes compensation, location, seniority, and interview-stage fields.
- Drafts candidate-facing role language.

**Solopreneur lead**

- Runs the intake conversation.
- Clarifies must-haves versus preferences.
- Negotiates fee terms.
- Approves role activation.

**System controls**

- Require human verification for hard requirements.
- Capture role visibility and confidentiality.
- Store `FeeAgreement` before introductions.

**Output:** Active `Role` and `RoleRequirement` set.

## 2.9 Match candidates to roles

**Purpose:** Identify credible opportunities without turning the product into opaque ranking.

**AI agent**

- Applies hard constraints.
- Calculates fit across skills, seniority, motivation, location, compensation, and company preferences.
- Produces reasons, evidence, and risks.
- Creates `Match` records.
- Ranks the review queue.

**Solopreneur lead**

- Reviews exceptions and top opportunities.
- Adds relationship and context judgment.
- Approves matches that could materially affect reputation.

**System controls**

- Enforce representation scope and consent.
- Store model version and evidence.
- Keep candidate and employer interests separate.

**Output:** Explainable match and candidate opportunity queue.

## 2.10 Present an opportunity to the candidate

**Purpose:** Ask whether the candidate wants to pursue a particular role.

**AI agent**

- Generates a concise, role-specific opportunity brief.
- Explains why the match exists.
- Answers routine questions from approved role data.
- Captures interest, decline, or “need more information.”
- Schedules follow-up within a bounded cadence.

**Solopreneur lead**

- Handles sensitive compensation, career, or reputation questions.
- Decides when to advocate personally.
- Stops a campaign if the candidate experience deteriorates.

**System controls**

- Do not imply an employer has approved a candidate before employer interest exists.
- Store `InterestSignal`.
- Respect preferences and opt-outs.

**Output:** Candidate interest signal.

## 2.11 Present the candidate to the employer

**Purpose:** Share a controlled candidate packet only after appropriate candidate permission.

**AI agent**

- Creates a role-specific `CandidatePacket`.
- Redacts fields outside the approved scope.
- Drafts the employer presentation.
- Answers routine employer questions from evidence.
- Tracks employer response.

**Solopreneur lead**

- Approves the packet for strategic or sensitive candidates.
- Handles objections, negotiation, and narrative positioning.
- Decides whether to withdraw a candidate.

**System controls**

- Check representation, consent, candidate interest, and role status.
- Log exactly what was shared and when.
- Never expose the full internal profile by default.

**Output:** Employer interest signal or decline.

## 2.12 Create the mutual introduction

**Purpose:** Turn two independent positive signals into an accountable introduction.

**AI agent**

- Verifies both interest signals.
- Drafts the introduction.
- Selects the approved channel.
- Shares scheduling links or contact details.
- Tracks acceptance and response.

**Solopreneur lead**

- Approves non-standard introductions.
- Makes high-value personal introductions.
- Resolves disagreement over candidate ownership or fee attribution.

**System controls**

- Require candidate and employer opt-in.
- Identify the platform’s role.
- Attach the role and fee agreement.
- Create an immutable `Introduction` record.

**Output:** Introduction sent and attributable.

## 2.13 Coordinate interviews and collect feedback

**Purpose:** Keep the process moving without requiring the founder to chase every participant.

**AI agent**

- Schedules interviews.
- Sends reminders.
- Collects candidate and employer feedback.
- Detects stalled processes.
- Escalates negative sentiment or contradictory updates.

**Solopreneur lead**

- Intervenes in sensitive negotiations.
- Coaches the candidate or employer when appropriate.
- Decides whether to repair, pause, or end the relationship.

**System controls**

- Store stage history and feedback separately.
- Avoid presenting AI-generated feedback as a participant’s own words.
- Record no-shows, withdrawals, and rejection reasons where voluntarily provided.

**Output:** Current `Interview` state and feedback signals.

## 2.14 Confirm hiring and attribute placement

**Purpose:** Determine whether the platform created a payable placement.

**AI agent**

- Detects likely hiring signals.
- Requests confirmation from the candidate and employer.
- Compares dates, role, and compensation.
- Creates a provisional placement recommendation.

**Solopreneur lead**

- Confirms disputed or ambiguous placements.
- Negotiates attribution and replacement terms.
- Approves the revenue event.

**System controls**

- Require evidence-backed `HiringOutcome`.
- Link placement to introduction, role, candidate, and fee agreement.
- Mark attribution as provisional until the guarantee period passes.

**Output:** Confirmed or disputed `Placement`.

## 2.15 Invoice and collect success fees

**Purpose:** Convert confirmed placement into cash.

**AI agent**

- Calculates the fee from approved terms.
- Drafts and sends invoice.
- Monitors due dates.
- Sends payment reminders.
- Flags disputes and overdue accounts.

**Solopreneur lead**

- Approves unusual calculations, discounts, replacements, and disputes.
- Manages strategic employer relationships.

**System controls**

- Invoice only against a valid placement and fee agreement.
- Preserve terms version and calculation basis.
- Track payment and replacement windows.

**Output:** Paid, overdue, disputed, or void `Invoice`.

## 2.16 Manage privacy requests, complaints, and suppression

**Purpose:** Protect candidates and maintain the credibility of the marketplace.

**AI agent**

- Classifies inbound complaints and requests.
- Resolves identity across records.
- Freezes outreach and sharing.
- Produces a deletion or access checklist.
- Monitors for repeat contact.
- Summarizes campaign-level patterns.

**Solopreneur lead**

- Owns the response to serious complaints.
- Makes legal or reputational escalation decisions.
- Approves restoration after a dispute.

**System controls**

- Create `PrivacyRequest`, `TrustEvent`, `Suppression`, and `AuditEvent`.
- Block all sends and shares while a deletion or opt-out is active.
- Preserve only minimum necessary compliance evidence.

**Output:** Completed request, resolved complaint, or escalated risk.

## 2.17 Monitor business health and agent performance

**Purpose:** Give one founder a complete operating picture.

**AI agent**

- Produces a daily operating brief.
- Monitors funnel conversion and revenue.
- Detects agent errors, stalled tasks, complaint clusters, and unusual outreach.
- Recommends which exceptions deserve founder attention.

**Solopreneur lead**

- Reviews the exception queue.
- Adjusts market focus, policies, templates, and agent permissions.
- Makes the final decision on risk appetite.

**System outputs**

- Candidate funnel: discovery → consent → representation → interest
- Employer funnel: account → role → interest → fee agreement
- Placement funnel: match → introduction → interview → hire → paid
- Trust funnel: contact → opt-out → complaint → deletion → repeat-contact violation
- Agent metrics: completion rate, escalation rate, error rate, override rate

**Core operating metrics**

- Candidate consent rate
- Candidate opt-out and complaint rate per 1,000 contacts
- Profile approval rate
- Candidate interest rate
- Employer interest rate
- Introduction acceptance rate
- Interview rate
- Placement rate
- Revenue per active candidate
- Time from role activation to first qualified introduction
- Percentage of shared profile fields with provenance
- Repeat-contact violations

---

## Human-in-the-lead operating rules

1. **Agents prepare; the founder owns consequential judgment.**
2. **No external contact without a policy, identity, purpose, and suppression check.**
3. **No employer sharing without candidate scope and permission.**
4. **No introduction without two independent interest signals.**
5. **No invoice without evidence-backed attribution.**
6. **No AI inference becomes a candidate fact without source, confidence, and review.**
7. **Any complaint pauses the relevant workflow before growth continues.**
8. **The founder’s dashboard is an exception queue, not an activity feed.**

The resulting business is a small human-led agency whose operating leverage comes from agents, but whose defensibility comes from **trusted representation, proprietary relationship data, high-quality evidence, and reliable placement attribution**.
