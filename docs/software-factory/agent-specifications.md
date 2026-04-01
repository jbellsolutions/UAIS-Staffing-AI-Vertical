# The 10 AI Agents -- Complete Specifications

> **Document Type:** Agent Technical Specifications
> **Last Updated:** 2026-04-01
> **Reference:** See `solutions/automation-blueprints/resume-screening-agent.md` for full implementation blueprint of Agent 1

---

## Agent Index

| # | Agent | Tier Availability |
|---|-------|-------------------|
| 1 | Resume Screening Agent | All tiers |
| 2 | Candidate Sourcing Pipeline | All tiers |
| 3 | Client Outreach Agent (AI SDR) | Enterprise (or $1K/mo add-on) |
| 4 | Interview Scheduling Agent | Professional and Enterprise |
| 5 | Compliance Monitoring Agent | Professional and Enterprise |
| 6 | Market Intelligence Agent | Enterprise |
| 7 | Client Health and Retention Agent | Enterprise |
| 8 | Recruiter Productivity Agent | Enterprise |
| 9 | Post-Placement Follow-Up Agent | Professional and Enterprise |
| 10 | Analytics and Reporting Agent | Enterprise (basic reporting in Professional) |

---

## Agent 1: Resume Screening Agent

**STATUS: PRODUCTION-READY -- Full blueprint at `solutions/automation-blueprints/resume-screening-agent.md`**

### Purpose

Automatically parse, score, and rank resumes against job requirements, eliminating the manual screening bottleneck that consumes 15+ recruiter hours per week.

### The Problem It Solves

A recruiter spends 5-10 minutes per resume doing initial screening. For a typical job order receiving 50-200 applications, that is 4-33 hours of screening time per role. Most of those resumes -- roughly 75% -- are unqualified. The recruiter is spending the majority of their screening time on candidates who will never get a call. Meanwhile, qualified candidates sit in the queue waiting, and the fastest agency to submit wins the fee.

Manual screening also introduces inconsistency. A recruiter at 8 AM evaluates differently than the same recruiter at 4 PM. Different recruiters apply different standards to the same role. There is no audit trail for why one candidate was advanced and another was not.

### How It Works

1. **Resumes arrive** via email attachment, ATS upload, or file drop to a watched folder. n8n triggers the screening pipeline on arrival or on a scheduled batch (e.g., every morning at 8 AM).
2. **Text extraction** converts PDF/DOCX to plain text using pdf-parse or mammoth. The ATS parser is used when available.
3. **Claude Haiku screens at volume.** Fast, cheap ($0.001/resume), accurate enough for first-pass filtering. Candidates scoring below 40/100 are auto-categorized as NO_MATCH.
4. **Borderline candidates (score 40-75) get re-screened with Claude Sonnet** for nuanced evaluation. Sonnet catches transferable skills, implicit experience, and context that Haiku might miss.
5. **Results are ranked and pushed to the ATS** with a structured scorecard: overall score, dimension scores (skills match, experience level, industry relevance, career trajectory, red flags), key strengths, concerns, suggested interview questions, and a 2-3 sentence summary written for a busy recruiter.

### Inputs

- Resume text (extracted from PDF/DOCX/TXT)
- Job requisition data from ATS: title, description, required skills, preferred skills, minimum experience, location requirement, salary range, certifications, clearance requirements
- Agency-specific scoring rubric configuration (weights per dimension, threshold overrides)

### Outputs

- Structured JSON scorecard per candidate (see blueprint for full schema)
- Ranked shortlist pushed to ATS pipeline stage
- Rejection notifications queued for unqualified candidates (configurable delay and messaging)
- Recruiter-ready talking points for qualified candidates
- Daily screening summary report

### Primary ATS Integration

- **Read:** Job requisition details, candidate records, resume attachments
- **Write:** Candidate scores, screening notes, pipeline stage updates, activity logs
- **Endpoints:** Bullhorn `/entity/Candidate`, `/entity/JobOrder`, `/entity/Note`; Lever `/opportunities`, `/candidates`; Greenhouse `/applications`, `/scorecards`

### Claude Model Routing

| Task | Model | Rationale |
|------|-------|-----------|
| Initial bulk screening (all resumes) | Haiku 4.5 | Speed and cost -- $0.001/resume, <1 second response |
| Borderline re-screening (score 40-75) | Sonnet 4.6 | Nuanced judgment on transferable skills, implicit experience |
| Obviously irrelevant resumes | Haiku 4.5 | Fast reject with brief explanation, minimal token usage |
| Batch overnight processing | Batch API (Haiku) | 50% cost reduction for non-urgent volume |
| Repeated job requisition schemas | Prompt Caching | Up to 90% savings on cached system prompt + rubric |

### Cost Estimate

- Small agency (50 resumes/day): ~$2-5/month
- Mid agency (200 resumes/day): ~$8-15/month
- Large agency (500+ resumes/day): ~$20-40/month
- Includes Haiku volume screening + Sonnet re-screening on ~20% of candidates

