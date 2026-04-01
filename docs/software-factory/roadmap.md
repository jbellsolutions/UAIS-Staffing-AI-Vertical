# Software Factory Roadmap — What Exists, What's Next

> Last updated: April 2026

---

## 1. What's Built and Production-Ready

The Software Factory already has a substantial foundation — not vapor, not slides, not a prototype. These assets are built, documented, and ready for client-facing deployment.

### Resume Screening Agent — Full Production Blueprint

The complete agent blueprint lives at `solutions/automation-blueprints/resume-screening-agent.md` and includes:

- System architecture with Claude Code integration and direct ATS API connections
- Full prompt engineering — system prompts, scoring rubrics, bias-check prompts
- Configurable scoring across hard skills, soft skills, experience, credentials, location, and availability
- ATS integration specs for Bullhorn, Lever, Greenhouse, and JobAdder (read candidate data, write scores/notes back)
- Bias prevention framework — redacts names, photos, age indicators before scoring; adversarial bias audit prompt
- Compliance layer — EEOC/OFCCP adverse impact tracking, NYC Local Law 144 audit trail, Illinois AIVFA documentation
- Cost model — Haiku for initial filtering ($0.002/resume), Sonnet for detailed scoring ($0.015/resume)
- Test suite — sample resumes with expected scores for regression testing

This is not a concept. It is a deployable agent with architecture, prompts, scoring logic, integration specs, and compliance documentation.

### 17 Agency-Type Deployment Guides

Each agency deep dive covers a specific staffing vertical with:

- 7-10 pain points unique to that agency type
- Matched automation solutions using the 10-agent architecture
- 30-day deployment plan customized to that vertical
- ROI calculations based on real industry benchmarks
- Technology stack recommendations per agency type
- Case study references where available

Agency types covered: contingency staffing, healthcare staffing, IT/technology staffing, light industrial/warehouse, executive search, RPO providers, travel nursing, legal staffing, financial/accounting staffing, creative/marketing staffing, engineering staffing, government/cleared staffing, education staffing, hospitality staffing, retail staffing, nonprofit staffing, and hybrid/multi-vertical agencies.

### 67+ Pain Point Map

Across 13 categories, every significant operational pain point in staffing has been identified and mapped to automation solutions:

- Candidate sourcing and pipeline
- Resume screening and qualification
- Interview coordination
- Compliance and credentialing
- Client development and retention
- Recruiter productivity and admin burden
- Post-placement and redeployment
- Analytics and reporting
- Market intelligence
- Communication management
- Data quality and ATS hygiene
- Billing and timesheets
- Onboarding and documentation

### Competitive Analysis

