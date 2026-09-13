# AI-Native Talent Marketplace — Inferred Product Model

## 0. Scope and confidence

This is an external product inference based on market research, not an internal company specification.

**Market-analysis boundary:** Cleara is treated only as a competitor and market reference. The product model below is brand-neutral and describes the category opportunity rather than Cleara’s internal system.

- **Observed / stated:** candidate-side representation, employer success fee, candidate and startup counts, web/email/iMessage/WhatsApp onboarding, matching to startups, mutual opt-in introductions, employer-side Slack workflows, and reported placements.
- **Strongly inferred:** normalized records for candidates, companies, roles, consent, outreach, matches, introductions, interviews, and placements.
- **Unknown:** the exact ranking model, internal recruiter operations, identity resolution logic, billing system, retention rules, and whether every outreach recipient has an explicit account.

The inferred product is best understood as a **two-sided talent marketplace with an AI-assisted candidate agent and a success-fee recruiting back office**.

---

## 1. Product thesis

The product appears to operate three connected systems:

1. **Candidate agent**
   - Finds or receives candidate signals.
   - Builds a structured representation of the candidate.
   - Communicates opportunities through web, email, iMessage, or WhatsApp.
   - Requests preferences and permission to represent the candidate.

2. **Employer marketplace**
   - Ingests startup companies and open roles.
   - Converts role requirements into a structured hiring brief.
   - Matches candidates to roles.
   - Lets employers review candidate packets and signal interest.

3. **Placement and revenue operations**
   - Manages mutual opt-in introductions.
   - Tracks interviews and hiring outcomes.
   - Creates a fee obligation when a placement succeeds.
   - Handles replacement, disputes, attribution, and payment collection.

The product’s key data invariant should be:

> A candidate may be discovered without consent, but must not be represented, contacted as a platform user, shared with an employer, or enrolled into a marketplace workflow without a recorded legal and product-level basis.

That invariant is inferred as a necessary product requirement because the report’s central risk concerns unclear sourcing, consent, auto-enrollment, and simulated recruiter identities.

---

## 2. Bounded contexts

| Context | Owns | Primary business question |
|---|---|---|
| Identity & organizations | People, companies, users, permissions | Who is acting, and for which organization? |
| Candidate representation | Candidate profile, preferences, consent, agent relationship | Can the platform represent this person, and under what terms? |
| Employer hiring | Company, role, hiring team, role requirements | What does the employer need? |
| Marketplace matching | Match scores, candidate packets, employer interest | Is there a credible fit? |
| Communication | Outreach, conversations, messages, agent identity | What was sent, by whom, through which channel, and why? |
| Introduction & hiring | Opt-ins, introductions, interviews, outcomes | Did both sides agree, and did the process progress? |
| Revenue & trust | Placements, fees, invoices, disputes, deletion requests | Was value created, and can every action be explained? |

---

## 3. Canonical data model

The following is a relational model expressed in TypeScript-like pseudocode. IDs are opaque UUIDs. All timestamps are UTC. Sensitive fields should be encrypted or tokenized.

### 3.1 Identity and organizations

```ts
type UserId = string;
type OrganizationId = string;
type CandidateId = string;
type CompanyId = string;
type RoleId = string;

type User = {
  id: UserId;
  email: string;
  name?: string;
  phone?: string;
  accountType: "candidate" | "employer_member" | "platform_operator" | "admin";
  status: "invited" | "active" | "suspended" | "deleted";
  createdAt: Date;
  lastSeenAt?: Date;
};

type Organization = {
  id: OrganizationId;
  type: "platform_operator" | "employer" | "partner";
  legalName: string;
  displayName: string;
  website?: string;
  fundingStage?: string;
  source: "self_reported" | "imported" | "operator_created" | "partner";
  verificationStatus: "unverified" | "partially_verified" | "verified";
  createdAt: Date;
};

type OrganizationMember = {
  organizationId: OrganizationId;
  userId: UserId;
  role: "owner" | "hiring_manager" | "recruiter" | "viewer" | "operator";
  status: "active" | "revoked";
};

type Company = {
  id: CompanyId;
  organizationId: OrganizationId;
  name: string;
  domain?: string;
  description?: string;
  location?: string;
  stage?: string;
  sector?: string;
  investorTags?: string[];
  verificationStatus: "unverified" | "verified" | "restricted";
  candidateVisibility: "public" | "confidential" | "anonymized";
  createdAt: Date;
};
```