### Tier Availability

All tiers (Starter, Professional, Enterprise).

---

## Agent 2: Candidate Sourcing Pipeline

### Purpose

Automatically discover, enrich, and score candidates across multiple sources -- internal ATS database, job boards, LinkedIn, and enrichment APIs -- to build a qualified pipeline for every open requisition.

### The Problem It Solves

Recruiters spend 10+ hours per week manually searching LinkedIn, job boards, and their own ATS database. They search one source at a time. They use the same Boolean strings they have been using for years. They miss passive candidates who are not actively looking. They submit candidates from their ATS without checking if contact info is current. And when they find a good candidate, they spend another 20 minutes researching them before reaching out.

The result: a narrow candidate pool, stale data, and slow sourcing that lets competitors submit first.

### How It Works

1. **New job order triggers the pipeline.** When a job order is created or updated in the ATS (via n8n webhook), the Sourcing Pipeline activates.
2. **AI generates optimized search queries.** Claude Sonnet analyzes the job requirements and generates Boolean search strings, keyword combinations, and semantic search queries tailored to each source. Not generic templates -- queries built for this specific role.
3. **Multi-source search executes in parallel.** Internal ATS database search, LinkedIn (via Apify scraper), Indeed API, ZipRecruiter API, Dice API, and GitHub/Stack Overflow for technical roles.
4. **Candidate profiles are enriched.** Apollo.io adds current contact information, company details, and social profiles. Apify scrapers pull recent activity and publications. Stale ATS records get updated.
5. **Deduplication and scoring.** Candidates are matched against existing ATS records to prevent duplicates. Each candidate receives a relevance score using Claude Haiku, and the top candidates are pushed to the recruiter's review queue with enriched profiles and outreach suggestions.

### Inputs

- Job requisition data from ATS (title, skills, experience, location, salary range)
- Agency ATS candidate database (existing records)
- External source credentials (LinkedIn, Indeed, ZipRecruiter, Dice, Apollo, Apify)

### Outputs

- Ranked candidate list with relevance scores and enriched profiles
- New candidate records created in ATS (deduplicated)
- Updated contact information on stale ATS records
- Suggested outreach messages per candidate
- Source effectiveness metrics (which source produced the most qualified candidates per role)

### Primary ATS Integration

- **Read:** Job requisitions, existing candidate records, placement history
- **Write:** New candidate records, updated contact info, source tracking fields, activity logs
- **Endpoints:** Bullhorn `/search/Candidate`, `/entity/Candidate` (POST/PUT); Lever `/candidates` (POST), `/opportunities`; Greenhouse `/candidates` (POST)

### Claude Model Routing

| Task | Model | Rationale |
|------|-------|-----------|
| Search query generation | Sonnet 4.6 | Requires understanding of role nuances to generate effective queries |
| Candidate relevance scoring | Haiku 4.5 | Volume task -- score hundreds of profiles quickly |
| Profile enrichment analysis | Haiku 4.5 | Structured data extraction from enrichment results |
| Outreach message suggestions | Sonnet 4.6 | Personalization requires reasoning about candidate background |

### Cost Estimate

- Small agency (5-10 active reqs): ~$15-30/month
- Mid agency (20-50 active reqs): ~$40-80/month
- Large agency (100+ active reqs): ~$100-200/month
- Primary cost driver: external API usage (Apollo, Apify) rather than Claude tokens

### Tier Availability

All tiers (Starter, Professional, Enterprise).

---

## Agent 3: Client Outreach Agent (AI SDR)

### Purpose

Identify target companies, research their hiring patterns, and execute personalized multi-channel outreach sequences to generate new client conversations for the agency.

### The Problem It Solves

Most staffing agencies rely on their recruiters to do business development in addition to filling reqs. That means BD happens sporadically -- when a recruiter has a slow week, or when a manager pushes for it. The outreach is generic ("Hi, we're a staffing firm and we'd love to help you hire"). Response rates are 2-5%. There is no systematic prospecting engine.

Enterprise agencies have dedicated sales teams, but even they are limited to 20-30 personalized touches per day per person. At those volumes, pipeline building is slow and inconsistent.

### How It Works

1. **Prospector Agent identifies targets.** Monitors job posting data, funding announcements, headcount growth signals, and LinkedIn activity to find companies actively hiring in the agency's verticals.
2. **Research Agent builds intelligence packages.** For each target company: open roles, growth trajectory, hiring manager names and contact info, tech stack (for IT staffing), patient volume trends (for healthcare staffing), seasonal patterns.
3. **Email SDR Agent generates and sends personalized sequences.** Multi-touch email campaigns that reference the prospect's specific open roles, industry context, and growth signals. Not mail merge templates -- each email reads like it was written by a human who did their homework.
4. **Multi-channel follow-up.** LinkedIn SDR sends connection requests with personalized notes. Text SDR runs SMS sequences for leads who do not respond to email. Phone SDR generates call scripts with talking points for warm leads.
5. **Response handling and qualification.** AI classifies responses (interested, not now, not interested, wrong person). Interested leads get fast-tracked to the agency's human sales team with a full intelligence package.