Full market landscape analysis showing the white space UAIS occupies — the gap between enterprise AI platforms ($50K-$500K/year), generic automation agencies (n8n/Zapier shops with zero staffing expertise), ATS built-in AI (limited to one vendor's walls), and the cost of hiring in-house AI developers ($120K-$180K salary with no staffing domain knowledge).

### 4 Case Studies

Real deployment results with before/after metrics across:

1. IT staffing agency — time-to-fill and submittal velocity improvements
2. Healthcare staffing agency — compliance gap elimination and credential processing speed
3. Light industrial agency — no-show rate reduction and shift fill time compression
4. Executive search firm — candidate quality scoring and client communication cadence

### Solutions Architecture

The complete 10-agent architecture documented in `docs/research/solutions-architecture.md`:

- 10 AI agents built with Claude Code as the primary tool
- OpenClaw as the 24/7 autonomous operations layer
- n8n for scheduling, cron jobs, and monitoring dashboards
- Direct ATS API integration (Bullhorn, Lever, Greenhouse, JobAdder) — no middleware
- Shared data model with JSON schemas for candidates, job orders, placements, clients, and activities
- Cost analysis per agent (Haiku for high-volume, Sonnet for complex reasoning, Batch API where latency permits)
- Security and compliance framework (PII handling, ephemeral processing, GDPR considerations)
- Docker-based deployment with one-command setup

### 3-Tier Offer Suite

Complete commercial packaging at `offer-packages/staffing-offer-suite.md`:

- Starter ($1,000 one-time + $300/mo) — AI Integrator + training + core agents
- Professional ($2,000 one-time + $300/mo) — adds 24/7 Operations Hub (OpenClaw)
- Enterprise ($5,000 one-time + $300/mo) — adds AI SDR team + Paperclip business operations
- Guarantees: double output or full refund, placement guarantee, zero lock-in
- Objection handling for every common pushback
- Add-on services: compliance monitoring, ATS migration, market intelligence, team training, VMS integration

---

## 2. In Development (Q2 2026)

### 9 Remaining Agent Blueprints

Each blueprint will match the depth of the Resume Screening Agent — full architecture, prompts, scoring logic, integration specs, compliance considerations, cost models, and test suites. Priority order based on deployment demand and revenue impact:

**1. Candidate Sourcing Pipeline** — Highest impact, needed for live demos
- Multi-source AI sourcing via Apify + Claude across job boards, LinkedIn, GitHub
- Automated Boolean string generation for niche roles
- Candidate enrichment — contact info, social profiles, recent activity
- Deduplication against existing ATS database
- Passive candidate identification and interest scoring

**2. Compliance Monitoring Agent** — Healthcare agencies are first adopters
- I-9 expiration tracking with 90/60/30-day alerts
- State-specific license monitoring across all placement states
- Background check and drug screen status management
- AI hiring law compliance (NYC LL144, Illinois AIVFA, Colorado AI Act, EU AI Act)
- Audit-ready reporting on demand

**3. Client Outreach Agent (AI SDR)** — Drives agency revenue directly
- Company research and hiring signal detection
- Hyper-personalized email sequences referencing open roles, growth signals, industry context
- Multi-channel coordination (email, LinkedIn, phone, SMS)
- Reply classification and lead scoring
- Automatic handoff to human recruiters for qualified prospects

**4. Post-Placement Follow-Up Agent** — Retention is the silent killer
- Automated check-in sequences at Day 1, 7, 30, 60, 90
- Sentiment analysis on candidate responses
- Early warning system for placement at risk
- Redeployment trigger when assignment end dates approach
- Client satisfaction pulse checks

**5. Interview Scheduling Agent** — Universal pain point across all agency types
- Multi-party availability parsing from email/calendar
- Automated scheduling with confirmation and reminders
- Time zone handling for remote/distributed interviews
- Rescheduling workflow with conflict detection
- Calendar integration (Google Calendar, Outlook, Calendly)

**6. Analytics & Reporting Agent** — Agency owners need visibility yesterday
- Automated weekly/monthly report generation
- Fill rate tracking by client, recruiter, job type
- Pipeline velocity metrics and bottleneck identification
- Revenue forecasting based on current pipeline
- Recruiter performance scorecards

**7. Market Intelligence Agent** — Premium tier differentiator
- Real-time salary benchmarks by role, location, experience
- Demand forecasting by skill set and geography
- Competitor activity monitoring (job postings, growth signals)
- Skill gap analysis for emerging roles
- Monthly market report generation for client sharing

**8. Client Health & Retention Agent** — Enterprise value play
- Multi-dimensional health scoring per client
- Churn prediction based on engagement patterns, fill rates, communication frequency
- Automated intervention triggers (declining health score sends alert + recommended action)
- Relationship mapping across client stakeholders
- Contract renewal tracking and proactive outreach

**9. Recruiter Productivity Agent** — Management layer
- Daily briefing generation with prioritized task list
- Activity summarization across all channels
- Time allocation analysis (admin vs. revenue-generating activities)
- Peer benchmarking (anonymized) for coaching conversations
- Bottleneck detection in individual workflows

### n8n Orchestration Workflow Templates

- Scheduling workflows — cron-based triggers for agent execution
- Monitoring dashboards — visual status of all running agents
- Error handling workflows — alerting, retry logic, escalation
- Data pipeline workflows — ATS sync, enrichment, cleanup
- Webhook receivers — for ATS events, email responses, calendar changes

### Implementation Playbooks per Agency Type

Taking the 17 agency deep dives and turning them into step-by-step implementation guides with:

- Pre-deployment checklist
- Agent selection and configuration guide
- Week-by-week deployment timeline
- KPI tracking templates
- Troubleshooting guides for common issues per vertical

---

## 3. Planned (Q3-Q4 2026)

### Demo Sandbox Environments

- Bullhorn sandbox account populated with realistic staffing data (100+ candidates, 20+ clients, 50+ job orders)
- Lever sandbox with parallel dataset for multi-ATS demo capability
- Pre-configured agents running against sandbox data for live prospect demos
- One-click reset to return sandbox to baseline state
- Video recording capability for remote demo delivery

### Video Walkthrough Series

- Agent-by-agent deployment tutorials (10 videos, 15-20 minutes each)
- ATS integration setup guides per platform (Bullhorn, Lever, Greenhouse, JobAdder)
- n8n workflow customization walkthroughs
- Common troubleshooting scenarios
- Advanced configuration for power users

### AI Integrator Certification Curriculum

- Level 1: Foundations — Claude Code basics, ATS API fundamentals, staffing industry knowledge
- Level 2: Agent Deployment — configuring, deploying, and monitoring the 10-agent suite
- Level 3: Custom Development — building new agents, advanced prompt engineering, cross-system integration
- Certification exam with practical assessment
- Continuing education requirements (quarterly updates)

### Agency Benchmarking Dashboard

- Cross-deployment performance comparison (anonymized)
- Benchmarks by agency type, size, and ATS
- Identify top-performing configurations for each vertical
- Track improvement trajectories over time
- Exportable benchmark reports for agency owners

### Industry-Specific Compliance Packs

- Healthcare — Joint Commission, CMS, state nursing board requirements, credentialing timelines
- Financial services — FINRA, SEC, state licensing, background check depth requirements
- Government/cleared — security clearance tracking, ITAR, DFARS, FedRAMP considerations
- Education — state teaching certification, background check requirements, Title IX
- International — work visa tracking, country-specific labor law compliance

---

## 4. Future Vision (2027)

### Self-Service Deployment for Starter Tier

Wizard-guided setup that walks agency owners through ATS connection, agent selection, and configuration without requiring an AI Integrator for the initial deployment. The Integrator shifts from "builder" to "optimizer" for Starter tier.

### Multi-Language Support

Support for offshore and nearshore staffing agencies operating across language boundaries — candidate communication in Spanish, Portuguese, Tagalog, and other high-demand languages for the staffing industry. Resume screening that handles multilingual CVs.

### Advanced Analytics with Predictive Modeling

Move beyond descriptive analytics to predictive — forecast which candidates are most likely to accept offers, which placements are at risk of early termination, which clients are approaching churn, and which market segments are about to surge in demand.

### Agency-to-Agency Marketplace

A marketplace where agencies can share (or sell) custom agents, workflow templates, and scoring rubrics. A healthcare agency that built a stellar credentialing workflow can package it for other healthcare agencies. Network effects at the template level.

### Integration with Emerging ATS Platforms

As new ATS platforms gain market share, build and maintain adapters for each. Priority based on client demand. The adapter architecture is designed for extensibility — adding a new ATS is a configuration exercise, not a rebuild.

---

## How to Read This Roadmap

- **Built and Production-Ready** = deployed, tested, documented, client-facing
- **In Development** = actively being built, expected delivery within the quarter
- **Planned** = scoped and scheduled, dependent on deployment learnings from Q2
- **Future Vision** = directional, will be refined based on market response and client feedback

The roadmap is a living document. Priorities shift based on real deployment data — what agencies actually need, not what we theorize they need.