### 3.2 Candidate representation

Separate the person from the representation. A person can exist as a discovered lead before becoming an active platform candidate.

```ts
type Candidate = {
  id: CandidateId;
  userId?: UserId;
  canonicalEmailHash?: string;
  canonicalPhoneHash?: string;
  name?: string;
  location?: string;
  linkedinUrl?: string;
  source: CandidateSource;
  lifecycle:
    | "discovered"
    | "contacted"
    | "engaged"
    | "consented"
    | "represented"
    | "paused"
    | "deleted"
    | "suppressed";
  dataQuality: "unresolved" | "partial" | "complete" | "needs_review";
  createdAt: Date;
  updatedAt: Date;
};

type CandidateSource =
  | { type: "self_signup"; campaignId?: string }
  | { type: "referral"; referrerCandidateId?: CandidateId }
  | { type: "employer_referral"; companyId: CompanyId }
  | { type: "public_profile_import"; sourceUrl: string }
  | { type: "operator_import"; batchId: string }
  | { type: "unknown"; detail?: string };

type CandidateProfile = {
  id: string;
  candidateId: CandidateId;
  version: number;
  headline?: string;
  summary?: string;
  skills: SkillEvidence[];
  workHistory: WorkExperience[];
  education: Education[];
  locationPreferences?: LocationPreference[];
  compensationExpectation?: CompensationRange;
  rolePreferences?: RolePreference[];
  workAuthorization?: WorkAuthorization;
  availability?: "immediate" | "notice_period" | "not_looking" | "unknown";
  resumeAssetId?: string;
  generatedBy: "candidate" | "operator" | "ai" | "import";
  approvedByCandidateAt?: Date;
  effectiveFrom: Date;
  effectiveTo?: Date;
};

type SkillEvidence = {
  skill: string;
  proficiency?: "familiar" | "working" | "advanced" | "expert";
  evidence: string[];
  source: "resume" | "linkedin" | "candidate_statement" | "interview" | "reference";
  confidence: number; // 0.0 - 1.0
};

type WorkExperience = {
  employer: string;
  title: string;
  startedOn?: string;
  endedOn?: string;
  description?: string;
  verified: boolean;
};

type CandidateRepresentation = {
  id: string;
  candidateId: CandidateId;
  status: "pending" | "active" | "paused" | "revoked" | "expired";
  scope: "marketplace_matching" | "specific_role" | "specific_company";
  sourceConsentId: string;
  termsVersion: string;
  startsAt: Date;
  endsAt?: Date;
  revokedAt?: Date;
  revocationReason?: string;
};
```

### 3.3 Consent, provenance, and privacy

This is a first-class subsystem, not a boolean on `Candidate`.

```ts
type ConsentRecord = {
  id: string;
  subjectType: "candidate";
  subjectId: CandidateId;
  purpose:
    | "contact"
    | "create_profile"
    | "represent_candidate"
    | "share_profile_with_employer"
    | "send_specific_opportunity"
    | "research_public_profile";
  legalBasis: "consent" | "contract" | "legitimate_interest" | "unknown";
  status: "granted" | "denied" | "withdrawn" | "expired" | "needs_review";
  scope: {
    channels?: ("email" | "sms" | "imessage" | "whatsapp" | "web")[];
    employerIds?: CompanyId[];
    roleIds?: RoleId[];
  };
  noticeVersion?: string;
  capturedAt?: Date;
  capturedFrom?: string; // URL, message, form, or operator workflow
  evidenceAssetId?: string;
  expiresAt?: Date;
  withdrawnAt?: Date;
};

type DataProvenance = {
  id: string;
  entityType: "candidate" | "profile" | "attribute" | "resume" | "message";
  entityId: string;
  fieldPath?: string;
  sourceType: "candidate" | "public_profile" | "employer" | "operator" | "ai_inference";
  sourceLocator?: string;
  collectedAt: Date;
  processingPurpose: string;
  confidence?: number;
  retentionUntil?: Date;
};

type PrivacyRequest = {
  id: string;
  candidateId: CandidateId;
  type: "access" | "correction" | "deletion" | "restriction" | "opt_out";
  status: "received" | "in_review" | "executing" | "completed" | "blocked";
  requestedAt: Date;
  completedAt?: Date;
  blockingReason?: string;
  auditLogId: string;
};
```

### 3.4 Employer hiring