### Inputs

- Target company criteria (industry, size, location, hiring velocity)
- Agency positioning and value propositions per vertical
- CRM/ATS client records (to avoid contacting existing clients)
- Email sending infrastructure (Instantly, Smartlead, or direct SMTP)

### Outputs

- 100-200+ personalized prospect touches per day across email, LinkedIn, SMS
- Qualified lead handoffs to human sales team with full context
- Response tracking and engagement analytics
- A/B test results across messaging variants, channels, and timing
- Pipeline metrics: touches, opens, replies, conversations, meetings booked

### Primary ATS Integration

- **Read:** Existing client records (to exclude from prospecting), placement history by client
- **Write:** New prospect records, activity logs, meeting scheduling data
- **Endpoints:** Bullhorn `/entity/ClientCorporation`, `/entity/Lead`; CRM integration for pipeline tracking

### Claude Model Routing

| Task | Model | Rationale |
|------|-------|-----------|
| Company research and signal detection | Sonnet 4.6 | Requires reasoning about multiple data points to identify signals |
| Email personalization | Sonnet 4.6 | Each email must read as human-written with specific references |
| Response classification | Haiku 4.5 | Binary/categorical classification at volume |
| LinkedIn message generation | Sonnet 4.6 | Short-form personalization requires quality over speed |
| Call script generation | Sonnet 4.6 | Talking points need strategic framing |

### Cost Estimate

- 100 touches/day: ~$80-120/month (Claude tokens + email infrastructure)
- 200 touches/day: ~$150-250/month
- Primary cost driver: email sending tool subscriptions ($30-100/month) plus Claude Sonnet for personalization

### Tier Availability

Enterprise tier included. Available as $1,000/month add-on for Starter and Professional.

---

## Agent 4: Interview Scheduling Agent

### Purpose

Automatically match availability across candidates, recruiters, and hiring managers, then coordinate interview scheduling with confirmations, reminders, and rescheduling -- reducing the typical 5-8 email scheduling chain to a single interaction.

### The Problem It Solves

Interview scheduling is one of the most universally hated tasks in recruiting. A recruiter sends an email to the candidate with available times. The candidate responds with different times. The recruiter checks the hiring manager's calendar. The hiring manager has conflicts. Back to the candidate. The candidate's first choice is now taken. Three days later, the interview is finally scheduled. Meanwhile, the candidate accepted a competing offer.

For high-volume staffing (light industrial, healthcare travel nursing), this problem multiplies. A recruiter scheduling 20 interviews a week can spend 10+ hours just on coordination.

### How It Works

1. **Trigger:** Candidate reaches "Schedule Interview" stage in ATS, or recruiter manually triggers scheduling.
2. **Calendar aggregation.** Agent reads availability from candidate (via email reply or scheduling link), recruiter calendar (Google/Outlook), and hiring manager calendar (Google/Outlook/Calendly).
3. **AI-powered matching.** Claude Haiku finds optimal time slots considering: all parties' availability, time zone differences, interview duration requirements, buffer time between interviews, and hiring manager preferences (e.g., "no interviews before 10 AM").
4. **Confirmation and calendar holds.** Agent sends calendar invitations to all parties, including interview details (format, location/link, preparation instructions). Creates calendar holds to prevent double-booking.
5. **Rescheduling and reminders.** If anyone needs to reschedule, the agent finds the next available slot automatically. Sends reminders at 24 hours and 1 hour before the interview.

### Inputs

- Candidate contact information and timezone
- Recruiter and hiring manager calendar access (Google Calendar API, Microsoft Graph)
- Interview requirements: type (phone, video, onsite), duration, number of rounds
- Scheduling preferences and constraints

### Outputs

- Confirmed interview appointments with calendar invitations to all parties
- Reminder notifications at configurable intervals
- Rescheduling handled automatically when conflicts arise
- Interview scheduling metrics: average time-to-schedule, no-show rates, rescheduling frequency
- ATS updated with interview date, time, and details

### Primary ATS Integration

- **Read:** Candidate records, interview stage triggers, hiring manager contacts
- **Write:** Interview records, appointment details, activity logs
- **Endpoints:** Bullhorn `/entity/Appointment`; Lever `/interviews`; Greenhouse `/scheduled_interviews`; Google Calendar API `/events`; Microsoft Graph `/events`

### Claude Model Routing

| Task | Model | Rationale |
|------|-------|-----------|
| Availability parsing from free-text emails | Haiku 4.5 | Structured extraction from natural language |
| Optimal slot matching | Haiku 4.5 | Deterministic logic with simple constraint satisfaction |
| Confirmation message generation | Haiku 4.5 | Template-based with light personalization |
| Rescheduling negotiation | Sonnet 4.6 | Handling complex multi-party rescheduling requires reasoning |

