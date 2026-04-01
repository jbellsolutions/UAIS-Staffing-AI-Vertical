# Software Factory Architecture -- Technical Overview

> **Document Type:** Technical Architecture
> **Last Updated:** 2026-04-01
> **Audience:** Agency CTOs, technical evaluators, AI Integrators

---

## 1. Architecture Overview

The Software Factory runs on a five-layer architecture. Each layer has a specific role. Together, they form a complete automation system that is custom-configured for each agency deployment.

```
┌─────────────────────────────────────────────────────────┐
│             LAYER 5: AI INTEGRATOR (Human)               │
│       Strategy . Configuration . Optimization            │
│  Learns your agency . Configures agents . Trains team    │
├─────────────────────────────────────────────────────────┤
│           LAYER 4: 10 AI AGENTS (Claude Code)            │
│  Resume Screening . Sourcing . Outreach . Scheduling     │
│  Compliance . Market Intel . Client Health . Recruiter   │
│  Productivity . Post-Placement . Analytics & Reporting   │
├─────────────────────────────────────────────────────────┤
│            LAYER 3: ORCHESTRATION (n8n)                  │
│   Cron Jobs . Webhooks . Event Triggers . Monitoring     │
│         Visual Dashboard . Workflow Builder               │
├─────────────────────────────────────────────────────────┤
│         LAYER 2: AUTONOMOUS OPS (OpenClaw)               │
│   24/7 Execution . Self-Healing . Proactive Alerts       │
│     Natural Language Interface . Audit Trail              │
├─────────────────────────────────────────────────────────┤
│          LAYER 1: DATA & INTEGRATIONS                    │
│  Bullhorn . Lever . Greenhouse . JobAdder . iCIMS        │
│  Gmail/Outlook . LinkedIn . Indeed . ZipRecruiter        │
│  Apollo . Apify . Calendly . Twilio . Slack              │
└─────────────────────────────────────────────────────────┘
```

**How the layers work together:** Data enters at Layer 1 from your ATS and connected systems. Layer 2 (OpenClaw) keeps the system running autonomously around the clock. Layer 3 (n8n) handles scheduling and event triggers. Layer 4 is where the intelligence lives -- 10 Claude Code agents that reason about your recruiting data. Layer 5 is your AI Integrator, the human expert who configures, optimizes, and trains your team.

---

## 2. Layer 1: Data and Integrations

**Principle:** Your ATS is the source of truth. Always.

Every agent reads from your ATS and writes back to your ATS. No shadow databases. No data silos. If a candidate gets scored, that score goes into your ATS. If an interview gets scheduled, it shows up in your ATS. Your recruiters never have to check two systems.

### ATS Connections (Direct API -- No Middleware)

Connections are built with Claude Code using direct REST API calls. No Zapier. No Make. No middleware that breaks when the vendor changes their API.

| ATS | Auth Method | Capabilities | Adapter Status |
|-----|-------------|--------------|----------------|
| Bullhorn | OAuth 2.0 + session tokens | Full CRUD on candidates, jobs, placements, notes, activities | Production-ready |
| Lever | API key | Candidates, opportunities, interviews, offers | Production-ready |
| Greenhouse | Basic auth (API token) | Candidates, applications, jobs, scorecards | Production-ready |
| JobAdder | OAuth 2.0 | Candidates, jobs, placements, companies | Production-ready |
| Crelate | API key | Contacts, companies, jobs, activities | Production-ready |
| iCIMS | OAuth 2.0 | Candidates, jobs, workflows | Adapter available |
| Avionte | API key | Candidates, jobs, timesheets | Adapter available |
| TempWorks | API key | Employees, assignments, timesheets | Adapter available |
| Generic REST | Configurable | Any ATS with a REST API | Template available |

### Additional Integrations

**Email:** Gmail API, Microsoft Graph API (Outlook), Instantly, Smartlead. Agents send personalized emails, track opens/replies, and categorize responses.

**Calendar:** Google Calendar API, Microsoft Graph (Outlook Calendar), Calendly, Cal.com. The Interview Scheduling Agent coordinates availability across all parties.

**Job Boards:** LinkedIn (via Apify scrapers), Indeed API, ZipRecruiter API, Dice API, Monster. The Candidate Sourcing Pipeline searches across all sources simultaneously.

**Data Enrichment:** Apollo.io for contact data and company intelligence. Apify for web scraping and profile enrichment. LinkedIn Sales Navigator data (via compliant scraping).

**Communication:** Slack and Microsoft Teams for internal alerts. Twilio for SMS/voice automation. SendGrid for transactional email.

**Background Checks:** Checkr, Sterling, GoodHire. The Compliance Agent tracks check statuses and flags delays.

---

## 3. Layer 2: Autonomous Operations (OpenClaw)

**Available on:** Professional and Enterprise tiers.

OpenClaw is the Operations Hub. It is a dedicated cloud instance running your AI workflows 24/7. Think of it as the tireless operations manager who never sleeps, never forgets, and never drops the ball.