```ts
type Role = {
  id: RoleId;
  companyId: CompanyId;
  title: string;
  department?: string;
  description: string;
  status: "draft" | "active" | "paused" | "filled" | "closed";
  visibility: "named" | "confidential" | "anonymized";
  employmentType?: "full_time" | "part_time" | "contract";
  locationPolicy?: "onsite" | "hybrid" | "remote";
  locations?: string[];
  compensation?: CompensationRange;
  targetStartDate?: string;
  successFeeTermsId?: string;
  source: "employer_submitted" | "operator_created" | "imported";
  publishedAt?: Date;
  closedAt?: Date;
};

type RoleRequirement = {
  id: string;
  roleId: RoleId;
  category: "must_have" | "nice_to_have" | "constraint" | "signal";
  field: string;
  operator: "equals" | "contains" | "minimum" | "maximum" | "one_of";
  value: string | number | string[];
  weight: number;
  humanVerified: boolean;
};

type HiringProcess = {
  id: string;
  roleId: RoleId;
  stage:
    | "new"
    | "intake"
    | "sourcing"
    | "reviewing"
    | "interviewing"
    | "offer"
    | "filled"
    | "closed";
  ownerUserId: UserId;
  sla?: { firstResponseHours?: number; feedbackHours?: number };
  createdAt: Date;
  updatedAt: Date;
};

type CompensationRange = {
  currency: string;
  min?: number;
  max?: number;
  period: "hour" | "month" | "year";
  isPublic: boolean;
};
```

### 3.5 Matching and candidate presentation

Matching should preserve both the machine output and the explanation shown to a human.

```ts
type Match = {
  id: string;
  candidateId: CandidateId;
  roleId: RoleId;
  status:
    | "scored"
    | "reviewed"
    | "candidate_pending"
    | "candidate_opted_in"
    | "employer_pending"
    | "employer_interested"
    | "declined"
    | "expired"
    | "converted";
  score?: number;
  scoreVersion?: string;
  modelRunId?: string;
  hardConstraintResult: "pass" | "fail" | "unknown";
  reasons: MatchReason[];
  risks: MatchRisk[];
  createdAt: Date;
  expiresAt?: Date;
};

type MatchReason = {
  dimension: "skills" | "seniority" | "industry" | "location" | "compensation" | "motivation";
  explanation: string;
  supportingEvidenceIds: string[];
  weight: number;
};

type MatchRisk = {
  dimension: string;
  explanation: string;
  severity: "low" | "medium" | "high";
};

type CandidatePacket = {
  id: string;
  matchId: string;
  version: number;
  visibility: "internal" | "candidate_visible" | "employer_visible";
  redactionRules: string[];
  fields: Record<string, unknown>;
  generatedAt: Date;
  approvedByCandidateAt?: Date;
  sharedAt?: Date;
};
```

### 3.6 Outreach and agent identity

The sender identity must be modeled explicitly so every message is attributable.

```ts
type AgentIdentity = {
  id: string;
  organizationId: OrganizationId;
  displayName: string;
  identityType: "human_operator" | "ai_disclosed" | "ai_assisted_human";
  disclosureText?: string;
  channelAddresses: {
    channel: "email" | "sms" | "imessage" | "whatsapp" | "web";
    address: string;
    verified: boolean;
  }[];
  active: boolean;
};

type OutreachCampaign = {
  id: string;
  purpose: "candidate_discovery" | "candidate_onboarding" | "opportunity_alert" | "employer_sourcing";
  audienceDefinition: Record<string, unknown>;
  consentPolicy: "opt_in_only" | "legitimate_interest_review" | "manual_approval";
  agentIdentityId: string;
  templateVersion: string;
  status: "draft" | "approved" | "running" | "paused" | "completed";
  createdAt: Date;
};

type Outreach = {
  id: string;
  campaignId?: string;
  candidateId?: CandidateId;
  roleId?: RoleId;
  agentIdentityId: string;
  channel: "email" | "sms" | "imessage" | "whatsapp" | "web";
  destination: string;
  purpose: string;
  consentRecordId?: string;
  status: "queued" | "sent" | "delivered" | "replied" | "bounced" | "opted_out" | "blocked";
  sentAt?: Date;
  repliedAt?: Date;
  optOutAt?: Date;
  messageId?: string;
};

type Conversation = {
  id: string;
  candidateId?: CandidateId;
  companyId?: CompanyId;
  roleId?: RoleId;
  channel: "email" | "sms" | "imessage" | "whatsapp" | "web" | "slack";
  participantIds: string[];
  state: "open" | "waiting" | "snoozed" | "closed" | "suppressed";
  assignedTo?: UserId;
  lastMessageAt?: Date;
};

type Message = {
  id: string;
  conversationId: string;
  senderType: "candidate" | "employer" | "platform_operator" | "ai_agent" | "system";
  senderId?: string;
  agentIdentityId?: string;
  aiGenerated: boolean;
  disclosureShown: boolean;
  body: string;
  templateVersion?: string;
  sentAt: Date;
  deliveryStatus: "queued" | "sent" | "delivered" | "failed";
};
```