### Cost Estimate

- Small agency (20 interviews/week): ~$3-5/month
- Mid agency (50-100 interviews/week): ~$8-15/month
- Large agency (200+ interviews/week): ~$20-30/month
- Minimal Claude token cost; primary cost is calendar API calls (free tier sufficient for most agencies)

### Tier Availability

Professional and Enterprise tiers.

---

## Agent 5: Compliance Monitoring Agent

### Purpose

Proactively track credentials, licenses, certifications, and regulatory requirements across all placed and active candidates, alerting the agency before compliance gaps become violations.

### The Problem It Solves

Compliance tracking is a spreadsheet nightmare. A healthcare staffing agency placing nurses across 15 states needs to track RN licenses, BLS/ACLS certifications, TB tests, drug screens, background checks, I-9 verification, and state-specific requirements -- for every single candidate. Licenses expire. Certifications lapse. States change requirements. One missed expiration can result in a candidate working without a valid license, exposing the agency to massive liability.

The manual approach: a recruiter or compliance coordinator maintains a spreadsheet, sets calendar reminders, and checks on things when they remember. Things fall through cracks. Agencies report 3-5 compliance gaps per quarter on average. Each gap is a potential lawsuit, a client relationship risk, and a regulatory headache.

### How It Works

1. **Initial audit.** When activated, the agent scans all active candidate and placement records in the ATS for credentials, licenses, certifications, and compliance documents. Builds a complete compliance profile per candidate.
2. **Expiration monitoring.** Continuous monitoring of all tracked items with proactive alerts at configurable intervals (90, 60, 30, 14, 7 days before expiration).
3. **Multi-jurisdiction tracking.** For agencies placing across state lines, the agent tracks state-specific license requirements, Nurse Licensure Compact (NLC) status for healthcare, and jurisdiction-specific compliance rules.
4. **Automated candidate notifications.** When credentials approach expiration, the agent sends personalized renewal reminders to candidates with specific instructions on what to renew and where.
5. **Compliance dashboard and reporting.** Real-time compliance status across all placements. Audit-ready reports generated on demand. Trend analysis showing recurring compliance gaps.

### Inputs

- Candidate records with credential/license fields from ATS
- State licensing board data (via web scraping or API where available)
- Federal compliance requirements (I-9, E-Verify, WOTC)
- Industry-specific rules (healthcare: Joint Commission, CMS; government: security clearances; financial: FINRA)
- Agency-defined custom compliance rules

### Outputs

- Proactive expiration alerts at 90/60/30/14/7 days
- Candidate renewal reminder emails with specific instructions
- Real-time compliance dashboard showing status across all placements
- Audit-ready compliance reports (PDF/CSV) generated on demand
- I-9 verification tracking and reverification alerts
- Workers' compensation classification monitoring
- Trend reports identifying recurring compliance gaps

### Primary ATS Integration

- **Read:** Candidate credential fields, placement records, license/certification data
- **Write:** Compliance status updates, alert activity logs, expiration tracking fields
- **Endpoints:** Bullhorn `/entity/CandidateCertification`, `/entity/Placement`; custom fields via ATS-specific endpoints

### Claude Model Routing

| Task | Model | Rationale |
|------|-------|-----------|
| Document classification (is this a valid license?) | Haiku 4.5 | Pattern matching on document types |
| Multi-jurisdiction rule evaluation | Sonnet 4.6 | Complex logic across 41+ state regulatory frameworks |
| Renewal reminder personalization | Haiku 4.5 | Template-based messaging with candidate details |
| Compliance gap analysis and recommendations | Sonnet 4.6 | Strategic reasoning about risk and remediation |

### Cost Estimate

- Small agency (50 active placements): ~$10-20/month
- Mid agency (200 active placements): ~$30-50/month
- Large agency (500+ active placements): ~$60-100/month
- Cost is driven primarily by the number of active placements being monitored, not by candidate database size

### Tier Availability

Professional and Enterprise tiers. Available as part of the Compliance Monitoring Package ($500/month add-on) for Starter.

---

## Agent 6: Market Intelligence Agent

### Purpose

Deliver real-time salary benchmarking, demand forecasting, competitor monitoring, and bill rate optimization so the agency can price competitively, spot trends early, and win client conversations with data.

### The Problem It Solves

Most agencies price reactively. They set bill rates based on what they charged last year, adjusted by gut feel. They learn about market shifts after they have already lost deals. They have no systematic way to know that AI engineer demand in Austin just spiked 40%, or that a competitor opened an office in their market, or that average RN travel pay rates in Florida dropped 15% this quarter.

This lack of market intelligence means agencies leave money on the table (underpricing), lose candidates to competitors (not matching market rates), and miss emerging opportunities (not seeing demand shifts early enough).

### How It Works