### What It Does

**Autonomous Execution:** You tell it what to do in plain English. "Follow up with every candidate who hasn't responded in 3 days." Done. "Send credential renewal reminders to all nurses expiring in 30 days." Done. "Generate this week's fill rate report for the VP." Done.

**Self-Healing:** When something breaks -- an API credential expires, a webhook fails, a data format changes -- OpenClaw detects the failure, diagnoses the cause, and either fixes it automatically or escalates to the AI Integrator with a clear description of what went wrong and what it needs.

**Proactive Monitoring:** OpenClaw does not wait for problems. It watches for:
- Credential expirations approaching (API keys, OAuth tokens)
- Stale candidate records that need re-engagement
- Compliance gaps before they become violations
- Client churn signals (declining communication, missed meetings, falling fill rates)
- Recruiter productivity drops that indicate workflow bottlenecks

**Communication:** OpenClaw talks to your team via Slack, SMS, or email. Morning briefings to each recruiter. End-of-day summaries to managers. Instant alerts for high-priority events (hot candidate, compliance deadline, client risk).

### Infrastructure

- Runs on dedicated cloud compute (not shared infrastructure)
- Full audit trail of every action taken
- Mobile-accessible dashboard for monitoring
- Human-in-the-loop for critical decisions (placement offers, client communications above a configurable threshold)
- API rate limiting and cost monitoring built in

---

## 4. Layer 3: Orchestration (n8n)

**Available on:** All tiers.

n8n is the scheduler and trigger layer. It decides WHEN agents run and WHAT triggers them. It is not the intelligence layer -- Claude Code agents handle the reasoning. n8n handles the timing and plumbing.

### Trigger Types

**Cron Jobs (Time-Based):**
- Run resume screening every morning at 8:00 AM against all new applications from overnight
- Generate recruiter daily briefings at 7:30 AM
- Send compliance expiration reports every Monday at 9:00 AM
- Run client health scoring every Friday at 4:00 PM

**Webhooks (Event-Based):**
- New job order created in ATS --> trigger Candidate Sourcing Pipeline
- Candidate status changed to "Placed" --> trigger Post-Placement Follow-Up sequence
- New resume uploaded --> trigger Resume Screening Agent
- Client contract renewal date within 60 days --> trigger Client Health check

**Manual Triggers:**
- Recruiter clicks "Source candidates for this role" in the dashboard
- Manager requests an on-demand analytics report
- AI Integrator runs a test workflow during configuration

### Monitoring Dashboard

The n8n dashboard is designed so agency owners and managers can SEE what is running without needing technical knowledge:

- Visual workflow execution history (green = success, red = failure, yellow = needs review)
- Active agent status indicators
- Error logs with plain-language descriptions
- Cost tracking per workflow per day
- One-click enable/disable for any workflow

### Why n8n

n8n is self-hosted. No vendor lock-in. The agency owns their n8n instance. If they leave UAIS, the workflows keep running. It is open source, has a visual builder, and is simple enough that a non-technical person can understand the flow of data after a 30-minute walkthrough. But the important thing to understand: n8n is secondary. Claude Code agents are the intelligence. n8n just tells them when to run.

---

## 5. Layer 4: 10 AI Agents (Claude Code)

**Available on:** Varies by tier (see agent specifications document for details).

This is the intelligence layer. Each agent is purpose-built for a specific staffing function using Claude Code -- the primary tool in the UAIS stack. Agents do not just route data. They REASON about it.

The difference between a Claude Code agent and a simple automation: an automation matches keywords. A Claude Code agent understands that "managed 12 engineers" implies leadership experience even when "management" is not listed as a skill. It understands that 3 years of React Native counts toward a React requirement. It understands that a pastry chef applying for a DevOps role should get a fast rejection, not a deep analysis that wastes tokens.

### The 10 Agents

| # | Agent | One-Line Description |
|---|-------|---------------------|
| 1 | Resume Screening | Scores and ranks candidates against job requirements using two-tier Haiku/Sonnet evaluation |
| 2 | Candidate Sourcing Pipeline | Multi-source candidate discovery across job boards, LinkedIn, ATS database, and enrichment APIs |
| 3 | Client Outreach (AI SDR) | Multi-channel personalized prospecting to hiring managers and HR directors |
| 4 | Interview Scheduling | Automated availability matching and calendar coordination across all parties |
| 5 | Compliance Monitoring | Proactive tracking of credentials, licenses, certifications, and regulatory requirements |
| 6 | Market Intelligence | Real-time salary benchmarking, demand forecasting, and competitor monitoring |
| 7 | Client Health and Retention | Relationship scoring, churn prediction, and automated intervention triggers |
| 8 | Recruiter Productivity | Activity tracking, bottleneck identification, and daily workflow optimization |
| 9 | Post-Placement Follow-Up | Automated check-in sequences with sentiment detection and early warning |
| 10 | Analytics and Reporting | Centralized dashboards, KPI tracking, and on-demand report generation |