### 3.7 Mutual opt-in, introductions, and hiring

```ts
type InterestSignal = {
  id: string;
  matchId: string;
  actorType: "candidate" | "employer";
  actorId: CandidateId | CompanyId;
  signal: "interested" | "not_interested" | "needs_more_info" | "snooze";
  reason?: string;
  capturedAt: Date;
};

type Introduction = {
  id: string;
  matchId: string;
  candidateId: CandidateId;
  companyId: CompanyId;
  roleId: RoleId;
  candidateInterestSignalId: string;
  employerInterestSignalId: string;
  status: "pending" | "approved" | "sent" | "accepted" | "declined" | "completed";
  method: "email" | "shared_slack" | "portal" | "direct_contact";
  introMessage?: string;
  sentAt?: Date;
  acceptedAt?: Date;
};

type Interview = {
  id: string;
  introductionId: string;
  stage: string;
  scheduledAt?: Date;
  status: "requested" | "scheduled" | "completed" | "cancelled" | "no_show";
  candidateFeedback?: string;
  employerFeedback?: string;
  feedbackCapturedAt?: Date;
};

type HiringOutcome = {
  id: string;
  introductionId: string;
  status: "unknown" | "rejected" | "withdrew" | "offer" | "hired" | "contracted";
  startDate?: string;
  compensation?: CompensationRange;
  reportedBy: "candidate" | "employer" | "operator" | "integration";
  reportedAt: Date;
  evidenceAssetId?: string;
};

type Placement = {
  id: string;
  hiringOutcomeId: string;
  candidateId: CandidateId;
  companyId: CompanyId;
  roleId: RoleId;
  attributionStatus: "provisional" | "confirmed" | "disputed";
  guaranteeEndsAt?: Date;
  feeAgreementId: string;
  createdAt: Date;
};

type FeeAgreement = {
  id: string;
  companyId: CompanyId;
  feeType: "success_fee" | "subscription" | "other";
  percentageOfSalary?: number;
  flatAmount?: number;
  currency?: string;
  replacementWindowDays?: number;
  termsVersion: string;
  acceptedAt?: Date;
};

type Invoice = {
  id: string;
  placementId: string;
  companyId: CompanyId;
  amount: number;
  currency: string;
  status: "draft" | "issued" | "paid" | "overdue" | "disputed" | "void";
  issuedAt?: Date;
  dueAt?: Date;
  paidAt?: Date;
};
```

### 3.8 Trust, safety, and audit

```ts
type TrustEvent = {
  id: string;
  actorType: "candidate" | "employer" | "operator" | "system";
  actorId?: string;
  eventType:
    | "spam_report"
    | "privacy_complaint"
    | "identity_concern"
    | "data_deletion_request"
    | "review"
    | "abuse_report"
    | "consent_mismatch"
    | "delivery_bounce";
  severity: "low" | "medium" | "high" | "critical";
  relatedEntityType?: string;
  relatedEntityId?: string;
  description: string;
  status: "open" | "triaged" | "resolved" | "dismissed";
  createdAt: Date;
  resolvedAt?: Date;
};

type AuditEvent = {
  id: string;
  actorType: "user" | "operator" | "ai" | "system";
  actorId?: string;
  action: string;
  entityType: string;
  entityId: string;
  before?: Record<string, unknown>;
  after?: Record<string, unknown>;
  reason?: string;
  occurredAt: Date;
};
```

---

## 4. Relationship map