1. **Salary data aggregation.** Pulls compensation data from job postings (Indeed, LinkedIn, Glassdoor), BLS statistics, and proprietary data sources. Normalizes by role, location, experience level, and industry.
2. **Demand forecasting.** Analyzes job posting volume trends, company hiring patterns, funding announcements, and macroeconomic indicators to predict demand shifts 30-90 days out by role and geography.
3. **Competitor monitoring.** Tracks competitor job postings, office openings, LinkedIn employee counts, and market positioning to detect competitive moves.
4. **Bill rate optimization.** Combines salary data, demand signals, and agency historical margins to recommend optimal bill and pay rates for each role and market.
5. **Monthly market briefing.** Generates an AI-written market report the agency can share with clients to demonstrate expertise and data-driven advisory.

### Inputs

- Job posting data from major boards (Indeed, LinkedIn, ZipRecruiter, Dice)
- Agency historical placement data (pay rates, bill rates, margins by role/market)
- BLS employment statistics and wage data
- Company funding and growth data (Crunchbase, PitchBook via Apify)
- Competitor intelligence sources

### Outputs

- Real-time salary benchmarks by role, location, experience level, and industry
- 30/60/90-day demand forecasts by skill set and geography
- Competitor activity alerts (new job postings, office moves, hiring surges)
- Bill rate recommendations with margin analysis
- Monthly market intelligence report (PDF) for client-facing use
- Wage inflation tracking and trend alerts

### Primary ATS Integration

- **Read:** Historical placement data (pay rates, bill rates, margins), job order data
- **Write:** Market data enrichment on job orders, recommended rate fields
- **Endpoints:** Bullhorn `/entity/JobOrder` (custom fields for market data); analytics queries across placement history

### Claude Model Routing

| Task | Model | Rationale |
|------|-------|-----------|
| Job posting data extraction and normalization | Haiku 4.5 | Structured extraction from semi-structured job posting data |
| Demand trend analysis and forecasting | Sonnet 4.6 | Requires reasoning across multiple data signals |
| Competitor intelligence synthesis | Sonnet 4.6 | Analysis of qualitative and quantitative competitive signals |
| Market report generation | Sonnet 4.6 | Long-form analytical writing with data citations |
| Bill rate optimization calculations | Haiku 4.5 | Primarily mathematical with simple rule application |

### Cost Estimate

- Base infrastructure (data collection, APIs): ~$100-200/month
- Claude processing: ~$30-60/month
- Total: ~$130-260/month depending on number of markets and roles tracked

### Tier Availability

Enterprise tier included. Available as part of the Staffing Market Intelligence Dashboard ($500/month add-on) for Professional.

---

## Agent 7: Client Health and Retention Agent

### Purpose

Score client relationships, predict churn risk, generate QBR preparation documents, and trigger automated intervention workflows when a client relationship shows signs of deterioration.

### The Problem It Solves

Client churn in staffing runs 20-30% annually. Most agencies do not know a client is at risk until the client stops returning calls. By then it is too late. There are signals -- declining fill rates, fewer job orders, slower communication responses, negative feedback on candidates -- but no one is systematically tracking them.

The cost of losing a client is enormous. Acquiring a new client costs 5-7x more than retaining an existing one. A $500K/year client lost to churn means $500K in revenue that has to be replaced, plus the 6-12 months it takes to ramp a new client to that level.

### How It Works

1. **Health score calculation.** The agent computes a composite health score (0-100) for each client based on: fill rate trend, job order volume trend, time-to-fill trend, communication frequency, candidate feedback ratings, payment timeliness, and NPS/satisfaction signals.
2. **Churn prediction.** Using historical patterns from the agency's data, Claude Sonnet identifies clients whose health score trajectory matches clients who churned in the past. Flags high-risk clients before the churn decision is made.
3. **Automated intervention triggers.** When a health score drops below configurable thresholds, the agent automatically triggers intervention workflows: schedule a check-in call, send a proactive report showing recent performance, assign the account manager to review the relationship.
4. **QBR preparation.** Before quarterly business reviews, the agent generates a comprehensive prep document: performance metrics vs. targets, placement highlights, market insights relevant to the client's industry, and recommended discussion topics.
5. **Continuous relationship scoring.** Scores update in real-time as new data enters the ATS. Every placement, every candidate rejection, every communication (or lack thereof) adjusts the health score.

### Inputs

- Client records and history from ATS (job orders, placements, fill rates, revenue)
- Communication logs (email frequency, response times, meeting cadence)
- Candidate feedback and placement satisfaction data
- Payment and invoice history
- Agency-defined client tier classifications

### Outputs

- Client health scores (0-100) updated in real-time
- Churn risk flags with probability estimates and contributing factors
- Automated intervention workflows triggered by score thresholds
- QBR prep documents (PDF) with performance data, insights, and talking points
- Weekly client health summary report for account managers
- Client lifetime value projections

### Primary ATS Integration

- **Read:** Client records, job order history, placement history, activity logs, notes
- **Write:** Client health score custom fields, intervention activity logs, QBR document links
- **Endpoints:** Bullhorn `/entity/ClientCorporation`, `/entity/JobOrder`, `/entity/Placement`, `/entity/Note`