### Claude Model Routing Strategy

Not every task needs the most powerful model. The factory uses intelligent model routing to keep costs low and speed high:

| Model | Use Cases | Cost | Speed |
|-------|-----------|------|-------|
| Claude Haiku 4.5 | Resume parsing, email classification, data extraction, template filling, initial screening | ~$0.001/call | <1 second |
| Claude Sonnet 4.6 | Complex scoring, market analysis, churn prediction, personalized outreach, strategy recommendations | ~$0.01/call | 2-5 seconds |
| Batch API | Non-urgent processing: overnight resume batches, weekly analytics, monthly reports | 50% discount | Async (minutes to hours) |
| Prompt Caching | Repeated schemas, system prompts, scoring rubrics | Up to 90% savings on cached tokens | Same as base model |

---

## 6. Layer 5: AI Integrator (Human)

**Available on:** All tiers.

The AI Integrator is the strategist, the builder, and the trainer. They are the reason this system works where generic SaaS tools fail.

### What They Do

**Configure:** The AI Integrator connects the factory to your specific ATS, maps your custom fields, configures agents for your workflows, and tunes scoring rubrics for your job types. A healthcare staffing agency has fundamentally different screening criteria than an IT staffing firm. The AI Integrator handles that.

**Build:** Using Claude Code as their primary tool, the AI Integrator builds custom automations that do not exist in any off-the-shelf product. If you have a unique workflow -- say, a three-stage credentialing process for travel nurses -- they build it.

**Train:** During the 30-day bootcamp (Starter and Professional) or 90-day deployment (Enterprise), the AI Integrator trains your team on the system. Not just "here is how to click buttons" -- they teach your recruiters how to work alongside AI, how to interpret agent output, and how to provide feedback that makes the system smarter.

**Optimize:** After deployment, the AI Integrator reviews real performance data and tunes the system. If resume screening is rejecting too many qualified candidates, they adjust the scoring rubric. If outreach sequences are getting low reply rates, they modify the personalization prompts. Continuous improvement based on real results.

**Handle the Human Side:** Technology adoption fails when people resist it. The AI Integrator handles change management -- getting recruiter buy-in, addressing concerns, showing early wins, and making sure the team sees AI as a tool that helps them, not a threat that replaces them.

---

## 7. Data Flow Principles

### ATS as Source of Truth
Every agent reads from and writes back to the agency's ATS. There is no separate database that agents maintain on their own. If a recruiter opens their ATS, they see everything the agents have done.

### Candidate PII Stays in Agency Infrastructure
All candidate personally identifiable information stays within the agency's own infrastructure. When Claude API calls are made, data is processed ephemerally -- Anthropic does not store or train on the data. No candidate resumes, contact information, or placement records leave the agency's control.

### Agents Share Context Through a Unified Data Model
When the Resume Screening Agent scores a candidate, that score is available to the Interview Scheduling Agent, the Recruiter Productivity Agent, and the Analytics Agent. Agents do not operate in silos. A shared JSON schema ensures consistent data structures across all ten agents.

### Every Action Is Logged and Auditable
Every agent action produces a log entry: what was done, when, why, and what data was involved. This is critical for compliance-sensitive operations (healthcare staffing, government staffing) and for optimization (you cannot improve what you cannot measure).

### The System Is Additive
The factory enhances existing workflows. It does not replace them. Recruiters keep using their ATS the same way they always have. The AI agents work alongside the existing process, handling the repetitive parts while recruiters focus on relationships and judgment.

---

## 8. Security and Compliance

### Data Protection
- All API credentials encrypted at rest using AES-256
- TLS 1.3 for all data in transit
- Role-based access controls -- recruiters see their pipeline, managers see their team, owners see everything
- PII detection and redaction in agent logs
- Data retention policies configurable per agency

### Healthcare Staffing (HIPAA)
- HIPAA-aware agent configurations for agencies handling PHI
- Candidate health records processed in memory only, never persisted outside the ATS
- Business Associate Agreements (BAA) available for Enterprise tier
- Audit trail meets HIPAA access logging requirements

### Infrastructure Security
- SOC 2 Type II aligned infrastructure practices
- Penetration testing on deployment infrastructure
- Automated vulnerability scanning on all dependencies
- Incident response playbook for data breach scenarios

### Regulatory Compliance
- GDPR compliant candidate data handling (right to be forgotten, data portability, consent management)
- CCPA compliant for California-based agencies and candidates
- EEOC/OFCCP adverse impact monitoring in AI-assisted screening decisions
- AI hiring law compliance monitoring: NYC Local Law 144, Illinois AIVFA, Colorado AI Act, EU AI Act
- State-by-state salary transparency requirement tracking

### AI-Specific Safeguards
- Bias detection prompts built into the Resume Screening Agent
- Human-in-the-loop required for all final hiring decisions
- No autonomous candidate rejections without human review option
- Regular audit of agent scoring distributions to detect statistical bias
- Configurable guardrails on all outbound communications