```text
Organization
 ├── OrganizationMember ── User
 └── Company
      └── Role
           ├── RoleRequirement
           ├── Match ───────────── Candidate
           │    ├── MatchReason
           │    ├── CandidatePacket
           │    └── InterestSignal × 2
           └── Introduction
                ├── Interview
                ├── HiringOutcome
                └── Placement ── FeeAgreement ── Invoice

Candidate
 ├── CandidateProfile (versioned)
 ├── CandidateRepresentation
 ├── ConsentRecord
 ├── DataProvenance
 ├── Outreach
 ├── Conversation ── Message
 ├── PrivacyRequest
 └── TrustEvent

AgentIdentity
 ├── OutreachCampaign
 ├── Outreach
 └── Message
```

---

## 5. State machines

### 5.1 Candidate lifecycle

```text
discovered
  ├─ no legitimate basis / suppression ──> suppressed
  ├─ outreach permitted ────────────────> contacted
  └─ candidate self-signup ─────────────> engaged

contacted ── replies ──> engaged
engaged ── grants required consent ──> consented
consented ── profile approved / representation accepted ──> represented
represented ── pause ──> paused
represented ── revoke consent ──> revoked
any state ── deletion completed ──> deleted
```

### 5.2 Match lifecycle

```text
scored
  ├─ hard constraint failure ──> declined
  └─ human/operator review ────> reviewed

reviewed ── candidate outreach ──> candidate_pending
candidate_pending ── candidate interested ──> candidate_opted_in
candidate_pending ── candidate declines ──> declined
candidate_opted_in ── employer packet shared ──> employer_pending
employer_pending ── employer interested ──> employer_interested
employer_interested + candidate_opted_in ──> converted / Introduction
```

### 5.3 Introduction-to-revenue lifecycle

```text
Introduction: approved
  └─ contact details / intro message delivered ──> sent
        ├─ employer accepts ──> accepted
        ├─ employer declines ──> declined
        └─ both sides complete process ──> completed

accepted ── interviews ──> HiringOutcome
HiringOutcome: offer ──> Placement: provisional
Placement: provisional ── guarantee period passes ──> confirmed
Placement: confirmed ──> Invoice: issued ──> paid
Any placement ── dispute ──> disputed
```

---

## 6. Business processes

## Process A — Candidate discovery and identity resolution

**Goal:** create a candidate lead without confusing public discovery with consent.

1. Receive a candidate signal from self-signup, referral, employer request, public profile import, or operator research.
2. Normalize email, phone, name, and public profile URL.
3. Resolve against existing candidate records using deterministic identifiers first.
4. Create or update `Candidate`.
5. Write `DataProvenance` for every imported attribute.
6. Run suppression checks:
   - prior opt-out or deletion request;
   - duplicate or conflicting identity;
   - bounced address;
   - blocked geography or legal restriction;
   - source not permitted for the intended purpose.
7. If the system cannot establish a valid outreach basis, hold the record in `discovered` and do not send a message.

**Outputs:** candidate lead, provenance records, suppression decision, audit event.

**Important control:** “Publicly visible” is a source classification, not proof of permission to create a full candidate profile or contact the person.

---

## Process B — Candidate outreach and onboarding

**Goal:** convert a lead into an informed, active candidate relationship.

1. Select an approved `OutreachCampaign` and `AgentIdentity`.
2. Check the candidate’s `ConsentRecord` and communication preferences.
3. Render a message that states:
   - the platform’s identity;
   - whether the sender is human, AI, or AI-assisted;
   - why the candidate is being contacted;
   - what data is already held;
   - what the candidate is being asked to do;
   - how to opt out or request deletion.
4. Create `Outreach` before sending.
5. Send through the selected channel.
6. Record delivery, reply, bounce, or opt-out.
7. On reply, create or attach a `Conversation`.
8. If the candidate proceeds, capture granular consent for profile creation, representation, and opportunity sharing.

**Outputs:** outreach record, conversation, consent event, candidate state transition.

**Failure handling:**

- Any opt-out immediately creates a suppression entry and blocks future campaigns.
- A bounce blocks retry until the address is re-verified.
- A complaint creates a high-priority `TrustEvent`.
- If an agent identity or domain changes, pause the campaign until re-approved.

---

## Process C — Profile construction and candidate representation

**Goal:** create a useful, explainable candidate profile.

1. Collect candidate-provided information, resume, public profile data, and optional conversation signals.
2. Parse structured fields and store source-level `DataProvenance`.
3. Generate a draft `CandidateProfile`.
4. Attach confidence and evidence to each material claim.
5. Ask the candidate to correct or approve the profile.
6. Create or activate `CandidateRepresentation` only for the approved scope.
7. Version the profile rather than overwriting it.
8. Make the candidate’s visibility and sharing preferences queryable.