### Claude Model Routing

| Task | Model | Rationale |
|------|-------|-----------|
| Health score calculation | Haiku 4.5 | Weighted formula application across structured data |
| Churn pattern recognition | Sonnet 4.6 | Requires reasoning about complex behavioral patterns |
| QBR document generation | Sonnet 4.6 | Long-form strategic analysis and writing |
| Intervention recommendation | Sonnet 4.6 | Context-dependent decision making |

### Cost Estimate

- Small agency (20-50 active clients): ~$15-30/month
- Mid agency (50-200 active clients): ~$30-60/month
- Large agency (200+ active clients): ~$50-100/month

### Tier Availability

Enterprise tier.

---

## Agent 8: Recruiter Productivity Agent

### Purpose

Track recruiter activity metrics, identify bottlenecks, generate daily priority briefings, suggest workflow optimizations, and provide coaching insights for managers.

### The Problem It Solves

Agency managers have limited visibility into recruiter productivity until placements (or lack thereof) show up in the monthly report. By then, a struggling recruiter has already wasted 4 weeks. A top performer who is burning out has already started looking at other agencies.

Most agencies track output (placements, revenue) but not activity (calls made, submittals sent, interviews scheduled, follow-ups completed). Without activity data, managers cannot diagnose WHY a recruiter is underperforming. Is it a sourcing problem? A screening problem? A closing problem? Without data, coaching is guesswork.

### How It Works

1. **Activity tracking.** The agent monitors recruiter activities logged in the ATS: calls, emails, submittals, interviews scheduled, offers extended, placements made. Supplemented by calendar data and communication logs.
2. **Daily priority briefing.** Every morning at 7:30 AM, each recruiter receives a personalized briefing: top priority tasks for the day, candidates requiring follow-up, interviews to prepare for, job orders needing attention, and a motivational snapshot of their week-to-date performance.
3. **Bottleneck identification.** The agent analyzes the recruiter's pipeline conversion rates at each stage (sourced > screened > submitted > interviewed > offered > placed) and identifies where candidates are dropping off. "You screened 40 candidates this week but only submitted 3 -- your submit rate is 7.5% vs. the team average of 15%."
4. **Workflow optimization suggestions.** Based on conversion data and best practices, the agent suggests specific improvements: "Your time-to-submit averages 3 days. Top performers submit within 24 hours. Consider pre-screening candidates during sourcing to reduce the screening-to-submit gap."
5. **Manager coaching dashboard.** Aggregated view of team performance with drill-down per recruiter. Flags underperformers early. Highlights top performers and what they are doing differently.

### Inputs

- Recruiter activity data from ATS (calls, emails, submittals, interviews, placements)
- Calendar data (meetings, time allocation)
- Pipeline stage data and conversion metrics
- Team benchmarks and historical performance data
- Manager-defined KPI targets

### Outputs

- Daily priority briefings per recruiter (email or Slack)
- Pipeline conversion analysis with stage-by-stage bottleneck identification
- Weekly activity reports per recruiter and per team
- Workflow optimization suggestions based on data patterns
- Manager coaching dashboard with early warning flags
- Top performer analysis (what behaviors correlate with highest placement rates)

### Primary ATS Integration

- **Read:** All recruiter activity records, pipeline stage data, placement data, job order data
- **Write:** Daily briefing activity logs, performance metric snapshots
- **Endpoints:** Bullhorn `/entity/Note`, `/entity/Sendout`, `/entity/Placement`, activity search queries

### Claude Model Routing

| Task | Model | Rationale |
|------|-------|-----------|
| Activity data aggregation and calculation | Haiku 4.5 | Structured data processing |
| Daily briefing generation | Haiku 4.5 | Template-based personalization with recruiter data |
| Bottleneck analysis and root cause identification | Sonnet 4.6 | Requires reasoning about pipeline dynamics |
| Workflow optimization recommendations | Sonnet 4.6 | Strategic reasoning about process improvement |
| Coaching insight generation | Sonnet 4.6 | Sensitive communication requiring nuanced framing |

### Cost Estimate

- Small agency (5-10 recruiters): ~$10-20/month
- Mid agency (20-50 recruiters): ~$25-50/month
- Large agency (50-200 recruiters): ~$50-100/month

### Tier Availability

Enterprise tier.

---

## Agent 9: Post-Placement Follow-Up Agent

### Purpose

Execute automated check-in sequences with placed candidates and their hiring managers at critical milestones (Day 1, 7, 30, 60, 90), detect early warning signs, and trigger redeployment pipelines for ending assignments.

### The Problem It Solves

The placement is not the end of the relationship -- it is the beginning of the most critical period. Candidate falloff (early terminations, no-shows after Day 1, dissatisfaction leading to early resignation) costs the agency the placement fee, damages the client relationship, and wastes the sourcing and screening investment.