**Outputs:** versioned profile, evidence links, representation agreement, audit trail.

**Product principle:** AI may draft a profile; it should not silently convert an inferred attribute into an asserted fact.

---

## Process D — Employer and role intake

**Goal:** translate a startup’s hiring need into a matchable brief.

1. Create or verify `Company`.
2. Capture the company’s hiring team and permissions.
3. Capture `FeeAgreement` before candidate introductions.
4. Create `Role`.
5. Parse the job description into `RoleRequirement`.
6. Have an employer or platform operator verify hard constraints:
   - role level;
   - location / work authorization;
   - compensation;
   - must-have skills;
   - start date;
   - interview process.
7. Select role visibility: named, confidential, or anonymized.
8. Publish the role to the matching system.

**Outputs:** verified company, active role, structured requirements, fee terms.

**Control:** an anonymized role needs an internal reason and a controlled disclosure path; anonymization should not become a way to conceal that the platform is collecting candidate data.

---

## Process E — Candidate-role matching

**Goal:** rank plausible matches while keeping the reasoning inspectable.

1. Retrieve active roles and represented candidates.
2. Apply hard constraints first.
3. Calculate a match score using skills, seniority, industry, location, compensation, and candidate motivation.
4. Store model version, score, reasons, evidence IDs, and risks in `Match`.
5. Apply policy filters:
   - candidate representation scope;
   - role visibility;
   - consent to share;
   - employer restrictions;
   - duplicate submission rules;
   - candidate suppression.
6. Route high-value or low-confidence matches to operator review.
7. Create a `CandidatePacket` containing only the fields permitted for that role and stage.

**Outputs:** explainable match, candidate packet, review queue.

**Success metric:** not simply “number of matches.” Better measures are candidate opt-in rate, employer interest rate, interview rate, placement rate, and complaint rate per 1,000 contacts.

---

## Process F — Mutual opt-in and introduction

**Goal:** ensure a warm introduction occurs only after both sides express interest.

1. Send a role-specific opportunity to the candidate.
2. Capture candidate interest, decline, or request for more information.
3. If interested, share the permitted candidate packet with the employer.
4. Capture employer interest or decline.
5. When both sides are positive, create `Introduction`.
6. Select introduction method: email, shared Slack channel, portal, or direct contact.
7. Send a clear introduction that identifies the platform’s role and any relevant fee terms.
8. Track acceptance, scheduling, and subsequent communication.

**Outputs:** two-sided interest signals, introduction record, communication history.

**Control:** a candidate packet should not be shared with an employer merely because a matching score is high. The share event needs an explicit permission check.

---

## Process G — Interview, placement, and fee collection

**Goal:** convert a successful introduction into attributable revenue.

1. Create interview records as the process advances.
2. Capture candidate and employer feedback after each meaningful stage.
3. Record a hiring outcome from the employer, candidate, operator, or integration.
4. If hired, create a provisional `Placement`.
5. Attach the applicable `FeeAgreement`.
6. Calculate the success fee from the agreed compensation basis.
7. Issue an `Invoice`.
8. Track payment, replacement window, dispute, and collection status.
9. Convert the placement to confirmed when the guarantee period passes.
10. Feed outcome data back into matching quality and employer account health.

**Outputs:** placement attribution, invoice, revenue event, quality feedback.

**Revenue invariant:** every invoice should trace back to a role, introduction, candidate opt-in, employer opt-in, and fee agreement.

---

## Process H — Privacy request, complaint, and trust recovery

**Goal:** make trust issues operationally resolvable.

1. Receive an opt-out, deletion request, privacy complaint, identity complaint, or spam report.
2. Resolve the candidate identity across aliases and duplicate records.
3. Freeze active campaigns and employer sharing for the candidate.
4. Create `PrivacyRequest` and `TrustEvent`.
5. Preserve only the minimum audit evidence needed to prove compliance.
6. Remove, redact, or restrict personal data according to the request and applicable policy.
7. Notify downstream employers or systems where the candidate’s data was shared.
8. Record completion and reopen only after a new valid basis exists.
9. Analyze the event by campaign, agent identity, sender domain, source, template, and operator.

**Outputs:** suppression/deletion action, complaint resolution, campaign-level learning.