Most agencies do not have a systematic follow-up process. A recruiter might check in on Day 1 and then forget. The 30-day check-in gets missed because new reqs came in. The 90-day assignment-ending touchpoint never happens, and the candidate takes a competing assignment instead of being redeployed by the same agency.

For temp and contract staffing, the redeployment pipeline is a revenue engine. A candidate finishing a 13-week assignment should be pre-matched to the next assignment before the current one ends. Without systematic follow-up, that candidate drifts to a competitor.

### How It Works

1. **Placement triggers the sequence.** When a candidate's status changes to "Placed" in the ATS, the agent initiates a configurable check-in sequence.
2. **Automated check-ins at milestones.** Day 1: "How was your first day?" Day 7: "How is the first week going?" Day 30: detailed satisfaction survey. Day 60: mid-assignment review. Day 90: redeployment conversation.
3. **Sentiment analysis.** Claude Haiku analyzes responses for sentiment signals. Positive responses get a thank-you. Neutral responses get a follow-up question. Negative responses or non-responses trigger escalation to the recruiter.
4. **Early warning detection.** Patterns that indicate risk: missed response to Day 1 check-in, negative language in feedback, decreased communication from the hiring manager, changes in candidate's LinkedIn activity (updating profile, connecting with recruiters).
5. **Redeployment pipeline.** For temp/contract placements, the agent begins matching the candidate to upcoming openings 30 days before assignment end. Generates a redeployment recommendation with matching roles and candidate interest.

### Inputs

- Placement records from ATS (candidate, client, start date, end date, role, rate)
- Candidate communication data (email, SMS responses)
- Hiring manager feedback (if collected)
- Upcoming job orders matching the candidate's profile
- Historical redeployment success data

### Outputs

- Automated check-in messages at Day 1, 7, 30, 60, 90 (email and/or SMS)
- Sentiment analysis reports per placement
- Early warning alerts for at-risk placements
- Redeployment recommendations with matching roles (30 days before assignment end)
- Placement retention metrics: Day 30 retention rate, Day 90 retention rate, redeployment rate
- Hiring manager satisfaction tracking

### Primary ATS Integration

- **Read:** Placement records, candidate profiles, job order data for redeployment matching
- **Write:** Follow-up activity logs, sentiment scores, redeployment candidate records
- **Endpoints:** Bullhorn `/entity/Placement`, `/entity/Candidate`, `/entity/JobOrder`; email/SMS delivery via Twilio/SendGrid

### Claude Model Routing

| Task | Model | Rationale |
|------|-------|-----------|
| Check-in message generation | Haiku 4.5 | Template-based with light personalization |
| Response sentiment analysis | Haiku 4.5 | Binary/categorical classification |
| Early warning pattern detection | Sonnet 4.6 | Complex pattern recognition across multiple signals |
| Redeployment matching and recommendation | Sonnet 4.6 | Candidate-to-job matching with nuanced fit assessment |

### Cost Estimate

- Small agency (20 active placements): ~$5-10/month
- Mid agency (100 active placements): ~$15-30/month
- Large agency (500+ active placements): ~$40-80/month

### Tier Availability

Professional and Enterprise tiers.

---

## Agent 10: Analytics and Reporting Agent

### Purpose

Centralize data from all agents and the ATS into unified dashboards, track KPIs in real-time, and generate custom reports on demand for agency leadership, clients, and stakeholders.

### The Problem It Solves

Data in a staffing agency is siloed across 5-10 tools: ATS, email, job boards, spreadsheets, accounting software, and now AI agent outputs. Agency owners cannot get a single view of their business without manually pulling reports from each system and combining them in a spreadsheet. By the time the monthly report is compiled, it is already stale.

Recruiters do not know their own performance metrics in real-time. Managers cannot compare team performance without building ad-hoc reports. Clients ask for fill rate data and it takes a day to compile. Board meetings require weeks of report preparation.

The lack of real-time analytics means decisions are made on gut feel rather than data. The agency that knows its time-to-fill dropped 20% this month can double down on what is working. The agency that discovers the same thing 6 weeks later has already missed the window.

### How It Works

1. **Data aggregation.** The agent pulls data from all 9 other agents plus the ATS into a unified data model. Candidate pipeline metrics, placement metrics, client health scores, compliance status, market data, recruiter activity -- all in one place.
2. **Dashboard generation.** Pre-built dashboard templates for three audiences: recruiter (daily view), manager (weekly/monthly view), and executive (strategic view). Dashboards update in real-time as data changes.
3. **KPI tracking.** Core staffing KPIs tracked automatically: time-to-fill, fill rate, gross margin, revenue per recruiter, submittals-to-placement ratio, candidate satisfaction, client NPS, compliance rate, cost-per-hire, recruiter utilization.
4. **Custom report generation.** On-demand reports via natural language request. "Generate a report showing fill rate by client for Q1 2026." "Show me recruiter productivity trends over the last 6 months." The agent interprets the request, queries the data, and produces a formatted report.
5. **Trend analysis and recommendations.** Beyond reporting, the agent identifies trends and generates actionable recommendations. "Time-to-fill for IT roles has increased 25% over the last quarter. Contributing factors: reduced candidate pipeline from LinkedIn (down 30%), increased client requirements specificity. Recommendation: expand sourcing to GitHub and Stack Overflow for technical roles."

### Inputs

- Data feeds from all 9 other agents (scores, metrics, activity logs)
- ATS data (placements, revenue, pipeline stages, activity records)
- Financial data (invoices, margins, revenue) if accessible via API
- Agency-defined KPI targets and benchmarks
- Custom report requests via natural language

### Outputs

- Real-time dashboards: recruiter daily, manager weekly, executive strategic
- Core KPI tracking with trend visualization
- Custom reports generated on demand (PDF, CSV, or in-dashboard)
- Automated weekly/monthly reports delivered via email
- Trend analysis with actionable recommendations
- Revenue forecasting based on current pipeline
- ROI tracking for the AI system itself (showing value delivered)

### Primary ATS Integration

- **Read:** All entity types for comprehensive analytics (candidates, jobs, placements, clients, activities, notes, revenue)
- **Write:** Dashboard links and report references in ATS notes/custom fields
- **Endpoints:** Bullhorn reporting API, `/search` endpoints across all entities; Lever Analytics API; Greenhouse reporting endpoints

### Claude Model Routing

| Task | Model | Rationale |
|------|-------|-----------|
| Data aggregation and calculation | Haiku 4.5 | Structured data processing and arithmetic |
| Dashboard data formatting | Haiku 4.5 | Template application to structured data |
| Natural language report request interpretation | Sonnet 4.6 | Understanding complex, ambiguous report requests |
| Trend analysis and recommendation generation | Sonnet 4.6 | Strategic reasoning about data patterns |
| Executive summary writing | Sonnet 4.6 | High-quality analytical prose for leadership audience |
| Batch overnight analytics processing | Batch API (Haiku) | 50% cost reduction for non-urgent aggregation |

### Cost Estimate

- Small agency: ~$15-25/month (primarily Haiku for data processing)
- Mid agency: ~$30-60/month
- Large agency: ~$60-120/month
- Note: dashboard hosting/visualization tooling may add $20-50/month depending on agency preference (Grafana, Metabase, or custom HTML dashboards)

### Tier Availability

Enterprise tier for full analytics suite. Professional tier includes basic reporting (weekly automated reports, core KPI tracking, pre-built dashboards).

---

## Agent Interconnection Summary

The 10 agents are not isolated systems. They form a collaborative intelligence network where each agent's output becomes another agent's input:

```
Resume Screening ──────> Candidate Sourcing
       |                        |
       | scored candidates      | new candidates
       v                        v
Interview Scheduling <── Recruiter Productivity
       |                        |
       | scheduled interviews   | daily priorities
       v                        v
Post-Placement <──────── Compliance Monitoring
       |                        |
       | placement health       | compliance status
       v                        v
Client Health <────────── Analytics & Reporting
       |                        ^
       | client scores          | all agent data
       v                        |
Client Outreach <──────── Market Intelligence
       |                        |
       | new opportunities      | market data
       └────────────────────────┘
```

**Key data flows:**
- When Resume Screening scores candidates, those scores feed into Recruiter Productivity for daily briefings
- When Compliance Monitoring detects a gap, Post-Placement adjusts its follow-up sequences
- When Client Health flags a churn risk, Client Outreach pauses prospecting that client and Analytics generates a retention report
- When Market Intelligence detects a demand spike, Candidate Sourcing adjusts its search priorities and Client Outreach targets companies in that market

This interconnection is what makes the Software Factory more than the sum of its parts. A standalone resume screener is useful. Ten agents sharing context across the entire placement lifecycle is transformative.

---

## Cost Summary by Tier

| Agent | Monthly API Cost | Starter | Professional | Enterprise |
|-------|-----------------|---------|--------------|------------|
| Resume Screening | $2-40 | Included | Included | Included |
| Candidate Sourcing | $15-200 | Included | Included | Included |
| Client Outreach (AI SDR) | $80-250 | Add-on | Add-on | Included |
| Interview Scheduling | $3-30 | -- | Included | Included |
| Compliance Monitoring | $10-100 | Add-on | Included | Included |
| Market Intelligence | $130-260 | -- | Add-on | Included |
| Client Health | $15-100 | -- | -- | Included |
| Recruiter Productivity | $10-100 | -- | -- | Included |
| Post-Placement | $5-80 | -- | Included | Included |
| Analytics & Reporting | $15-120 | -- | Basic | Included |
| **Total estimated range** | | **$17-240** | **$50-470** | **$285-1,280** |

Note: Ranges reflect agency size (small to large). Actual costs depend on volume of candidates, placements, clients, and recruiters. The $300/month subscription covers coaching, community, and system updates -- API costs for Claude and external services are borne by the agency as a pass-through operating expense.