**Key metrics:** complaint rate by sender identity, opt-out rate, deletion completion time, repeat-contact rate after opt-out, and percentage of candidate records with complete provenance.

---

## 7. Event model

The relational records above should emit an append-only event stream for analytics, debugging, and compliance.

```ts
type ProductEvent = {
  id: string;
  eventName:
    | "candidate.discovered"
    | "candidate.contacted"
    | "candidate.replied"
    | "candidate.opted_out"
    | "consent.granted"
    | "consent.withdrawn"
    | "profile.created"
    | "profile.approved"
    | "role.activated"
    | "match.scored"
    | "match.reviewed"
    | "candidate.interested"
    | "employer.interested"
    | "introduction.sent"
    | "interview.completed"
    | "candidate.hired"
    | "placement.created"
    | "invoice.issued"
    | "invoice.paid"
    | "trust.complaint_created"
    | "privacy_request.completed";
  entityType: string;
  entityId: string;
  actorType: "candidate" | "employer" | "operator" | "ai" | "system";
  actorId?: string;
  occurredAt: Date;
  metadata: Record<string, unknown>;
  correlationId: string;
};
```

Recommended event funnels:

```text
discovered
  → contacted
  → delivered
  → replied
  → consented
  → represented
  → opportunity shown
  → candidate interested
  → employer interested
  → introduction
  → interview
  → offer
  → hire
  → paid placement
```

Trust funnel:

```text
contacted
  → opt-out
  → spam report
  → privacy complaint
  → deletion request
  → repeat contact after suppression
```

The second funnel should be monitored alongside revenue. A growing placement funnel with a deteriorating trust funnel is not healthy marketplace growth.

---

## 8. Minimum viable database

If this product were rebuilt from scratch, the first production schema could be limited to:

1. `users`
2. `organizations`
3. `organization_members`
4. `companies`
5. `roles`
6. `candidates`
7. `candidate_profiles`
8. `candidate_representations`
9. `consent_records`
10. `data_provenance`
11. `outreach`
12. `conversations`
13. `messages`
14. `matches`
15. `candidate_packets`
16. `interest_signals`
17. `introductions`
18. `interviews`
19. `hiring_outcomes`
20. `placements`
21. `fee_agreements`
22. `invoices`
23. `privacy_requests`
24. `trust_events`
25. `audit_events`

Do not start with a single `candidates` table containing a profile blob and a `consent = true/false` column. That shape cannot answer the core operational questions:

- Where did each attribute come from?
- Which purpose did the candidate authorize?
- Was this person contacted before or after consent?
- Which employer saw which version of the profile?
- Which introduction generated the placement fee?
- Was the candidate suppressed before the message was sent?

---

## 9. Highest-value product controls

1. **Identity disclosure**
   - Every sender has an `AgentIdentity`.
   - AI participation and the relationship to the platform are disclosed.
   - Sender domains and channel addresses are verified and attributable.

2. **Purpose-bound consent**
   - Contact, profile creation, representation, role-specific sharing, and employer sharing are separate purposes.
   - Consent scopes can be revoked independently.

3. **Provenance on material claims**
   - Every education, employer, skill, location, and preference claim has a source and confidence.

4. **Two-sided opt-in**
   - High match score never substitutes for candidate or employer interest.

5. **Suppression enforcement**
   - Opt-out and deletion checks run immediately before every send and every share.

6. **Attributable revenue**
   - Placement and invoice records point back to introduction, match, role, fee terms, and both interest signals.

7. **Trust observability**
   - Complaint and opt-out metrics are segmented by campaign, template, sender identity, source, geography, and acquisition channel.

---

## 10. Inferred product architecture

```text
Channels
  Web app · Email · SMS/iMessage/WhatsApp · Employer Slack/portal
                              │
                    Conversation service
                              │
          Candidate agent + Employer workflow layer
               │                       │
     Representation / consent      Roles / requirements
               └──────────────┬──────────────┘
                              │
                    Matching and ranking
                              │
                 Mutual opt-in coordinator
                              │
       Introductions → Interviews → Outcomes → Placements
                              │
                    Billing and revenue ops

Cross-cutting:
Identity resolution · provenance · privacy · suppression · audit · trust analytics
```

The critical architectural conclusion is that **communication, consent, provenance, and matching cannot be separate afterthoughts**. They form one chain: source → permission → representation → match → share → introduction → placement.
