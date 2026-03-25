# Staffing & Recruiting AI Automation Suite — Complete Solutions Architecture

> **Open-Source Repository: `staffing-ai-automation`**
> Designed by [Using AI to Scale](https://usingaitoscale.com) — Launching March 25, 2026
> License: MIT

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [GitHub Repository Structure](#2-github-repository-structure)
3. [System Architecture Overview](#3-system-architecture-overview)
4. [Agent Architectures (10 Agents)](#4-agent-architectures)
   - 4.1 Resume Screening Agent
   - 4.2 Candidate Sourcing Agent
   - 4.3 Client Outreach / Cold Email Agent
   - 4.4 Interview Scheduling Agent
   - 4.5 Compliance Monitoring Agent
   - 4.6 Market Intelligence Agent
   - 4.7 Client Health / Retention Agent
   - 4.8 Recruiter Productivity Agent
   - 4.9 Post-Placement Follow-up Agent
   - 4.10 Analytics & Reporting Agent
5. [n8n Workflow Designs](#5-n8n-workflow-designs)
6. [ATS Integration Points](#6-ats-integration-points)
7. [Data Model & Shared Schema](#7-data-model--shared-schema)
8. [Deployment Guide](#8-deployment-guide)
9. [Cost Analysis](#9-cost-analysis)
10. [Security & Compliance](#10-security--compliance)
11. [Roadmap](#11-roadmap)

---

## 1. Executive Summary

Staffing and recruiting agencies operate in an industry where margins are thin (typically 15–25% for contract placements), speed determines revenue, and manual processes consume 60–70% of a recruiter's day. The average time-to-fill has climbed to 44 days, while candidate ghosting rates exceed 40% and client churn hovers around 20–30% annually.

This open-source suite provides **10 AI-powered agents** orchestrated through **n8n workflows** that automate the highest-impact, most time-consuming functions in staffing. Built on Claude (Anthropic's API), these agents integrate with every major ATS (Bullhorn, JobAdder, Lever, Greenhouse) and the broader staffing tech stack.

### What This Solves

| Problem | Current Pain | Our Solution | Expected Impact |
|---------|-------------|--------------|-----------------|
| Resume screening bottleneck | 23 seconds per resume, 75% are unqualified | AI screening + scoring in <2 seconds | 90% reduction in screening time |
| Slow candidate sourcing | Manual LinkedIn/job board searching | Multi-source automated sourcing | 3x more qualified candidates surfaced |
| Client development | Cold outreach is manual, inconsistent | AI-personalized multi-channel sequences | 2–3x improvement in reply rates |
| Interview scheduling | 5–8 emails per interview to coordinate | Automated availability matching | Scheduling in <5 minutes vs. 2 days |
| Compliance gaps | Manual tracking of I-9, certs, licenses | Proactive monitoring with expiration alerts | Zero compliance lapses |
| No market intelligence | Reactive to market changes | Real-time salary, demand, competitor tracking | Data-driven pricing and positioning |
| Client churn | No early warning system | Health scoring with intervention triggers | 25–40% reduction in churn |
| Recruiter inefficiency | 60%+ time on admin tasks | AI handles admin, humans handle relationships | 2x recruiter capacity |
| Post-placement falloff | No systematic follow-up | Automated check-in sequences | 30% improvement in retention/redeployment |
| No unified analytics | Data siloed across 5–10 tools | Centralized dashboards and reporting | Real-time visibility into all KPIs |

### Architecture Principles

1. **ATS-Agnostic** — Works with any ATS via adapter pattern; ships with Bullhorn, Lever, Greenhouse, JobAdder adapters
2. **Cost-Optimized** — Uses Claude Haiku for high-volume tasks, Sonnet for complex reasoning, Batch API where latency isn't critical
3. **Self-Hosted First** — n8n self-hosted as the workflow engine; no vendor lock-in
4. **Modular** — Deploy one agent or all ten; each works independently
5. **Privacy-Conscious** — Candidate PII stays in your infrastructure; Claude API calls use ephemeral processing

---

## 2. GitHub Repository Structure

```
staffing-ai-automation/
│
├── README.md                          # Project overview, quick start, badges
├── LICENSE                            # MIT License
├── CONTRIBUTING.md                    # Contribution guidelines
├── CHANGELOG.md                       # Version history
├── .env.example                       # Environment variable template
├── docker-compose.yml                 # One-command deployment
├── Makefile                           # Common commands (setup, test, deploy)
│
├── docs/                              # Documentation
│   ├── architecture-overview.md       # This document (system design)
│   ├── getting-started.md             # Step-by-step setup guide
│   ├── agent-deep-dives/             # Detailed docs per agent
│   │   ├── resume-screening.md
│   │   ├── candidate-sourcing.md
│   │   ├── client-outreach.md
│   │   ├── interview-scheduling.md
│   │   ├── compliance-monitoring.md
│   │   ├── market-intelligence.md
│   │   ├── client-health.md
│   │   ├── recruiter-productivity.md
│   │   ├── post-placement.md
│   │   └── analytics-reporting.md
│   ├── ats-integration-guide.md       # ATS connection setup
│   ├── n8n-workflow-guide.md          # Workflow import/customization
│   ├── cost-optimization.md           # Keeping API costs low
│   ├── security-compliance.md         # Data handling, GDPR, SOC2
│   └── troubleshooting.md            # Common issues and fixes
│
├── agents/                            # AI Agent definitions
│   ├── shared/                        # Shared utilities across agents
│   │   ├── prompts/                   # Reusable prompt components
│   │   │   ├── system-context.md      # Base staffing industry context
│   │   │   ├── output-formats.md      # Standard JSON output schemas
│   │   │   └── safety-guidelines.md   # PII handling, bias prevention
│   │   ├── schemas/                   # JSON schemas for validation
│   │   │   ├── candidate.schema.json
│   │   │   ├── job-order.schema.json
│   │   │   ├── placement.schema.json
│   │   │   ├── client.schema.json
│   │   │   └── activity.schema.json
│   │   └── utils/                     # Shared helper functions
│   │       ├── ats-adapter.js         # ATS abstraction layer
│   │       ├── claude-client.js       # Claude API wrapper with retry/batching
│   │       ├── cost-tracker.js        # Token usage and cost monitoring
│   │       └── pii-handler.js         # PII detection and redaction
│   │
│   ├── resume-screening/
│   │   ├── README.md
│   │   ├── prompts/
│   │   │   ├── screening-system.md    # System prompt for resume analysis
│   │   │   ├── scoring-rubric.md      # Configurable scoring criteria
│   │   │   └── bias-check.md          # Bias detection prompt
│   │   ├── config.json                # Agent configuration
│   │   └── tests/                     # Test resumes and expected outputs
│   │
│   ├── candidate-sourcing/
│   │   ├── README.md
│   │   ├── prompts/
│   │   │   ├── search-query-builder.md
│   │   │   ├── profile-evaluator.md
│   │   │   └── outreach-composer.md
│   │   ├── config.json
│   │   └── tests/
│   │
│   ├── client-outreach/
│   │   ├── README.md
│   │   ├── prompts/
│   │   │   ├── company-research.md
│   │   │   ├── email-personalization.md
│   │   │   ├── sequence-builder.md
│   │   │   └── reply-classifier.md
│   │   ├── config.json
│   │   ├── templates/                 # Email sequence templates
│   │   │   ├── new-client-intro.json
│   │   │   ├── reactivation.json
│   │   │   └── vertical-specific/
│   │   └── tests/
│   │
│   ├── interview-scheduling/
│   │   ├── README.md
│   │   ├── prompts/
│   │   │   ├── availability-parser.md
│   │   │   └── confirmation-composer.md
│   │   ├── config.json
│   │   └── tests/
│   │
│   ├── compliance-monitoring/
│   │   ├── README.md
│   │   ├── prompts/
│   │   │   ├── document-classifier.md
│   │   │   ├── expiration-tracker.md
│   │   │   └── regulation-checker.md
│   │   ├── config.json
│   │   ├── rules/                     # Compliance rule definitions
│   │   │   ├── federal.json
│   │   │   ├── state/                 # State-specific rules
│   │   │   └── industry/             # Industry-specific (healthcare, IT, etc.)
│   │   └── tests/
│   │
│   ├── market-intelligence/
│   │   ├── README.md
│   │   ├── prompts/
│   │   │   ├── salary-analyzer.md
│   │   │   ├── demand-forecaster.md
│   │   │   └── competitor-tracker.md
│   │   ├── config.json
│   │   └── tests/
│   │
│   ├── client-health/
│   │   ├── README.md
│   │   ├── prompts/
│   │   │   ├── health-scorer.md
│   │   │   ├── churn-predictor.md
│   │   │   └── intervention-recommender.md
│   │   ├── config.json
│   │   └── tests/
│   │
│   ├── recruiter-productivity/
│   │   ├── README.md
│   │   ├── prompts/
│   │   │   ├── daily-briefing.md
│   │   │   ├── task-prioritizer.md
│   │   │   └── activity-summarizer.md
│   │   ├── config.json
│   │   └── tests/
│   │
│   ├── post-placement/
│   │   ├── README.md
│   │   ├── prompts/
│   │   │   ├── check-in-composer.md
│   │   │   ├── satisfaction-analyzer.md
│   │   │   └── redeployment-matcher.md
│   │   ├── config.json
│   │   ├── templates/                 # Check-in sequence templates
│   │   └── tests/
│   │
│   └── analytics-reporting/
│       ├── README.md
│       ├── prompts/
│       │   ├── kpi-narrator.md
│       │   ├── trend-analyzer.md
│       │   └── recommendation-engine.md
│       ├── config.json
│       ├── dashboards/               # Dashboard templates
│       └── tests/
│
├── workflows/                         # n8n workflow JSON exports
│   ├── core/                         # Essential workflows
│   │   ├── resume-screening.json
│   │   ├── candidate-sourcing.json
│   │   ├── client-outreach.json
│   │   ├── interview-scheduling.json
│   │   └── compliance-monitoring.json
│   ├── advanced/                     # Advanced workflows
│   │   ├── market-intelligence.json
│   │   ├── client-health-scoring.json
│   │   ├── recruiter-daily-briefing.json
│   │   ├── post-placement-sequence.json
│   │   └── analytics-pipeline.json
│   ├── integrations/                 # ATS-specific connector workflows
│   │   ├── bullhorn-sync.json
│   │   ├── lever-sync.json
│   │   ├── greenhouse-sync.json
│   │   └── jobadder-sync.json
│   └── utilities/                    # Helper workflows
│       ├── ats-health-check.json
│       ├── cost-monitor.json
│       └── data-backup.json
│
├── integrations/                      # ATS adapter implementations
│   ├── base-adapter.js               # Abstract base class
│   ├── bullhorn/
│   │   ├── adapter.js                # Bullhorn-specific implementation
│   │   ├── auth.js                   # OAuth 2.0 + session token handler
│   │   ├── mappings.json             # Field mappings to shared schema
│   │   └── README.md
│   ├── lever/
│   │   ├── adapter.js
│   │   ├── auth.js                   # API key auth
│   │   ├── mappings.json
│   │   └── README.md
│   ├── greenhouse/
│   │   ├── adapter.js
│   │   ├── auth.js                   # Basic auth (token:)
│   │   ├── mappings.json
│   │   └── README.md
│   ├── jobadder/
│   │   ├── adapter.js
│   │   ├── auth.js                   # OAuth 2.0
│   │   ├── mappings.json
│   │   └── README.md
│   └── generic-webhook/
│       ├── adapter.js                # For any ATS with webhook support
│       └── README.md
│
├── templates/                         # Ready-to-use templates
│   ├── email-sequences/             # Cold outreach sequence templates
│   │   ├── staffing-intro.json
│   │   ├── it-staffing.json
│   │   ├── healthcare-staffing.json
│   │   ├── light-industrial.json
│   │   └── executive-search.json
│   ├── scoring-rubrics/             # Resume scoring configurations
│   │   ├── general.json
│   │   ├── technical.json
│   │   ├── healthcare.json
│   │   └── executive.json
│   ├── compliance-rules/            # Compliance rule templates
│   │   ├── i9-checklist.json
│   │   ├── background-check.json
│   │   └── state-requirements/
│   └── dashboards/                  # Analytics dashboard templates
│       ├── recruiter-daily.json
│       ├── agency-weekly.json
│       └── client-monthly.json
│
├── scripts/                           # Setup and utility scripts
│   ├── setup.sh                      # Initial environment setup
│   ├── import-workflows.sh           # Import n8n workflows
│   ├── seed-data.sh                  # Seed test data
│   ├── health-check.sh              # Verify all services running
│   ├── backup.sh                    # Database backup utility
│   └── migrate.sh                   # Schema migration tool
│
└── tests/                            # Test suite
    ├── unit/                        # Unit tests per agent
    ├── integration/                 # Integration tests (ATS connections)
    ├── e2e/                         # End-to-end workflow tests
    └── fixtures/                    # Test data
        ├── sample-resumes/
        ├── sample-jobs/
        └── sample-candidates/
```

### README.md Structure

The README should include:

```markdown
# 🏢 Staffing AI Automation Suite

> 10 AI agents that automate 80% of staffing agency operations.
> Built with Claude (Anthropic), n8n, and open-source tools.

[![License: MIT](badge)](LICENSE)
[![n8n](badge)](https://n8n.io)
[![Claude API](badge)](https://docs.anthropic.com)

## What This Does

[One-paragraph summary + architecture diagram]

## Quick Start (5 minutes)

1. Clone the repo
2. Copy `.env.example` to `.env` and add your API keys
3. Run `docker-compose up -d`
4. Import workflows: `make import-workflows`
5. Open n8n at `http://localhost:5678`

## The 10 Agents

| Agent | What It Does | Saves |
|-------|-------------|-------|
| Resume Screening | Scores and ranks candidates against job requirements | 15+ hrs/week |
| Candidate Sourcing | Finds candidates across LinkedIn, job boards, your ATS | 10+ hrs/week |
| Client Outreach | Personalized cold email sequences to target companies | 8+ hrs/week |
| Interview Scheduling | Auto-coordinates availability between all parties | 5+ hrs/week |
| Compliance Monitoring | Tracks I-9, certifications, licenses, expiration dates | 3+ hrs/week |
| Market Intelligence | Real-time salary data, demand trends, competitor analysis | Strategic advantage |
| Client Health | Predicts churn risk and triggers retention interventions | 25-40% less churn |
| Recruiter Productivity | Daily briefings, task prioritization, admin automation | 2x capacity |
| Post-Placement | Automated check-in sequences for placed candidates | 30% better retention |
| Analytics & Reporting | Unified dashboards across all agency KPIs | Real-time visibility |

## Supported ATS Systems

Bullhorn • Lever • Greenhouse • JobAdder • Any REST API (via generic adapter)

## Cost to Run

$450–1,000/month for a mid-size agency (10–50 placements/month).
See [Cost Analysis](docs/cost-optimization.md) for detailed breakdown.

## Documentation

- [Architecture Overview](docs/architecture-overview.md)
- [Getting Started Guide](docs/getting-started.md)
- [ATS Integration Guide](docs/ats-integration-guide.md)
- [Agent Deep Dives](docs/agent-deep-dives/)
```

---

## 3. System Architecture Overview

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        STAFFING AI AUTOMATION SUITE                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                     TRIGGER LAYER                            │   │
│  │  Webhooks │ Cron Schedules │ ATS Events │ Email │ Manual     │   │
│  └────────────────────────┬─────────────────────────────────────┘   │
│                           │                                         │
│  ┌────────────────────────▼─────────────────────────────────────┐   │
│  │                   n8n WORKFLOW ENGINE                         │   │
│  │                                                               │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐           │   │
│  │  │ Resume  │ │ Source  │ │ Outreach│ │Schedule │           │   │
│  │  │ Screen  │ │ Cand.  │ │ Client  │ │Interview│           │   │
│  │  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘           │   │
│  │  ┌────┴────┐ ┌────┴────┐ ┌────┴────┐ ┌────┴────┐           │   │
│  │  │Complianc│ │ Market │ │ Client │ │Recruiter│           │   │
│  │  │ Monitor │ │ Intel  │ │ Health │ │Productiv│           │   │
│  │  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘           │   │
│  │  ┌────┴────┐ ┌────┴────┐                                    │   │
│  │  │  Post   │ │Analytics│                                    │   │
│  │  │Placement│ │Reporting│                                    │   │
│  │  └─────────┘ └─────────┘                                    │   │
│  └────────────────────────┬─────────────────────────────────────┘   │
│                           │                                         │
│  ┌────────────────────────▼─────────────────────────────────────┐   │
│  │                    AI PROCESSING LAYER                        │   │
│  │                                                               │   │
│  │  Claude Haiku 4.5          Claude Sonnet 4.6                 │   │
│  │  ├─ Resume parsing         ├─ Complex scoring                │   │
│  │  ├─ Email classification   ├─ Market analysis                │   │
│  │  ├─ Data extraction        ├─ Churn prediction               │   │
│  │  └─ Template filling       ├─ Strategy recommendations       │   │
│  │                            └─ Personalized outreach          │   │
│  │                                                               │   │
│  │  Batch API (non-urgent)    Prompt Caching (repeated schemas) │   │
│  └────────────────────────┬─────────────────────────────────────┘   │
│                           │                                         │
│  ┌────────────────────────▼─────────────────────────────────────┐   │
│  │                  INTEGRATION LAYER                            │   │
│  │                                                               │   │
│  │  ATS Adapters    Email         Calendar      Job Boards      │   │
│  │  ┌──────────┐   ┌──────────┐  ┌──────────┐  ┌──────────┐   │   │
│  │  │ Bullhorn │   │ Gmail    │  │ Google   │  │ LinkedIn │   │   │
│  │  │ Lever    │   │ Outlook  │  │ Outlook  │  │ Indeed   │   │   │
│  │  │Greenhouse│   │ Instantly│  │ Calendly │  │ ZipRecr. │   │   │
│  │  │ JobAdder │   │ Smartlead│  │ Cal.com  │  │ Monster  │   │   │
│  │  └──────────┘   └──────────┘  └──────────┘  └──────────┘   │   │
│  │                                                               │   │
│  │  Background Checks    Payroll         Communication          │   │
│  │  ┌──────────┐        ┌──────────┐    ┌──────────┐           │   │
│  │  │ Checkr   │        │ ADP      │    │ Slack    │           │   │
│  │  │ Sterling │        │ Gusto    │    │ Teams    │           │   │
│  │  │ GoodHire │        │ Paychex  │    │ Twilio   │           │   │
│  │  └──────────┘        └──────────┘    └──────────┘           │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    DATA LAYER                                │   │
│  │                                                               │   │
│  │  PostgreSQL (primary)  │  Redis (cache/queue)  │  S3 (docs)  │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Agent Interconnection Map

The 10 agents are not isolated — they form a collaborative intelligence network:

```
Resume Screening ──────► Candidate Sourcing
       │                        │
       │ scored candidates      │ new candidates
       ▼                        ▼
Interview Scheduling ◄── Recruiter Productivity
       │                        │
       │ scheduled interviews   │ daily priorities
       ▼                        ▼
Post-Placement ◄──────── Compliance Monitoring
       │                        │
       │ placement health       │ compliance status
       ▼                        ▼
Client Health ◄────────── Analytics & Reporting
       │                        ▲
       │ client scores          │ all agent data
       ▼                        │
Client Outreach ◄──────── Market Intelligence
       │                        │
       │ new opportunities      │ market data
       └────────────────────────┘
```

### Data Flow Patterns

**Pattern 1: New Job Order Flow**
1. New job order created in ATS → triggers Candidate Sourcing Agent
2. Sourcing Agent searches ATS database + external sources → surfaces candidates
3. Resume Screening Agent scores each candidate against job requirements
4. Top candidates pushed to Interview Scheduling Agent
5. Recruiter Productivity Agent adds to recruiter's daily briefing

**Pattern 2: Placement Lifecycle Flow**
1. Candidate placed → Post-Placement Agent initiates check-in sequence
2. Compliance Monitoring Agent verifies all documents current
3. Client Health Agent updates client satisfaction score
4. Analytics Agent captures time-to-fill, margin, source effectiveness
5. Market Intelligence Agent updates salary/demand data

**Pattern 3: Business Development Flow**
1. Market Intelligence Agent identifies hot market (e.g., surge in AI engineer demand)
2. Client Outreach Agent generates personalized sequences for target companies
3. Analytics Agent tracks outreach performance
4. Client Health Agent monitors new client engagement

---

## 4. Agent Architectures

---

### 4.1 Resume Screening Agent

**Purpose:** Automatically parse, score, and rank resumes/CVs against job requirements, eliminating the manual screening bottleneck.

#### Inputs
- Resume file (PDF, DOCX, TXT) — from ATS upload, email attachment, or job board application
- Job order requirements (title, skills, experience, location, certifications, salary range)
- Custom scoring rubric (configurable per job type or client)
- Historical placement data (optional — for learning what "good" looks like for a specific client)

#### Outputs
```json
{
  "candidate_id": "string",
  "overall_score": 85,
  "recommendation": "STRONG_MATCH | GOOD_MATCH | PARTIAL_MATCH | NO_MATCH",
  "scoring_breakdown": {
    "skills_match": 90,
    "experience_match": 80,
    "education_match": 85,
    "location_match": 100,
    "certification_match": 75,
    "culture_fit_indicators": 70
  },
  "extracted_data": {
    "name": "string",
    "email": "string",
    "phone": "string",
    "current_title": "string",
    "years_experience": 7,
    "skills": ["Python", "AWS", "Docker"],
    "certifications": ["AWS Solutions Architect"],
    "education": [{"degree": "BS", "field": "Computer Science", "institution": "MIT"}],
    "work_history": [{"company": "string", "title": "string", "duration": "string", "highlights": []}]
  },
  "flags": {
    "job_hopper": false,
    "career_gap": true,
    "overqualified": false,
    "relocation_needed": true
  },
  "ai_summary": "7-year backend engineer with strong AWS experience. Exact match on 4/5 required skills. Gap in 2023 (parental leave noted). Currently in Austin, TX — role requires Chicago relocation. Strong candidate if relocation works.",
  "bias_check": "PASS",
  "tokens_used": 1847,
  "cost_usd": 0.003
}
```

#### Claude Prompt Architecture

**System Prompt (screening-system.md):**
```markdown
You are an expert staffing recruiter AI assistant. Your role is to objectively
evaluate resumes against job requirements and provide structured scoring.

CORE PRINCIPLES:
- Score based ONLY on qualifications, skills, and experience relevance
- Never factor in name, gender, age, ethnicity, or any protected characteristic
- Flag potential concerns (gaps, job hopping) factually without negative bias
- Career gaps are neutral — note them but do not penalize
- Equivalent experience can substitute for formal education
- Consider transferable skills from adjacent industries

SCORING METHODOLOGY:
- Skills Match (0-100): Direct match % of required + preferred skills
- Experience Match (0-100): Years and relevance of experience vs. requirements
- Education Match (0-100): Degree/certification alignment (consider equivalents)
- Location Match (0-100): Geographic fit including remote eligibility
- Certification Match (0-100): Required and preferred certifications held

OUTPUT: Always respond in the exact JSON schema provided. No additional text.
```

**Scoring Rubric Prompt (scoring-rubric.md):**
```markdown
## Scoring Rubric Configuration

This rubric is injected per-job-order. Default weights:

| Category | Weight | Scoring Guide |
|----------|--------|--------------|
| Skills Match | 35% | 100 = all required + preferred; 80 = all required; 60 = most required; <40 = missing critical skills |
| Experience Match | 25% | 100 = exceeds by 1-2 years; 80 = meets exactly; 60 = 1 year under with strong relevance; <40 = significantly under |
| Education Match | 15% | 100 = exact degree match; 80 = related field; 60 = unrelated degree + relevant experience; 40 = no degree, strong experience |
| Location Match | 15% | 100 = local or remote-OK; 80 = willing to relocate; 60 = partial remote possible; 0 = no match, no flexibility |
| Certification Match | 10% | 100 = all required + preferred; 80 = all required; 50 = most required; 0 = missing required certs |

THRESHOLDS:
- STRONG_MATCH: Overall ≥ 80
- GOOD_MATCH: Overall 65-79
- PARTIAL_MATCH: Overall 50-64
- NO_MATCH: Overall < 50
```

#### n8n Workflow Design

```
[Trigger: Webhook / ATS Event]
    → [Fetch Job Requirements from ATS]
    → [Extract Resume Text (PDF/DOCX parser)]
    → [IF resume text > 500 chars → proceed; ELSE → flag for manual review]
    → [Claude API Call: Haiku — Extract structured data from resume]
    → [Claude API Call: Haiku — Score against job requirements using rubric]
    → [Claude API Call: Haiku — Bias check (separate call for independence)]
    → [IF score ≥ threshold → Push to ATS as "Screened - Qualified"]
    → [IF score < threshold → Push to ATS as "Screened - Not Qualified"]
    → [Update candidate record with score + summary]
    → [IF STRONG_MATCH → Notify recruiter via Slack/email]
    → [Log to analytics database]
```

#### Tools & API Integrations
- **Claude API** (Haiku 4.5 for parsing, Haiku for scoring) — ~$0.002–0.005 per resume
- **PDF parsing**: `pdf-parse` npm package or Apache Tika (self-hosted)
- **DOCX parsing**: `mammoth` npm package
- **ATS API**: Push scores/summaries back to candidate record
- **Slack/Email**: Notify recruiter of strong matches

#### Estimated Cost
- Per resume: $0.003–0.008 (3 Claude Haiku calls)
- 500 resumes/month: $1.50–$4.00/month
- With Batch API (non-urgent): 50% discount → $0.75–$2.00/month

#### Inter-Agent Connections
- **Receives from**: Candidate Sourcing Agent (newly found candidates), ATS webhook (new applications)
- **Sends to**: Interview Scheduling Agent (qualified candidates), Recruiter Productivity Agent (daily candidate summary), Analytics Agent (screening metrics)

---

### 4.2 Candidate Sourcing Agent

**Purpose:** Proactively find candidates across multiple sources (ATS database, LinkedIn, job boards, referral networks) when a new job order is created or an existing pipeline runs thin.

#### Inputs
- Job order requirements (same schema as Resume Screening)
- Source configuration (which sources to search: ATS DB, LinkedIn, Indeed, etc.)
- Existing pipeline state (how many candidates already in pipeline, what stages)
- Past successful placements for similar roles (optional — for pattern matching)

#### Outputs
```json
{
  "job_order_id": "string",
  "sourcing_session_id": "string",
  "candidates_found": 47,
  "candidates_after_dedup": 32,
  "candidates_new_to_ats": 18,
  "sources_searched": [
    {"source": "ats_database", "found": 12, "qualified": 8},
    {"source": "linkedin", "found": 20, "qualified": 15},
    {"source": "indeed", "found": 15, "qualified": 9}
  ],
  "top_candidates": [
    {
      "name": "string",
      "source": "linkedin",
      "profile_url": "string",
      "relevance_score": 88,
      "key_matches": ["5 years React", "fintech experience", "San Francisco"],
      "suggested_outreach": "string (personalized message draft)"
    }
  ],
  "search_queries_used": ["string (boolean search strings generated)"],
  "pipeline_status": {
    "total_in_pipeline": 32,
    "screened": 18,
    "submitted_to_client": 5,
    "interviewing": 2,
    "gap_analysis": "Need 3 more senior candidates with fintech background"
  }
}
```

#### Claude Prompt Architecture

**Search Query Builder (search-query-builder.md):**
```markdown
You are an expert technical recruiter who builds precise Boolean search strings.

Given a job description, generate:
1. A LinkedIn Recruiter search string (Boolean operators, filters)
2. An Indeed/job board search string
3. An internal ATS keyword search
4. Alternative title variations to capture edge cases

RULES:
- Include synonyms and title variations (e.g., "Software Engineer" OR "Software Developer" OR "SDE")
- Use NOT operators to exclude irrelevant results (e.g., NOT intern NOT junior for senior roles)
- Consider adjacent skills that indicate capability (e.g., someone with Kubernetes likely knows Docker)
- Generate location variants (city + surrounding areas, remote indicators)

OUTPUT: JSON array of search objects with source, query_string, and estimated_reach.
```

**Profile Evaluator (profile-evaluator.md):**
```markdown
Given a candidate profile (LinkedIn data, resume snippet, or ATS record) and a job
order, evaluate the candidate's fit. Consider:

1. Skills overlap (required vs. present)
2. Experience level alignment
3. Industry relevance (exact match, adjacent, or transferable)
4. Location feasibility
5. Likely salary expectations vs. job budget
6. Flight risk indicators (passive vs. active, tenure patterns)

Provide a 0-100 relevance score and a 2-sentence assessment.
Do NOT factor in any demographic information.
```

#### n8n Workflow Design

```
[Trigger: New Job Order in ATS / Manual / Cron (daily pipeline check)]
    → [Fetch Job Requirements from ATS]
    → [Claude API: Sonnet — Generate search queries for each source]
    → [PARALLEL BRANCHES:]
        ├─ [Search ATS Database (internal API)] → [Collect results]
        ├─ [Search LinkedIn (via Apify actor or API)] → [Collect results]
        └─ [Search Job Boards (Indeed API / scraper)] → [Collect results]
    → [Merge Results]
    → [Deduplicate (email + name matching)]
    → [Cross-reference against existing ATS records]
    → [Claude API: Haiku — Batch score all candidates (Batch API)]
    → [Rank by relevance score]
    → [Top candidates → Claude API: Sonnet — Generate personalized outreach drafts]
    → [Add new candidates to ATS with source attribution]
    → [Notify recruiter: "Found {n} new candidates for {job_title}"]
    → [Pass qualified candidates to Resume Screening Agent for full evaluation]
```

#### Tools & API Integrations
- **Claude API**: Sonnet for query building + outreach drafts, Haiku for batch scoring
- **LinkedIn**: Apify LinkedIn Profile Scraper actor (~$0.01 per profile) or LinkedIn Recruiter API (if licensed)
- **Indeed Publisher API**: Job seeker resume search (limited)
- **ATS Internal Search**: Via adapter layer
- **Google Custom Search API**: For finding candidate profiles on personal sites, GitHub, etc.

#### Estimated Cost
- Per sourcing session: $0.50–$2.00 (Claude) + $0.50–$5.00 (LinkedIn scraping for 50 profiles)
- Monthly (20 active job orders): $20–$140/month

#### Inter-Agent Connections
- **Triggered by**: New job order in ATS, Recruiter Productivity Agent (pipeline gap alert), Market Intelligence Agent (new demand signal)
- **Sends to**: Resume Screening Agent (for scoring), Client Outreach Agent (when candidates found for speculative roles)

---

### 4.3 Client Outreach / Cold Email Agent

**Purpose:** Generate hyper-personalized cold email sequences to prospective staffing clients, managing multi-touch campaigns that feel hand-written.

#### Inputs
- Target company data (name, industry, size, location, recent news, job postings)
- Contact person data (name, title, LinkedIn profile, email)
- Agency value proposition (which verticals, specialties, differentiators)
- Sequence configuration (number of touches, cadence, tone)
- Market intelligence data (relevant trends for personalization)

#### Outputs
```json
{
  "campaign_id": "string",
  "contact": {
    "name": "string",
    "email": "string",
    "company": "string",
    "title": "string"
  },
  "sequence": [
    {
      "step": 1,
      "delay_days": 0,
      "subject": "string",
      "body": "string (plain text, personalized)",
      "personalization_points": ["referenced their Series B", "mentioned their Austin expansion"],
      "call_to_action": "15-min intro call"
    },
    {
      "step": 2,
      "delay_days": 3,
      "subject": "Re: {original_subject}",
      "body": "string (follow-up, different angle)",
      "personalization_points": ["referenced their open DevOps roles on careers page"]
    },
    {
      "step": 3,
      "delay_days": 5,
      "subject": "string (breakup email)",
      "body": "string (final touch, low-pressure)"
    }
  ],
  "reply_handling": {
    "positive_reply_action": "Notify recruiter, schedule meeting, move to CRM pipeline",
    "negative_reply_action": "Mark as not interested, add to do-not-contact for 90 days",
    "out_of_office_action": "Pause sequence, resume after return date",
    "bounce_action": "Flag for email verification, try alternate email"
  }
}
```

#### Claude Prompt Architecture

**Company Research (company-research.md):**
```markdown
Research this company and extract key information for a staffing sales outreach:

EXTRACT:
1. What they do (1 sentence)
2. Company size and growth stage
3. Recent news (funding, expansion, product launches, leadership changes)
4. Current job openings (especially roles our agency could fill)
5. Tech stack or industry vertical
6. Pain points we can infer (rapid hiring = scaling pain, many open roles = sourcing struggle)
7. Decision maker identification (VP Talent, Head of People, HR Director patterns)

IMPORTANT: Only use verifiable information. Do not fabricate news or details.
```

**Email Personalization (email-personalization.md):**
```markdown
You are writing a cold email from a staffing agency recruiter to a prospective client.

RULES:
- First line must reference something SPECIFIC about their company (not generic flattery)
- Keep subject lines under 50 characters, curiosity-driven, no spam triggers
- Body must be under 125 words
- One clear CTA (not multiple asks)
- Tone: peer-to-peer, not salesy. You're a specialist sharing insight, not pitching
- No "I hope this email finds you well" or similar filler
- Include a relevant market data point when available (e.g., "average time-to-fill for DevOps in Austin is 52 days")
- Each follow-up must take a DIFFERENT angle (not "just circling back")

PERSONALIZATION HIERARCHY (use highest available):
1. Reference their specific open role by title
2. Reference recent company news/milestone
3. Reference industry-specific challenge with data
4. Reference mutual connection or shared community
5. Reference geographic/market trend
```

#### n8n Workflow Design

```
[Trigger: New prospect list uploaded / CRM stage change / Manual]
    → [Enrich Contact Data (Apify Apollo scraper or Apollo API)]
    → [Fetch Company Info (LinkedIn company profile, website scrape)]
    → [Claude API: Sonnet — Research company + extract personalization points]
    → [Claude API: Sonnet — Generate 3-step email sequence]
    → [Validate: No spam trigger words, correct merge fields, proper formatting]
    → [Push to Email Platform (Instantly / Smartlead / custom SMTP)]
    → [Set up reply webhook listener]
    → [REPLY HANDLER:]
        ├─ [Positive reply → Classify → Notify recruiter → Create CRM opportunity]
        ├─ [Negative reply → Classify → Add to suppression list]
        ├─ [OOO reply → Parse return date → Pause and reschedule]
        └─ [Bounce → Flag contact → Try alternate email if available]
    → [Log all activity to analytics database]
```

#### Tools & API Integrations
- **Claude API**: Sonnet for research + personalization (quality matters for outreach)
- **Instantly.ai** or **Smartlead**: Email sending, warm-up, deliverability
- **Apify Apollo Scraper**: Contact enrichment ($0.01–0.03 per contact)
- **Company data**: LinkedIn Company Scraper, Clearbit, or web scraping
- **CRM**: Push opportunities to Bullhorn, HubSpot, or Pipedrive

#### Estimated Cost
- Per prospect sequence (3 emails): $0.02–0.05 (Claude) + $0.02 (enrichment)
- Monthly (500 prospects): $20–$35/month (Claude + enrichment, excluding email tool subscription)
- Email platform: $100–$300/month (Instantly or Smartlead)

#### Inter-Agent Connections
- **Receives from**: Market Intelligence Agent (hot markets → target companies), Analytics Agent (performance data for optimization)
- **Sends to**: Client Health Agent (new client onboarded), Recruiter Productivity Agent (meetings booked)

---

### 4.4 Interview Scheduling Agent

**Purpose:** Eliminate the back-and-forth of coordinating interviews between candidates, hiring managers, and sometimes multiple interviewers across time zones.

#### Inputs
- Candidate contact info + calendar preferences
- Hiring manager/interviewer availability (from calendar API or manual input)
- Interview requirements (duration, format: in-person/video/phone, panel size)
- Time zone information for all parties
- Scheduling constraints (e.g., "not before 10am", "prefer Tuesday/Thursday")

#### Outputs
```json
{
  "scheduling_request_id": "string",
  "status": "SCHEDULED | PENDING_CANDIDATE | PENDING_CLIENT | CONFLICT",
  "interview": {
    "date": "2026-03-27",
    "time": "2:00 PM EST",
    "duration_minutes": 60,
    "format": "video",
    "meeting_link": "https://zoom.us/j/...",
    "participants": [
      {"name": "string", "role": "candidate", "email": "string", "confirmed": true},
      {"name": "string", "role": "interviewer", "email": "string", "confirmed": true}
    ]
  },
  "calendar_events_created": ["google_event_id_1", "google_event_id_2"],
  "confirmation_emails_sent": ["candidate@email.com", "manager@company.com"],
  "reminder_schedule": [
    {"when": "24h_before", "to": "all_participants"},
    {"when": "1h_before", "to": "all_participants"}
  ],
  "fallback_slots": [
    {"date": "2026-03-28", "time": "10:00 AM EST"},
    {"date": "2026-03-31", "time": "3:00 PM EST"}
  ]
}
```

#### Claude Prompt Architecture

**Availability Parser (availability-parser.md):**
```markdown
Parse the following natural language availability information and extract structured
time slots. Handle:

- "I'm free Tuesday and Thursday afternoons" → specific date ranges + afternoon hours
- "Anytime next week except Wednesday" → enumerate available slots
- "After 2pm EST works" → time zone conversion if needed
- "Morning works best" → 9am-12pm in their local time zone

OUTPUT: Array of {date, start_time, end_time, timezone} objects.
Always confirm timezone. If ambiguous, flag for clarification.
```

**Confirmation Composer (confirmation-composer.md):**
```markdown
Write interview confirmation emails that are:
- Professional but warm
- Include ALL logistics (date, time with timezone, format, link/address, who they'll meet)
- Include preparation tips appropriate to interview stage
- For candidates: remind them of the company and role
- For clients: remind them of the candidate highlights
- Keep under 150 words
```

#### n8n Workflow Design

```
[Trigger: Candidate moves to "Interview" stage in ATS / Manual trigger]
    → [Fetch candidate details from ATS]
    → [Fetch interviewer(s) from job order record]
    → [Check Interviewer Availability (Google Calendar API / Calendly API)]
    → [Check Candidate Availability:]
        ├─ [IF Calendly/Cal.com link exists → Fetch available slots]
        └─ [ELSE → Send availability request email to candidate]
            → [Wait for reply (webhook or email parse)]
            → [Claude API: Haiku — Parse availability from email response]
    → [Match overlapping slots across all participants]
    → [IF match found:]
        ├─ [Create calendar events (Google Calendar API)]
        ├─ [Create video meeting link (Zoom/Teams/Google Meet API)]
        ├─ [Claude API: Haiku — Generate confirmation emails]
        ├─ [Send confirmation to all parties]
        ├─ [Update ATS record with interview details]
        └─ [Schedule reminder workflow (24h + 1h before)]
    → [IF no match:]
        ├─ [Suggest alternative slots to candidate]
        ├─ [Notify recruiter of scheduling conflict]
        └─ [Queue for re-attempt in 24h]
```

#### Tools & API Integrations
- **Claude API**: Haiku for availability parsing + email composition
- **Google Calendar API**: Check availability, create events (OAuth 2.0)
- **Calendly API**: Scheduling link creation, webhook for booking confirmation
- **Cal.com**: Open-source alternative to Calendly (self-hosted option)
- **Zoom API**: Meeting link generation
- **Google Meet**: Via Google Calendar event with conferencing
- **Email (Gmail/SMTP)**: Send confirmations and reminders

#### Estimated Cost
- Per interview scheduled: $0.002–0.005 (Claude) + free (calendar APIs)
- Monthly (50 interviews): $0.10–$0.25/month Claude costs
- Calendly Pro (if used): $15–$20/month

#### Inter-Agent Connections
- **Receives from**: Resume Screening Agent (qualified candidates ready for interview), Recruiter Productivity Agent (scheduling requests)
- **Sends to**: Post-Placement Agent (interview outcomes feed placement pipeline), Analytics Agent (time-to-schedule metrics), Recruiter Productivity Agent (interview calendar updates)

---

### 4.5 Compliance Monitoring Agent

**Purpose:** Continuously monitor credential expirations, I-9 compliance, background check status, state licensing requirements, and EEOC adverse impact metrics across all active placements and candidates.

#### Inputs
- Candidate/employee records (certifications, licenses, I-9 status, background check dates)
- Placement records (start date, end date, work location state, client requirements)
- Compliance rule configuration (which regulations apply, thresholds, deadlines)
- State-specific requirements (varies by placement state)
- Client-specific requirements (some clients have additional compliance needs)

#### Outputs
```json
{
  "compliance_dashboard": {
    "total_active_placements": 85,
    "fully_compliant": 78,
    "action_needed": 5,
    "critical_alerts": 2,
    "upcoming_expirations_30d": 8
  },
  "alerts": [
    {
      "severity": "CRITICAL",
      "type": "certification_expiring",
      "employee": "Jane Smith",
      "placement_id": "P-1234",
      "detail": "RN license expires in 7 days (March 31, 2026)",
      "action_required": "Contact employee to verify renewal status",
      "action_deadline": "2026-03-28",
      "auto_actions_taken": ["Email sent to employee", "Recruiter notified via Slack"]
    }
  ],
  "eeoc_report": {
    "period": "Q1 2026",
    "selection_rates": {
      "overall": 0.15,
      "by_group": {},
      "four_fifths_rule_violations": [],
      "recommendation": "No adverse impact detected this quarter"
    }
  },
  "i9_status": {
    "total_active": 85,
    "complete": 83,
    "pending_section_2": 1,
    "reverification_needed": 1
  }
}
```

#### Claude Prompt Architecture

**Document Classifier (document-classifier.md):**
```markdown
Classify the uploaded document and extract compliance-relevant information:

DOCUMENT TYPES:
- Professional license (nursing, CPA, PE, etc.) → Extract: license number, state, expiration date, holder name
- Certification (AWS, PMP, OSHA, etc.) → Extract: cert name, cert number, issue date, expiration date
- I-9 supporting document (passport, driver's license, SSN card, etc.) → Extract: document type, list (A/B/C), expiration
- Background check result → Extract: status (clear/flagged), date, scope
- Drug test result → Extract: status, date, panel type
- W-4 / tax form → Extract: filing status, date signed
- Workers comp documentation → Extract: policy number, coverage dates, state

If the document is unclear or potentially fraudulent, flag for human review.
Do NOT store or output full SSN, just last 4 digits.
```

**Expiration Tracker (expiration-tracker.md):**
```markdown
Given a list of employee credentials with expiration dates, generate:

1. CRITICAL (expires within 7 days): Immediate action needed
2. WARNING (expires within 30 days): Plan renewal now
3. UPCOMING (expires within 90 days): Add to renewal pipeline
4. OK (expires 90+ days): No action needed

For each expiring credential, provide:
- Clear action steps for the recruiter
- Suggested email/notification to the employee
- Whether the placement is at risk if not renewed
- State-specific renewal requirements if applicable
```

#### n8n Workflow Design

```
[Trigger: Daily cron job (6:00 AM) + Real-time ATS events]
    → [Fetch all active placements from ATS]
    → [Fetch credential records for each placed employee]
    → [PARALLEL CHECKS:]
        ├─ [Certification Expiration Scan]
        │   → [Compare expiration dates to today + 7/30/90 day windows]
        │   → [Flag items by severity]
        │
        ├─ [I-9 Compliance Check]
        │   → [Verify Section 1 + Section 2 complete for all active]
        │   → [Check List B documents for expiration]
        │   → [Flag reverification needs]
        │
        ├─ [Background Check Status]
        │   → [Verify all placements have cleared background checks]
        │   → [Check for any pending results]
        │
        ├─ [State Licensing Check]
        │   → [Match placement state to required licenses]
        │   → [Verify current licensing in work state]
        │
        └─ [Client-Specific Requirements]
            → [Check client's additional requirements (drug testing, specific certs)]
            → [Flag any gaps]
    → [Merge all alerts]
    → [Claude API: Haiku — Prioritize alerts and generate action recommendations]
    → [CRITICAL alerts → Immediate Slack notification to recruiter + manager]
    → [WARNING alerts → Email to recruiter + automated email to employee]
    → [UPCOMING alerts → Add to weekly digest]
    → [Update compliance dashboard in database]
    → [Generate weekly compliance report (Sundays)]

[Quarterly Sub-Workflow: EEOC Adverse Impact Analysis]
    → [Pull all hiring decisions for quarter]
    → [Calculate selection rates by demographic group (if data available)]
    → [Apply four-fifths rule]
    → [Claude API: Sonnet — Analyze results and generate recommendations]
    → [Generate EEOC report PDF]
    → [Notify compliance officer]
```

#### Tools & API Integrations
- **Claude API**: Haiku for document classification + alert prioritization, Sonnet for EEOC analysis
- **ATS API**: Pull placement and credential data
- **E-Verify API**: Automated employment eligibility verification
- **State licensing databases**: Web scraping or API where available (varies by state)
- **Slack/Email**: Alert notifications
- **Google Sheets / Airtable**: Compliance tracking dashboard (lightweight option)
- **PDF generation**: For compliance reports

#### Compliance Rules Engine (rules/federal.json)
```json
{
  "i9": {
    "section_1_deadline_days": 1,
    "section_2_deadline_days": 3,
    "reverification_trigger": "document_expiration",
    "retention_after_termination_years": 3
  },
  "background_check": {
    "required_before_start": true,
    "max_age_days": 365,
    "types_required": ["criminal", "employment_verification"],
    "additional_for_healthcare": ["ocs_check", "drug_screen"]
  },
  "eeoc": {
    "adverse_impact_threshold": 0.80,
    "analysis_frequency": "quarterly",
    "minimum_sample_size": 30
  }
}
```

#### Estimated Cost
- Daily compliance scan: $0.05–$0.15 (Claude Haiku for 85 placements)
- Monthly: $1.50–$4.50/month (Claude) + minimal API costs
- Quarterly EEOC analysis: $0.10–$0.30 (Claude Sonnet)

#### Inter-Agent Connections
- **Receives from**: Post-Placement Agent (new placement data), ATS events (credential updates)
- **Sends to**: Recruiter Productivity Agent (compliance tasks for daily briefing), Analytics Agent (compliance metrics), Client Health Agent (compliance issues that affect client relationship)

---

### 4.6 Market Intelligence Agent

**Purpose:** Continuously gather and analyze salary data, hiring demand trends, competitor activity, and market conditions to inform pricing, positioning, and business development strategy.

#### Inputs
- Target markets (geographic regions, industry verticals, role categories)
- Current agency pricing/bill rates
- Competitor list (other staffing agencies in same markets)
- Job board data (Indeed, LinkedIn, Glassdoor)
- Bureau of Labor Statistics (BLS) data
- Industry reports and news

#### Outputs
```json
{
  "report_date": "2026-03-24",
  "market_snapshot": {
    "overall_demand_trend": "INCREASING",
    "demand_change_30d": "+8%",
    "supply_tightness": "TIGHT",
    "avg_time_to_fill_market": "44 days"
  },
  "role_insights": [
    {
      "role_category": "DevOps Engineer",
      "market": "Austin, TX",
      "demand_level": "HIGH",
      "demand_change_30d": "+15%",
      "salary_range": {"p25": 130000, "p50": 155000, "p75": 180000},
      "salary_change_yoy": "+6.2%",
      "bill_rate_suggestion": {"low": 85, "mid": 95, "high": 110},
      "candidate_supply": "LOW",
      "competitor_activity": "3 agencies actively posting for similar roles",
      "recommendation": "Increase bill rates by $5/hr. Source passive candidates. Pitch to clients expanding DevOps teams."
    }
  ],
  "competitor_intel": [
    {
      "competitor": "TechStaff Inc",
      "recent_activity": "Posted 12 new DevOps roles in Austin this week",
      "estimated_bill_rates": "MARKET",
      "client_wins_losses": "Won contract with Acme Corp (previously our client)"
    }
  ],
  "opportunities": [
    {
      "type": "market_gap",
      "description": "AI/ML engineering demand up 40% YoY in Austin but only 2 agencies actively sourcing",
      "recommended_action": "Launch AI/ML practice area. Target 10 companies with open AI roles.",
      "potential_revenue": "$150K-300K annual"
    }
  ],
  "alerts": [
    {
      "type": "client_risk",
      "detail": "Your client Acme Corp posted 5 roles directly on LinkedIn that match roles you're filling. Potential insourcing risk."
    }
  ]
}
```

#### Claude Prompt Architecture

**Salary Analyzer (salary-analyzer.md):**
```markdown
Analyze the following job posting data and salary survey information to produce
market rate insights for staffing agencies.

For each role + market combination:
1. Calculate 25th, 50th, and 75th percentile salary ranges
2. Convert salary to bill rate using standard staffing markups:
   - Contract: salary / 2080 * (1 + burden_rate) * (1 + margin)
   - Burden rate: 22-30% (taxes, insurance, benefits)
   - Target margin: 20-35% depending on role type
3. Compare to historical data to identify trends
4. Flag roles where demand is outpacing supply (indicator: time-to-fill increasing, job posting volume up, salary offers increasing)

Be precise with numbers. Use actual data provided, not assumptions.
```

**Demand Forecaster (demand-forecaster.md):**
```markdown
Given job posting volume data, economic indicators, and industry news, forecast
hiring demand for the next 30/60/90 days.

SIGNALS TO WEIGHT:
- Job posting volume change (30-day trailing) — strongest signal
- Company earnings reports mentioning hiring plans
- VC funding rounds (Series B+ = hiring surge in 6-12 months)
- Layoff announcements (negative signal for that company, but may increase supply)
- Seasonal patterns (Q1 budget release = demand spike, December = slowdown)
- Technology adoption trends (new tech = new role demand)

OUTPUT: Demand forecast by role category with confidence level (HIGH/MEDIUM/LOW).
```

#### n8n Workflow Design

```
[Trigger: Weekly cron (Monday 6:00 AM) + On-demand for specific markets]
    → [PARALLEL DATA COLLECTION:]
        ├─ [Scrape Job Postings: Indeed/LinkedIn for target roles + markets]
        │   → [Apify Indeed Scraper: volume counts by role/location]
        │   → [Apify LinkedIn Jobs Scraper: detailed posting data]
        │
        ├─ [Fetch Salary Data:]
        │   → [BLS API: Occupational Employment Statistics]
        │   → [Glassdoor API or scraper: company-specific salary data]
        │   → [Levels.fyi scraper for tech roles (if applicable)]
        │
        ├─ [Competitor Monitoring:]
        │   → [Scrape competitor websites for new job postings]
        │   → [LinkedIn company follower/posting activity]
        │
        └─ [Industry News:]
            → [Google News API: industry keywords + target companies]
            → [Crunchbase API: funding rounds in target markets]
    → [Merge all data sources]
    → [Claude API: Sonnet — Analyze market data and generate insights report]
    → [Claude API: Sonnet — Generate actionable recommendations]
    → [Store report in database]
    → [Send weekly market intelligence digest to leadership]
    → [Flag urgent opportunities/threats → Immediate Slack notification]
    → [Feed relevant insights to Client Outreach Agent for personalization]
```

#### Estimated Cost
- Weekly market scan: $5–$15 (scraping via Apify) + $0.50–$1.00 (Claude Sonnet for analysis)
- Monthly: $20–$65/month total

#### Inter-Agent Connections
- **Sends to**: Client Outreach Agent (market insights for personalization), Recruiter Productivity Agent (market context for daily briefings), Analytics Agent (market benchmarks for performance context), Client Health Agent (competitive threat alerts)
- **Receives from**: Analytics Agent (internal data for comparison)

---

### 4.7 Client Health / Retention Agent

**Purpose:** Monitor the health of every client relationship using quantitative signals, predict churn risk before it happens, and trigger proactive retention interventions.

#### Inputs
- Placement history per client (volume, frequency, fill rates, margin)
- Communication patterns (email frequency, response times, meeting cadence)
- Job order pipeline (open orders, time since last order, order acceptance rate)
- NPS/satisfaction survey data (if collected)
- Billing/payment patterns (late payments, disputed invoices)
- Compliance incident history
- Market context from Market Intelligence Agent

#### Outputs
```json
{
  "client_health_scores": [
    {
      "client_id": "string",
      "client_name": "Acme Corp",
      "health_score": 72,
      "health_trend": "DECLINING",
      "risk_level": "MEDIUM",
      "score_components": {
        "placement_velocity": 65,
        "fill_rate": 80,
        "communication_engagement": 60,
        "payment_health": 90,
        "order_pipeline": 55,
        "satisfaction_signals": 70
      },
      "warning_signals": [
        "No new job orders in 45 days (avg is 21 days)",
        "Response time to candidate submissions increased from 1 day to 4 days",
        "Saw them post 3 roles directly on LinkedIn"
      ],
      "recommended_interventions": [
        {
          "action": "Schedule QBR (quarterly business review)",
          "urgency": "THIS_WEEK",
          "talking_points": [
            "Share market data on their key roles",
            "Propose dedicated sourcing for their AI engineering needs",
            "Address any concerns about recent placement quality"
          ]
        },
        {
          "action": "Send market intelligence report",
          "urgency": "TODAY",
          "detail": "Custom report on DevOps salary trends in their market"
        }
      ],
      "lifetime_value": "$340,000",
      "annual_revenue": "$85,000",
      "churn_probability_90d": 0.35
    }
  ],
  "portfolio_summary": {
    "total_clients": 42,
    "healthy": 28,
    "at_risk": 10,
    "critical": 4,
    "total_annual_revenue_at_risk": "$520,000"
  }
}
```

#### Claude Prompt Architecture

**Health Scorer (health-scorer.md):**
```markdown
Calculate a client health score (0-100) based on the following signals.

SCORING WEIGHTS:
- Placement velocity (25%): How often they send job orders vs. historical baseline
- Fill rate (20%): What % of their orders we successfully fill
- Communication engagement (20%): Response times, meeting frequency, proactive outreach from them
- Payment health (15%): On-time payment %, outstanding balance
- Order pipeline (10%): Open orders, upcoming predicted need
- Satisfaction signals (10%): Survey scores, verbal feedback, referral activity

RISK THRESHOLDS:
- HEALTHY: Score ≥ 75
- AT RISK: Score 50-74
- CRITICAL: Score < 50

TREND: Compare current score to 30-day and 90-day prior scores.
If declining >10 points in 30 days = "DECLINING"
If stable ±5 points = "STABLE"
If increasing >10 points = "IMPROVING"

For each AT RISK or CRITICAL client, generate specific, actionable intervention
recommendations. Be concrete — not "improve communication" but "Schedule a call
this week to discuss their Q2 hiring plans and share the DevOps salary report."
```

#### n8n Workflow Design

```
[Trigger: Weekly cron (Monday 7:00 AM) + On-demand per client]
    → [Fetch all active client records from ATS/CRM]
    → [For each client, PARALLEL data gathering:]
        ├─ [Placement history: last 12 months of placements, margins, fill rates]
        ├─ [Communication data: email volume, response times (from email API)]
        ├─ [Job order pipeline: open orders, last order date, order frequency]
        ├─ [Billing data: payment timeliness, outstanding invoices]
        └─ [Survey data: NPS scores, feedback (if available)]
    → [Compile client data package]
    → [Claude API: Sonnet — Calculate health score + generate interventions]
    → [Store health scores in database (time series for trending)]
    → [CRITICAL clients → Immediate Slack alert to account manager + leadership]
    → [AT RISK clients → Email weekly digest to account managers]
    → [Generate portfolio health report for leadership]
    → [Feed intervention recommendations to Recruiter Productivity Agent]
```

#### Estimated Cost
- Weekly scoring (42 clients): $0.30–$0.80 (Claude Sonnet)
- Monthly: $1.20–$3.20/month (Claude only)

#### Inter-Agent Connections
- **Receives from**: Post-Placement Agent (placement satisfaction data), Compliance Agent (incident reports), Market Intelligence Agent (competitive threats), Analytics Agent (performance metrics)
- **Sends to**: Recruiter Productivity Agent (client priorities for daily briefing), Client Outreach Agent (reactivation targets), Analytics Agent (health score trends)

---

### 4.8 Recruiter Productivity Agent

**Purpose:** Serve as each recruiter's AI chief of staff — providing daily briefings, prioritizing tasks, automating administrative work, and ensuring nothing falls through the cracks.

#### Inputs
- Recruiter's active job orders and pipeline
- Calendar for the day
- Unread emails and pending communications
- Compliance alerts assigned to this recruiter
- Client health alerts for their accounts
- New candidate applications overnight
- Interview feedback pending
- Placement anniversaries and check-in due dates

#### Outputs
```json
{
  "recruiter": "Sarah Johnson",
  "briefing_date": "2026-03-24",
  "priority_summary": "3 critical items need attention today. You have 2 interviews to prep for and 1 client at risk of churning.",
  "critical_actions": [
    {
      "priority": 1,
      "type": "client_risk",
      "action": "Call Tom at Acme Corp — no new orders in 45 days, declining engagement",
      "context": "Health score dropped from 82 to 62. They posted 3 DevOps roles directly on LinkedIn.",
      "suggested_talking_points": ["Q2 hiring plans", "Exclusive partnership on DevOps roles", "Share Austin DevOps salary data"]
    },
    {
      "priority": 2,
      "type": "compliance",
      "action": "Jane Smith's RN license expires Friday — confirm renewal status",
      "context": "She's placed at Memorial Hospital. No replacement available if license lapses."
    },
    {
      "priority": 3,
      "type": "candidate_followup",
      "action": "Send offer details to Michael Chen — accepted verbal offer yesterday",
      "context": "Start date March 31. Need signed offer letter + background check initiated today."
    }
  ],
  "pipeline_snapshot": {
    "active_job_orders": 8,
    "candidates_in_pipeline": 34,
    "interviews_today": 2,
    "submissions_due_today": 3,
    "offers_pending": 1
  },
  "today_schedule": [
    {"time": "9:00 AM", "event": "Interview prep: Michael Chen → Acme Corp Sr. Developer"},
    {"time": "10:00 AM", "event": "Client call: Acme Corp status update"},
    {"time": "2:00 PM", "event": "Interview: Sarah Williams for Beta Inc DevOps role"}
  ],
  "quick_wins": [
    "3 new applications overnight for the Beta Inc DevOps role — all scored >75 by Resume Screening Agent",
    "Market Intelligence found 5 passive candidates matching the Gamma Corp AI Engineer req"
  ],
  "admin_automated": [
    "Sent 12 interview confirmation reminders",
    "Updated 5 candidate records with new screening scores",
    "Sent weekly placement check-in to 3 contractors"
  ]
}
```

#### Claude Prompt Architecture

**Daily Briefing (daily-briefing.md):**
```markdown
You are preparing a daily briefing for a staffing recruiter. Think like a great
executive assistant who knows exactly what matters today.

PRIORITIZATION FRAMEWORK:
1. CRITICAL: Revenue at risk, compliance deadlines, client churn signals
2. HIGH: Active offers, interview prep, client submissions due
3. MEDIUM: New applications to review, follow-ups, pipeline building
4. LOW: Administrative tasks, data cleanup, long-term sourcing

TONE: Direct, actionable, no fluff. Every item should answer "what do I need to DO?"
Include context that helps the recruiter take action immediately (key facts, talking points, history).

Keep the briefing scannable — recruiter should absorb priorities in 60 seconds.
```

**Task Prioritizer (task-prioritizer.md):**
```markdown
Given a recruiter's current pipeline, pending tasks, and calendar, create a
prioritized task list for the day. Consider:

1. Time sensitivity (deadlines today vs. this week vs. this month)
2. Revenue impact (offer in progress > new job order > sourcing)
3. Client importance (revenue tier, health score, relationship depth)
4. Candidate stage (further in pipeline = higher priority to progress)
5. Dependency chains (what's blocking other tasks?)

Tag each task with estimated time to complete and optimal time slot.
```

#### n8n Workflow Design

```
[Trigger: Daily cron (7:00 AM, 30 min before recruiter start time)]
    → [Identify recruiter(s) to brief (can be per-recruiter or all)]
    → [PARALLEL data gathering for each recruiter:]
        ├─ [Fetch their active job orders from ATS]
        ├─ [Fetch their candidate pipelines from ATS]
        ├─ [Fetch today's calendar (Google Calendar API)]
        ├─ [Fetch compliance alerts (from Compliance Agent output)]
        ├─ [Fetch client health alerts (from Client Health Agent output)]
        ├─ [Fetch overnight applications (ATS query: last 16 hours)]
        ├─ [Fetch pending interview feedback requests]
        └─ [Fetch placement check-in due dates (from Post-Placement Agent)]
    → [Compile all data into recruiter context package]
    → [Claude API: Sonnet — Generate prioritized daily briefing]
    → [Deliver briefing via preferred channel:]
        ├─ [Slack DM (formatted with sections and priorities)]
        ├─ [Email (formatted HTML digest)]
        └─ [ATS dashboard note (if supported)]
    → [Log briefing to analytics database]

[Ad-hoc Sub-workflow: Recruiter asks "What should I work on next?"]
    → [Webhook trigger from Slack slash command or chatbot]
    → [Re-generate current priorities based on latest data]
    → [Respond in Slack thread]
```

#### Estimated Cost
- Daily briefing per recruiter: $0.02–$0.05 (Claude Sonnet)
- Monthly (5 recruiters, 22 work days): $2.20–$5.50/month

#### Inter-Agent Connections
- **Receives from**: ALL other agents — this is the aggregation layer for recruiter-facing information
- **Sends to**: Analytics Agent (recruiter activity metrics), triggers to other agents (e.g., "recruiter requested sourcing for Job X")

---

### 4.9 Post-Placement Follow-up Agent

**Purpose:** Manage systematic check-in sequences with placed candidates and their hiring managers to ensure placement success, identify issues early, and generate redeployment opportunities.

#### Inputs
- Placement records (candidate, client, role, start date, contract duration, bill rate)
- Check-in schedule configuration (default: Day 1, Week 1, Week 4, Month 3, before contract end)
- Previous check-in responses and sentiment
- Placement history for this candidate (repeat placements)
- Client feedback patterns

#### Outputs
```json
{
  "placement_id": "P-1234",
  "candidate": "Michael Chen",
  "client": "Acme Corp",
  "role": "Senior Developer",
  "start_date": "2026-01-15",
  "contract_end": "2026-07-15",
  "check_in_history": [
    {
      "check_in": "day_1",
      "date": "2026-01-15",
      "candidate_sentiment": "POSITIVE",
      "client_sentiment": "POSITIVE",
      "issues": [],
      "notes": "Smooth onboarding. Badge and laptop ready."
    },
    {
      "check_in": "week_1",
      "date": "2026-01-22",
      "candidate_sentiment": "POSITIVE",
      "client_sentiment": "POSITIVE",
      "issues": [],
      "notes": "Getting up to speed. Good team dynamics."
    },
    {
      "check_in": "month_1",
      "date": "2026-02-15",
      "candidate_sentiment": "NEUTRAL",
      "client_sentiment": "POSITIVE",
      "issues": ["Candidate mentioned longer commute than expected"],
      "notes": "Performing well but commute is 90 min. May request hybrid schedule.",
      "flag": "MONITOR"
    }
  ],
  "next_check_in": {
    "type": "month_3",
    "due_date": "2026-04-15",
    "candidate_email_draft": "string",
    "client_email_draft": "string"
  },
  "redeployment_opportunity": {
    "contract_ends_in_days": 113,
    "candidate_interest_in_extension": "UNKNOWN",
    "client_extension_likelihood": "HIGH",
    "alternative_roles_matched": 2,
    "recommendation": "Begin extension conversation at month-3 check-in. If not extending, start sourcing alternative roles — 2 matches in current pipeline."
  }
}
```

#### Claude Prompt Architecture

**Check-in Composer (check-in-composer.md):**
```markdown
Write a check-in email for a placed candidate at a specific stage of their assignment.

STAGE-SPECIFIC GUIDELINES:
- Day 1: Short, warm. "How was your first day?" focus on logistics (badge, access, team intro)
- Week 1: Slightly deeper. "How are things going?" focus on team fit, workload clarity
- Month 1: Substantive. Ask about project involvement, manager relationship, any concerns
- Month 3: Strategic. Ask about long-term fit, career development, interest in extension
- Pre-end: Transition-focused. Interest in extension? Open to new roles? What went well?

For CLIENT check-ins:
- Focus on performance and fit
- Ask if there are any concerns to address
- Subtly gauge extension/conversion interest
- At month 3+, begin extension conversation naturally

TONE: Caring but professional. You're their advocate. Make it easy to share concerns.
Keep emails under 100 words. One question is better than five.
```

**Satisfaction Analyzer (satisfaction-analyzer.md):**
```markdown
Analyze the following check-in response (email, survey, or call notes) and extract:

1. Overall sentiment: POSITIVE, NEUTRAL, NEGATIVE
2. Specific issues mentioned (categorize: workload, culture, compensation, logistics, management, technical)
3. Flight risk indicators (mentions of looking elsewhere, frustration, disengagement)
4. Extension/conversion signals (positive mentions of team, project, wanting to stay)
5. Urgency level: IMMEDIATE_ACTION, MONITOR, NO_CONCERN

If NEGATIVE sentiment or IMMEDIATE_ACTION urgency, draft:
- A response to the candidate acknowledging their concern
- An internal alert message to the recruiter with specific follow-up actions
```

#### n8n Workflow Design

```
[Trigger: Placement Created in ATS + Daily cron for scheduled check-ins]

PLACEMENT CREATED:
    → [Record placement in check-in database with full schedule]
    → [Schedule Day 1 check-in for start date]

DAILY CHECK-IN RUNNER:
    → [Query database for check-ins due today]
    → [For each due check-in:]
        → [Fetch placement details from ATS]
        → [Fetch previous check-in history]
        → [Claude API: Haiku — Generate personalized check-in emails]
        → [Send candidate check-in email]
        → [Send client check-in email (if applicable for this stage)]
        → [Set up reply monitoring webhook]

REPLY HANDLER:
    → [Email received from candidate/client matching a placement]
    → [Claude API: Haiku — Analyze sentiment and extract issues]
    → [IF NEGATIVE or IMMEDIATE_ACTION:]
        ├─ [Slack alert to recruiter]
        ├─ [Create task in ATS/CRM]
        └─ [Claude API: Haiku — Draft response email for recruiter to review]
    → [IF mention of extension interest:]
        ├─ [Flag for redeployment pipeline]
        └─ [Notify recruiter of extension opportunity]
    → [Store check-in result in database]
    → [Update Client Health Agent data]

PRE-END WORKFLOW (triggered 30 days before contract end):
    → [Claude API: Sonnet — Match candidate to other open roles (Redeployment Matcher)]
    → [Notify recruiter: "Contract ending in 30 days. Extension likelihood: X. Alternative matches: Y."]
    → [Generate candidate check-in asking about extension/new opportunity interest]
```

#### Estimated Cost
- Per check-in cycle (compose + analyze): $0.003–0.008 (Claude Haiku)
- Monthly (85 placements, avg 1.5 check-ins each): $0.40–$1.00/month
- Redeployment matching (Sonnet): $0.05–$0.10 per candidate

#### Inter-Agent Connections
- **Receives from**: ATS (new placement events), Compliance Agent (compliance status per placement)
- **Sends to**: Client Health Agent (sentiment data from check-ins), Analytics Agent (placement satisfaction metrics, redeployment rates), Recruiter Productivity Agent (upcoming check-ins and issues), Candidate Sourcing Agent (redeployment candidate profiles)

---

### 4.10 Analytics & Reporting Agent

**Purpose:** Aggregate data from all agents and external sources into unified dashboards, generate narrative reports, identify trends, and provide strategic recommendations.

#### Inputs
- Metrics from all 9 other agents
- ATS data (pipeline, placements, revenue, fill rates)
- Financial data (revenue, margins, costs)
- Recruiter activity data
- Market benchmarks (from Market Intelligence Agent)
- Historical data for trending

#### Outputs
```json
{
  "report_type": "weekly_agency_summary",
  "period": "March 17-23, 2026",
  "kpi_dashboard": {
    "revenue": {
      "this_week": 42000,
      "last_week": 38000,
      "change": "+10.5%",
      "mtd": 165000,
      "monthly_target": 200000,
      "on_track": true
    },
    "placements": {
      "this_week": 3,
      "last_week": 2,
      "mtd": 10,
      "monthly_target": 12,
      "avg_margin": "23.5%"
    },
    "pipeline": {
      "active_job_orders": 32,
      "candidates_in_pipeline": 187,
      "submissions_this_week": 24,
      "interviews_scheduled": 8,
      "offers_extended": 3,
      "offers_accepted": 2,
      "conversion_rate": "66.7%"
    },
    "time_to_fill": {
      "average_days": 28,
      "market_benchmark": 44,
      "vs_benchmark": "-36%",
      "trend": "IMPROVING"
    },
    "client_health": {
      "healthy_clients": 28,
      "at_risk": 10,
      "critical": 4,
      "new_clients_this_month": 2,
      "churned_this_month": 1
    },
    "recruiter_productivity": {
      "avg_submittals_per_recruiter": 4.8,
      "avg_placements_per_recruiter": 2.0,
      "top_performer": "Sarah Johnson (3 placements)",
      "improvement_area": "John needs more sourcing activity on Tech roles"
    }
  },
  "ai_narrative": "Strong week with revenue up 10.5% driven by 3 placements including a high-margin Senior Developer at Acme Corp. Pipeline is healthy with 8 interviews scheduled for next week. Main concern: 4 clients in critical health status representing $520K annual revenue at risk. Recommend leadership QBRs for top 2 at-risk accounts. Market conditions favor us — DevOps demand up 15% in Austin while competitor TechStaff is losing candidates.",
  "recommendations": [
    {
      "category": "revenue",
      "action": "Increase DevOps bill rates by $5/hr based on market data. Would add ~$15K/month at current placement volume.",
      "impact": "HIGH",
      "effort": "LOW"
    },
    {
      "category": "retention",
      "action": "Schedule QBRs with Acme Corp and Beta Inc (critical health clients) this week.",
      "impact": "HIGH",
      "effort": "MEDIUM"
    }
  ]
}
```

#### Claude Prompt Architecture

**KPI Narrator (kpi-narrator.md):**
```markdown
Transform raw KPI data into an executive-level narrative summary.

GUIDELINES:
- Lead with the most important insight (good or bad)
- Compare to benchmarks and historical performance
- Call out anomalies and explain likely causes
- Keep it to 3-5 sentences maximum
- Use specific numbers, not vague language
- End with the single most important action the reader should take

AUDIENCE: Agency owner or VP of Sales. They want to know:
1. Are we on track for our revenue target?
2. What's working and what isn't?
3. What should I do differently this week?
```

**Trend Analyzer (trend-analyzer.md):**
```markdown
Analyze the following time-series metrics and identify:

1. Trends (improving, declining, stable — with magnitude)
2. Seasonality patterns (if historical data spans 12+ months)
3. Correlation between metrics (e.g., does sourcing activity predict placements 2 weeks later?)
4. Leading indicators (which metrics predict future revenue?)
5. Anomalies (unexpected spikes or dips) with potential explanations

Provide insights in order of business impact. Be specific about timeframes
and magnitudes. "Submissions are up" is useless. "Submissions increased 23%
week-over-week, the highest in 3 months, driven by the new DevOps pipeline"
is useful.
```

#### n8n Workflow Design

```
[DAILY PIPELINE:]
    [Trigger: Cron 6:00 AM]
    → [Fetch previous day's activity from all agent databases]
    → [Aggregate into daily metrics]
    → [Store in time-series database]
    → [Feed to Recruiter Productivity Agent for daily briefings]

[WEEKLY REPORT:]
    [Trigger: Cron Sunday 8:00 PM]
    → [Fetch week's data from all sources:]
        ├─ [ATS: placements, pipeline, revenue]
        ├─ [Resume Screening Agent: volumes, scores, pass rates]
        ├─ [Client Outreach Agent: campaigns, reply rates, meetings booked]
        ├─ [Interview Scheduling Agent: interviews scheduled, time-to-schedule]
        ├─ [Compliance Agent: compliance status, alerts generated]
        ├─ [Client Health Agent: portfolio health scores]
        ├─ [Post-Placement Agent: check-in results, satisfaction scores]
        ├─ [Market Intelligence Agent: market conditions summary]
        └─ [Recruiter Productivity Agent: activity metrics per recruiter]
    → [Claude API: Sonnet — Generate narrative report with recommendations]
    → [Generate formatted report (HTML + PDF)]
    → [Email to leadership team]
    → [Post summary to #agency-performance Slack channel]

[MONTHLY EXECUTIVE REPORT:]
    [Trigger: First Monday of month]
    → [Full month analysis including YoY comparison]
    → [Claude API: Sonnet — Deep analysis with strategic recommendations]
    → [Generate executive PDF report]
    → [Schedule delivery to agency leadership]

[AD-HOC QUERIES:]
    [Trigger: Slack slash command "/analytics {question}"]
    → [Claude API: Sonnet — Interpret question, query database, generate answer]
    → [Respond in Slack thread with data visualization (if applicable)]
```

#### Tools & API Integrations
- **Claude API**: Sonnet for narrative generation and trend analysis
- **PostgreSQL**: Time-series data storage
- **Metabase or Grafana**: Dashboard visualization (open-source)
- **Google Sheets API**: For lightweight reporting needs
- **Slack**: Report delivery and ad-hoc queries
- **Email**: Formatted report delivery
- **PDF generation**: Executive reports

#### Estimated Cost
- Weekly report generation: $0.05–$0.15 (Claude Sonnet)
- Monthly (4 weekly + 1 monthly + daily aggregations): $0.50–$1.50/month
- Ad-hoc queries: $0.01–$0.03 each

#### Inter-Agent Connections
- **Receives from**: ALL other agents — this is the data aggregation layer
- **Sends to**: All agents (benchmarks and context), leadership (reports), Recruiter Productivity Agent (performance context)

---

## 5. n8n Workflow Designs

### 5.1 Workflow Inventory

| # | Workflow Name | Trigger | Frequency | Agents Involved | Complexity |
|---|--------------|---------|-----------|----------------|------------|
| 1 | Resume Screening Pipeline | Webhook (new application) | Real-time | Resume Screening | Medium |
| 2 | Candidate Sourcing Engine | New job order + Daily cron | Real-time + Daily | Candidate Sourcing, Resume Screening | High |
| 3 | Client Outreach Campaign | New prospect list + Reply webhook | Real-time | Client Outreach | High |
| 4 | Interview Coordinator | Stage change in ATS | Real-time | Interview Scheduling | Medium |
| 5 | Compliance Daily Scan | Daily cron 6:00 AM | Daily | Compliance Monitoring | Medium |
| 6 | EEOC Quarterly Audit | Quarterly cron | Quarterly | Compliance Monitoring | Low |
| 7 | Market Intelligence Weekly | Weekly cron Monday 6:00 AM | Weekly | Market Intelligence | High |
| 8 | Client Health Scoring | Weekly cron Monday 7:00 AM | Weekly | Client Health | Medium |
| 9 | Recruiter Daily Briefing | Daily cron 7:00 AM | Daily | Recruiter Productivity (+ all agents) | High |
| 10 | Post-Placement Check-in Runner | Daily cron 9:00 AM | Daily | Post-Placement | Medium |
| 11 | Post-Placement Reply Handler | Webhook (email received) | Real-time | Post-Placement | Medium |
| 12 | Analytics Daily Aggregation | Daily cron 6:00 AM | Daily | Analytics | Low |
| 13 | Analytics Weekly Report | Weekly cron Sunday 8:00 PM | Weekly | Analytics | Medium |
| 14 | Bullhorn Sync | Webhook + Polling (5 min) | Near-real-time | Integration | Medium |
| 15 | Lever Sync | Webhook | Real-time | Integration | Medium |
| 16 | Greenhouse Sync | Webhook (built-in node) | Real-time | Integration | Low |
| 17 | JobAdder Sync | Webhook + Polling | Near-real-time | Integration | Medium |
| 18 | ATS Health Check | Cron every 15 min | Recurring | Integration | Low |
| 19 | Cost Monitor | Daily cron 11:00 PM | Daily | Utility | Low |
| 20 | Data Backup | Daily cron 2:00 AM | Daily | Utility | Low |

### 5.2 Workflow Design Patterns

**Pattern A: Real-Time Event Processing**
Used by: Resume Screening, Interview Scheduling, Reply Handlers
```
Webhook Trigger → Validate Payload → Fetch Context from ATS →
AI Processing (Claude) → Update ATS → Notify (Slack/Email) → Log
```

**Pattern B: Scheduled Batch Processing**
Used by: Compliance Scan, Market Intelligence, Client Health, Daily Briefing
```
Cron Trigger → Fetch All Records → Loop/Parallel Process →
AI Analysis (Claude Batch API) → Store Results → Generate Alerts →
Distribute Reports
```

**Pattern C: Multi-Source Aggregation**
Used by: Candidate Sourcing, Market Intelligence, Analytics
```
Trigger → Parallel Source Queries (3-5 branches) → Merge Results →
Deduplicate → AI Scoring/Analysis → Store → Distribute
```

**Pattern D: Sequence Management**
Used by: Client Outreach, Post-Placement Check-ins
```
Trigger → Generate Sequence → Send Step 1 → Wait → Check for Reply →
If Reply: Handle → If No Reply: Send Step 2 → Wait → ... → End Sequence
```

### 5.3 Shared Workflow Components

**Claude API Call (Reusable Sub-Workflow):**
```json
{
  "nodes": [
    {
      "name": "Prepare Claude Request",
      "type": "n8n-nodes-base.function",
      "parameters": {
        "functionCode": "// Select model based on task complexity\nconst model = $json.use_sonnet ? 'claude-sonnet-4-6' : 'claude-haiku-4-5-20251001';\n\n// Build messages array\nconst messages = [{\n  role: 'user',\n  content: $json.prompt\n}];\n\n// Add system prompt if provided\nconst system = $json.system_prompt || '';\n\nreturn {\n  model,\n  max_tokens: $json.max_tokens || 4096,\n  system,\n  messages\n};"
      }
    },
    {
      "name": "Claude API",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "POST",
        "url": "https://api.anthropic.com/v1/messages",
        "headers": {
          "x-api-key": "={{$env.ANTHROPIC_API_KEY}}",
          "anthropic-version": "2023-06-01",
          "content-type": "application/json"
        },
        "body": "={{JSON.stringify($json)}}"
      }
    },
    {
      "name": "Parse Response",
      "type": "n8n-nodes-base.function",
      "parameters": {
        "functionCode": "const response = $json;\nconst content = response.content[0].text;\nconst usage = response.usage;\n\n// Track cost\nconst inputCost = usage.input_tokens * (response.model.includes('haiku') ? 0.000001 : 0.000003);\nconst outputCost = usage.output_tokens * (response.model.includes('haiku') ? 0.000005 : 0.000015);\n\nreturn {\n  content: JSON.parse(content),\n  tokens: usage,\n  cost_usd: inputCost + outputCost\n};"
      }
    }
  ]
}
```

**ATS Adapter Call (Reusable Sub-Workflow):**
```json
{
  "description": "Universal ATS adapter that routes to the correct ATS API based on configuration",
  "parameters": {
    "ats_type": "bullhorn | lever | greenhouse | jobadder",
    "operation": "get_candidate | update_candidate | get_job | create_submission | ...",
    "data": {}
  },
  "implementation": "Switch node routes to ATS-specific HTTP Request nodes with proper auth"
}
```

### 5.4 Error Handling Pattern

Every workflow includes:
```
[Main Flow] → [On Error:]
    → [Log error to database with workflow_id, node_name, error_message]
    → [IF critical workflow (screening, compliance):]
        → [Slack alert to #engineering channel]
        → [Retry with exponential backoff (max 3)]
    → [IF non-critical:]
        → [Add to daily error digest]
    → [IF Claude API rate limit:]
        → [Queue for Batch API processing]
        → [Retry in 5 minutes]
```

---

## 6. ATS Integration Points

### 6.1 Integration Architecture

```
┌─────────────────────────────────────────────┐
│              ATS ADAPTER LAYER              │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │         Base Adapter (Abstract)      │    │
│  │  ─────────────────────────────────  │    │
│  │  + getCandidate(id)                 │    │
│  │  + searchCandidates(query)          │    │
│  │  + createCandidate(data)            │    │
│  │  + updateCandidate(id, data)        │    │
│  │  + getJobOrder(id)                  │    │
│  │  + searchJobOrders(query)           │    │
│  │  + createSubmission(candidateId,    │    │
│  │       jobId, data)                  │    │
│  │  + getPlacement(id)                 │    │
│  │  + createPlacement(data)            │    │
│  │  + addNote(entityType, id, note)    │    │
│  │  + getActivities(entityId, type)    │    │
│  │  + webhook(event) → normalize       │    │
│  └───────────┬─────────────────────────┘    │
│              │ implements                    │
│  ┌───────────┼─────────────────────────┐    │
│  │     │     │     │                   │    │
│  ▼     ▼     ▼     ▼                   │    │
│ Bull  Lever Green JobAdd  Generic      │    │
│ horn         house  er    Webhook      │    │
└─────────────────────────────────────────────┘
```

### 6.2 Bullhorn Integration

**Auth:** OAuth 2.0 → Access Token → REST Token + Base URL

```javascript
// Authentication flow
async function authenticateBullhorn() {
  // Step 1: Get authorization code
  const authUrl = `https://auth.bullhornstaffing.com/oauth/authorize?client_id=${CLIENT_ID}&response_type=code&redirect_uri=${REDIRECT_URI}`;

  // Step 2: Exchange code for access token
  const tokenResponse = await fetch('https://auth.bullhornstaffing.com/oauth/token', {
    method: 'POST',
    body: `grant_type=authorization_code&code=${code}&client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}`
  });
  const { access_token, refresh_token } = await tokenResponse.json();

  // Step 3: Get REST token and base URL
  const loginResponse = await fetch(`https://rest.bullhornstaffing.com/rest-services/login?version=*&access_token=${access_token}`);
  const { BhRestToken, restUrl } = await loginResponse.json();

  return { BhRestToken, restUrl };
}
```

**Key Endpoints:**

| Operation | Endpoint | Method | Notes |
|-----------|----------|--------|-------|
| Get Candidate | `{restUrl}/entity/Candidate/{id}?fields=*` | GET | Use `fields` parameter to limit response |
| Search Candidates | `{restUrl}/search/Candidate?query={lucene_query}` | GET | Lucene query syntax |
| Create Candidate | `{restUrl}/entity/Candidate` | PUT | Returns new entity ID |
| Update Candidate | `{restUrl}/entity/Candidate/{id}` | POST | Partial update supported |
| Get Job Order | `{restUrl}/entity/JobOrder/{id}?fields=*` | GET | |
| Search Jobs | `{restUrl}/search/JobOrder?query={query}` | GET | |
| Create Submission | `{restUrl}/entity/JobSubmission` | PUT | Links candidate to job |
| Create Placement | `{restUrl}/entity/Placement` | PUT | |
| Add Note | `{restUrl}/entity/Note` | PUT | Associate via `personReference` |
| Get Activities | `{restUrl}/query/Note?where=personReference.id={id}` | GET | |

**Rate Limits:** Standard: 100K calls/day, 700/minute. Enterprise: 2M calls/day, 1500/minute.

**Field Mappings (Bullhorn → Shared Schema):**
```json
{
  "candidate": {
    "id": "id",
    "firstName": "first_name",
    "lastName": "last_name",
    "email": "email",
    "phone": "phone",
    "status": "status",
    "source": "source",
    "skillList": "skills",
    "customText1": "custom_field_1",
    "dateAdded": "created_at"
  },
  "jobOrder": {
    "id": "id",
    "title": "title",
    "clientCorporation.name": "client_name",
    "employmentType": "employment_type",
    "payRate": "pay_rate",
    "clientBillRate": "bill_rate",
    "skillList": "required_skills",
    "description": "description"
  }
}
```

### 6.3 Lever Integration

**Auth:** API Key (Basic Auth header)

```javascript
const headers = {
  'Authorization': `Basic ${Buffer.from(LEVER_API_KEY + ':').toString('base64')}`,
  'Content-Type': 'application/json'
};
```

**Key Endpoints:**

| Operation | Endpoint | Method |
|-----------|----------|--------|
| List Opportunities | `https://api.lever.co/v1/opportunities` | GET |
| Get Opportunity | `https://api.lever.co/v1/opportunities/{id}` | GET |
| Create Opportunity | `https://api.lever.co/v1/opportunities` | POST |
| List Postings | `https://api.lever.co/v1/postings` | GET |
| Add Note | `https://api.lever.co/v1/opportunities/{id}/notes` | POST |
| Move Stage | `https://api.lever.co/v1/opportunities/{id}/stage` | PUT |
| List Candidates | `https://api.lever.co/v1/candidates` | GET |

**Webhook Events:** `candidateCreated`, `candidateStageChange`, `candidateArchived`, `interviewCreated`

### 6.4 Greenhouse Integration

**Auth:** Basic Auth (API token as username, empty password)

```javascript
const headers = {
  'Authorization': `Basic ${Buffer.from(GREENHOUSE_API_KEY + ':').toString('base64')}`,
  'On-Behalf-Of': USER_ID  // Required for some endpoints
};
```

**Key Endpoints (Harvest API):**

| Operation | Endpoint | Method |
|-----------|----------|--------|
| List Candidates | `https://harvest.greenhouse.io/v1/candidates` | GET |
| Get Candidate | `https://harvest.greenhouse.io/v1/candidates/{id}` | GET |
| List Applications | `https://harvest.greenhouse.io/v1/applications` | GET |
| List Jobs | `https://harvest.greenhouse.io/v1/jobs` | GET |
| Move Application | `https://harvest.greenhouse.io/v1/applications/{id}/move` | POST |
| Add Note | `https://harvest.greenhouse.io/v1/candidates/{id}/activity_feed/notes` | POST |

**n8n:** Built-in Greenhouse node available with trigger support for `candidateCreated` and `candidateStageChange`.

**Rate Limits:** 50 requests per 10 seconds. Response headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`.

### 6.5 JobAdder Integration

**Auth:** OAuth 2.0 (access token expires in 60 minutes; use refresh token)

```javascript
// Token refresh
const refreshResponse = await fetch('https://id.jobadder.com/connect/token', {
  method: 'POST',
  body: new URLSearchParams({
    grant_type: 'refresh_token',
    refresh_token: REFRESH_TOKEN,
    client_id: CLIENT_ID,
    client_secret: CLIENT_SECRET
  })
});
```

**Key Endpoints:**

| Operation | Endpoint | Method |
|-----------|----------|--------|
| List Candidates | `https://api.jobadder.com/v2/candidates` | GET |
| Create Candidate | `https://api.jobadder.com/v2/candidates` | POST |
| Update Candidate | `https://api.jobadder.com/v2/candidates/{id}` | PUT |
| List Jobs | `https://api.jobadder.com/v2/jobs` | GET |
| Create Application | `https://api.jobadder.com/v2/jobs/{id}/applications` | POST |

### 6.6 Generic Webhook Adapter

For ATS systems not explicitly supported, a generic webhook adapter normalizes incoming data:

```javascript
// Generic webhook handler
function normalizeWebhookPayload(atsType, event, payload) {
  const normalizer = normalizers[atsType] || defaultNormalizer;
  return {
    event_type: normalizer.mapEventType(event),
    entity_type: normalizer.mapEntityType(payload),
    entity_id: normalizer.extractId(payload),
    data: normalizer.mapFields(payload),
    raw: payload,
    received_at: new Date().toISOString()
  };
}
```

### 6.7 Common Integrations Beyond ATS

| System | Integration Method | Purpose |
|--------|-------------------|---------|
| **Google Calendar** | OAuth 2.0 REST API | Interview scheduling, availability checking |
| **Gmail** | OAuth 2.0 REST API | Email sending, reply monitoring |
| **Outlook/O365** | Microsoft Graph API | Email + calendar (alternative to Google) |
| **Slack** | Bot Token + Webhooks | Notifications, daily briefings, slash commands |
| **Calendly** | API Key + Webhooks | Scheduling links, booking events |
| **Zoom** | JWT/OAuth + REST API | Meeting link creation |
| **Instantly.ai** | REST API | Cold email campaigns |
| **Smartlead** | REST API | Cold email campaigns (alternative) |
| **Checkr** | REST API + Webhooks | Background checks |
| **LinkedIn** | Via Apify actors | Candidate sourcing, company research |
| **Indeed** | Publisher API | Job posting, resume search |
| **Google Sheets** | Service Account API | Lightweight data storage, reporting |
| **Twilio** | REST API | SMS notifications for candidates |
| **DocuSign** | REST API | Offer letter and contract signing |
| **Stripe/QuickBooks** | REST API | Invoice and payment tracking |

---

## 7. Data Model & Shared Schema

### Core Entities

```sql
-- Candidates
CREATE TABLE candidates (
  id UUID PRIMARY KEY,
  ats_id VARCHAR(255),
  ats_source VARCHAR(50), -- bullhorn, lever, greenhouse, jobadder
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  email VARCHAR(255),
  phone VARCHAR(50),
  location_city VARCHAR(100),
  location_state VARCHAR(50),
  location_country VARCHAR(50),
  current_title VARCHAR(200),
  years_experience DECIMAL(4,1),
  skills JSONB, -- ["Python", "AWS", "Docker"]
  certifications JSONB,
  education JSONB,
  work_history JSONB,
  source VARCHAR(100), -- linkedin, indeed, referral, ats_database
  screening_score DECIMAL(5,2),
  screening_data JSONB, -- Full screening agent output
  compliance_status VARCHAR(50),
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);

-- Job Orders
CREATE TABLE job_orders (
  id UUID PRIMARY KEY,
  ats_id VARCHAR(255),
  ats_source VARCHAR(50),
  title VARCHAR(200),
  client_id UUID REFERENCES clients(id),
  employment_type VARCHAR(50), -- contract, perm, contract-to-hire
  required_skills JSONB,
  preferred_skills JSONB,
  min_experience_years DECIMAL(4,1),
  education_requirements JSONB,
  certifications_required JSONB,
  location VARCHAR(200),
  remote_eligible BOOLEAN,
  pay_rate_min DECIMAL(10,2),
  pay_rate_max DECIMAL(10,2),
  bill_rate DECIMAL(10,2),
  description TEXT,
  status VARCHAR(50), -- open, filled, on_hold, closed
  recruiter_id UUID,
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);

-- Clients
CREATE TABLE clients (
  id UUID PRIMARY KEY,
  ats_id VARCHAR(255),
  ats_source VARCHAR(50),
  company_name VARCHAR(200),
  industry VARCHAR(100),
  size_range VARCHAR(50), -- 1-50, 51-200, 201-1000, 1000+
  primary_contact_name VARCHAR(200),
  primary_contact_email VARCHAR(255),
  primary_contact_title VARCHAR(200),
  health_score DECIMAL(5,2),
  health_trend VARCHAR(20), -- IMPROVING, STABLE, DECLINING
  lifetime_revenue DECIMAL(12,2),
  annual_revenue DECIMAL(12,2),
  last_order_date DATE,
  total_placements INTEGER,
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);

-- Placements
CREATE TABLE placements (
  id UUID PRIMARY KEY,
  ats_id VARCHAR(255),
  candidate_id UUID REFERENCES candidates(id),
  job_order_id UUID REFERENCES job_orders(id),
  client_id UUID REFERENCES clients(id),
  recruiter_id UUID,
  start_date DATE,
  end_date DATE,
  pay_rate DECIMAL(10,2),
  bill_rate DECIMAL(10,2),
  margin_pct DECIMAL(5,2),
  employment_type VARCHAR(50),
  status VARCHAR(50), -- active, completed, terminated, extended
  compliance_status VARCHAR(50),
  check_in_schedule JSONB,
  satisfaction_score DECIMAL(5,2),
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);

-- Agent Activity Log
CREATE TABLE agent_activity_log (
  id UUID PRIMARY KEY,
  agent_name VARCHAR(100),
  workflow_id VARCHAR(255),
  action_type VARCHAR(100),
  entity_type VARCHAR(50),
  entity_id UUID,
  input_summary JSONB,
  output_summary JSONB,
  tokens_used INTEGER,
  cost_usd DECIMAL(10,6),
  duration_ms INTEGER,
  status VARCHAR(50), -- success, error, partial
  error_message TEXT,
  created_at TIMESTAMP
);

-- Compliance Records
CREATE TABLE compliance_records (
  id UUID PRIMARY KEY,
  candidate_id UUID REFERENCES candidates(id),
  placement_id UUID REFERENCES placements(id),
  record_type VARCHAR(100), -- certification, license, i9, background_check, drug_test
  document_name VARCHAR(200),
  issuing_authority VARCHAR(200),
  issue_date DATE,
  expiration_date DATE,
  status VARCHAR(50), -- current, expiring_soon, expired, pending
  state VARCHAR(50),
  notes TEXT,
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);

-- Client Health Scores (time series)
CREATE TABLE client_health_history (
  id UUID PRIMARY KEY,
  client_id UUID REFERENCES clients(id),
  score_date DATE,
  overall_score DECIMAL(5,2),
  placement_velocity DECIMAL(5,2),
  fill_rate DECIMAL(5,2),
  communication_engagement DECIMAL(5,2),
  payment_health DECIMAL(5,2),
  order_pipeline DECIMAL(5,2),
  satisfaction_signals DECIMAL(5,2),
  risk_level VARCHAR(20),
  created_at TIMESTAMP
);

-- Outreach Campaigns
CREATE TABLE outreach_campaigns (
  id UUID PRIMARY KEY,
  prospect_company VARCHAR(200),
  prospect_name VARCHAR(200),
  prospect_email VARCHAR(255),
  prospect_title VARCHAR(200),
  sequence JSONB, -- Array of email steps
  current_step INTEGER,
  status VARCHAR(50), -- active, replied, bounced, completed, suppressed
  reply_sentiment VARCHAR(50),
  meeting_booked BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);
```

---

## 8. Deployment Guide

### 8.1 Prerequisites

- **Docker** and **Docker Compose** installed
- **Claude API key** from [console.anthropic.com](https://console.anthropic.com)
- **ATS API credentials** for your ATS (at least one)
- **Google Workspace** or **Microsoft 365** account (for calendar + email)
- A **server or VPS** with at least 2GB RAM, 2 CPU cores, 20GB storage (a $20/month DigitalOcean droplet works)

### 8.2 Quick Start (Docker Compose)

**Step 1: Clone and configure**
```bash
git clone https://github.com/usingaitoscale/staffing-ai-automation.git
cd staffing-ai-automation
cp .env.example .env
```

**Step 2: Edit .env with your credentials**
```env
# === REQUIRED ===
ANTHROPIC_API_KEY=sk-ant-xxxxx

# === ATS (configure at least one) ===
# Bullhorn
BULLHORN_CLIENT_ID=
BULLHORN_CLIENT_SECRET=
BULLHORN_USERNAME=
BULLHORN_PASSWORD=

# Lever
LEVER_API_KEY=

# Greenhouse
GREENHOUSE_API_KEY=

# JobAdder
JOBADDER_CLIENT_ID=
JOBADDER_CLIENT_SECRET=
JOBADDER_REFRESH_TOKEN=

# === EMAIL (for outreach + notifications) ===
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=
SMTP_PASS=

# Or use Instantly
INSTANTLY_API_KEY=

# === CALENDAR ===
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REFRESH_TOKEN=

# === NOTIFICATIONS ===
SLACK_BOT_TOKEN=xoxb-xxxxx
SLACK_CHANNEL_ALERTS=#staffing-alerts
SLACK_CHANNEL_REPORTS=#staffing-reports

# === DATABASE ===
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
POSTGRES_DB=staffing_ai
POSTGRES_USER=staffing
POSTGRES_PASSWORD=change_me_please

# === n8n ===
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=change_me_please
N8N_ENCRYPTION_KEY=generate_a_random_key_here
WEBHOOK_URL=https://your-domain.com/webhook

# === OPTIONAL: Scraping ===
APIFY_API_KEY=
```

**Step 3: Launch**
```bash
docker-compose up -d
```

**Step 4: Import workflows**
```bash
make import-workflows
# Or manually:
# ./scripts/import-workflows.sh
```

**Step 5: Verify**
```bash
make health-check
# Opens n8n at http://localhost:5678
# Verifies all services are running
# Tests ATS connectivity
```

### 8.3 docker-compose.yml

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD}
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - WEBHOOK_URL=${WEBHOOK_URL}
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - BULLHORN_CLIENT_ID=${BULLHORN_CLIENT_ID}
      - BULLHORN_CLIENT_SECRET=${BULLHORN_CLIENT_SECRET}
      - LEVER_API_KEY=${LEVER_API_KEY}
      - GREENHOUSE_API_KEY=${GREENHOUSE_API_KEY}
      - SLACK_BOT_TOKEN=${SLACK_BOT_TOKEN}
      - INSTANTLY_API_KEY=${INSTANTLY_API_KEY}
      - APIFY_API_KEY=${APIFY_API_KEY}
    volumes:
      - n8n_data:/home/node/.n8n
      - ./workflows:/home/node/workflows
      - ./agents:/home/node/agents
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:15
    restart: always
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    restart: always
    volumes:
      - redis_data:/data

  metabase:
    image: metabase/metabase:latest
    restart: always
    ports:
      - "3000:3000"
    environment:
      - MB_DB_TYPE=postgres
      - MB_DB_DBNAME=${POSTGRES_DB}
      - MB_DB_PORT=5432
      - MB_DB_USER=${POSTGRES_USER}
      - MB_DB_PASS=${POSTGRES_PASSWORD}
      - MB_DB_HOST=postgres
    depends_on:
      - postgres

volumes:
  n8n_data:
  postgres_data:
  redis_data:
```

### 8.4 Phased Deployment Recommendation

**Phase 1 (Week 1): Foundation**
- Deploy n8n + PostgreSQL + Redis
- Configure ATS adapter for your primary ATS
- Deploy Resume Screening Agent (highest immediate impact)
- Deploy Compliance Monitoring Agent (highest risk reduction)

**Phase 2 (Week 2-3): Core Workflows**
- Deploy Candidate Sourcing Agent
- Deploy Interview Scheduling Agent
- Deploy Recruiter Daily Briefing
- Connect calendar and email integrations

**Phase 3 (Week 4-5): Business Intelligence**
- Deploy Client Health Agent
- Deploy Market Intelligence Agent
- Deploy Analytics & Reporting Agent
- Set up Metabase dashboards

**Phase 4 (Week 6+): Outreach & Optimization**
- Deploy Client Outreach Agent
- Deploy Post-Placement Agent
- Fine-tune scoring rubrics based on placement outcomes
- Optimize Claude model selection based on cost data

### 8.5 Production Considerations

**SSL/HTTPS:** Use Caddy or Nginx reverse proxy with Let's Encrypt
```yaml
# Add to docker-compose.yml
  caddy:
    image: caddy:2
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
```

**Backups:** Daily automated PostgreSQL backups
```bash
# scripts/backup.sh runs daily via cron
pg_dump -h postgres -U ${POSTGRES_USER} ${POSTGRES_DB} | gzip > /backups/staffing_ai_$(date +%Y%m%d).sql.gz
```

**Monitoring:** Use n8n's built-in execution log + custom cost tracking workflow

**Scaling:** For agencies processing >1000 candidates/month, consider:
- Separate n8n worker instances for heavy workflows
- Redis queue for Claude API calls (rate limit management)
- Read replicas for PostgreSQL (reporting queries)

---

## 9. Cost Analysis

### 9.1 Monthly Cost Breakdown (Mid-Size Agency: 10-50 placements/month)

| Component | Service | Monthly Cost | Notes |
|-----------|---------|-------------|-------|
| **AI Processing** | Claude API (Haiku + Sonnet mix) | $25–$75 | 90% Haiku, 10% Sonnet. Batch API for non-urgent. |
| **Workflow Engine** | n8n Self-Hosted | $10–$20 | VPS hosting cost only |
| **Database** | PostgreSQL (on same VPS) | $0 | Included in VPS |
| **Caching** | Redis (on same VPS) | $0 | Included in VPS |
| **Dashboards** | Metabase (on same VPS) | $0 | Open-source, self-hosted |
| **Email Outreach** | Instantly.ai | $97–$297 | Growth or Hypergrowth plan |
| **Scraping** | Apify | $49–$99 | For LinkedIn + job board data |
| **Calendar** | Google Workspace | $0–$7/user | May already have this |
| **Notifications** | Slack | $0–$8.75/user | Free tier often sufficient |
| **VPS Hosting** | DigitalOcean / Hetzner | $20–$48 | 4GB RAM, 2 vCPU recommended |
| **Domain + SSL** | Cloudflare + Let's Encrypt | $0–$10 | For webhook URL |
| **Background Checks** | Checkr / GoodHire | $35–$50/check | Per placement, not monthly |
| **TOTAL** | | **$200–$560/month** | Excluding per-placement costs |

### 9.2 Claude API Cost Detail

| Agent | Model | Calls/Month | Avg Tokens/Call | Monthly Cost |
|-------|-------|-------------|----------------|-------------|
| Resume Screening | Haiku | 500 | 2,000 | $1.50–$4.00 |
| Candidate Sourcing | Haiku + Sonnet | 200 | 3,000 | $3.00–$8.00 |
| Client Outreach | Sonnet | 500 | 1,500 | $4.00–$12.00 |
| Interview Scheduling | Haiku | 100 | 1,000 | $0.10–$0.25 |
| Compliance Monitoring | Haiku | 100 | 1,500 | $0.20–$0.50 |
| Market Intelligence | Sonnet | 20 | 5,000 | $1.00–$3.00 |
| Client Health | Sonnet | 50 | 3,000 | $0.80–$2.00 |
| Recruiter Productivity | Sonnet | 110 | 2,000 | $2.00–$5.00 |
| Post-Placement | Haiku | 150 | 1,000 | $0.30–$0.80 |
| Analytics | Sonnet | 30 | 4,000 | $0.80–$2.00 |
| **TOTAL** | | **~1,760** | | **$13.70–$37.55** |

**Cost Optimization Levers:**
- **Batch API**: 50% discount for non-urgent processing (screening overnight, weekly reports)
- **Prompt Caching**: 90% discount on cached system prompts (all agents use static system prompts)
- **Haiku First**: Use Haiku for everything except where quality noticeably suffers, then upgrade to Sonnet
- Combined savings potential: **60–70% reduction** from naive implementation

### 9.3 ROI Analysis

| Metric | Before Automation | After Automation | Impact |
|--------|------------------|-----------------|--------|
| Recruiter screening time | 15 hrs/week | 1.5 hrs/week | 13.5 hrs saved/week |
| Time-to-fill | 44 days | 28 days | 36% faster |
| Candidate submissions/week | 12 | 24 | 2x throughput |
| Client response rate (outreach) | 3% | 8% | 2.7x improvement |
| Compliance incidents/quarter | 3-5 | 0-1 | ~90% reduction |
| Client churn rate | 25% | 15% | 40% reduction |
| Recruiter capacity (placements/month) | 4-6 | 8-12 | 2x capacity |

**Revenue Impact (10-person agency):**
- 5 recruiters × 4 additional placements/month × $5,000 avg. fee = **$100,000/month additional revenue**
- System cost: **$500/month**
- **ROI: 200:1**

---

## 10. Security & Compliance

### 10.1 Data Handling Principles

1. **PII Minimization**: Only send necessary candidate data to Claude API. Strip SSN, DOB, and other sensitive fields before API calls.
2. **Ephemeral Processing**: Claude API does not retain data from API calls. No training on your data.
3. **Encryption at Rest**: PostgreSQL with encrypted storage. All backups encrypted.
4. **Encryption in Transit**: All API calls over HTTPS/TLS 1.3.
5. **Access Control**: n8n basic auth + role-based access for dashboards.

### 10.2 PII Handling in Claude Calls

```javascript
// agents/shared/utils/pii-handler.js
function sanitizeForClaude(candidateData) {
  const sanitized = { ...candidateData };

  // Remove sensitive PII that Claude doesn't need
  delete sanitized.ssn;
  delete sanitized.date_of_birth;
  delete sanitized.driver_license_number;
  delete sanitized.bank_account;

  // Mask partial data
  if (sanitized.email) {
    sanitized.email = sanitized.email.replace(/(.{2})(.*)(@.*)/, '$1***$3');
  }

  return sanitized;
}
```

### 10.3 GDPR / Data Privacy Considerations

- **Right to Deletion**: Implement cascade delete across all tables when candidate requests removal
- **Data Retention**: Configure auto-purge for inactive candidates (default: 2 years)
- **Consent Tracking**: Track processing basis for each candidate record
- **Data Export**: Provide candidate data export in JSON format on request

### 10.4 Bias Mitigation

Every Resume Screening Agent call includes a separate bias check:
- Names, ages, and demographic indicators are not factors in scoring
- Quarterly EEOC adverse impact analysis via Compliance Agent
- All scoring rubrics documented and auditable
- Ability to A/B test AI scoring against human scoring for calibration

---

## 11. Roadmap

### v1.0 — Launch (March 2026)
- 10 agent architectures with prompts and configurations
- n8n workflow JSON exports for all 20 workflows
- ATS adapters for Bullhorn, Lever, Greenhouse, JobAdder
- Docker Compose one-command deployment
- Complete documentation

### v1.1 — Community Feedback (April 2026)
- Bug fixes and workflow refinements based on community feedback
- Additional email sequence templates by industry vertical
- Improved scoring rubric customization UI
- Video walkthrough series

### v1.2 — Enhanced Intelligence (May 2026)
- Claude tool use for multi-step reasoning in complex agents
- Agent-to-agent communication protocol (structured message passing)
- Historical performance learning (which scoring patterns predict successful placements)
- Slack bot interface for ad-hoc queries to any agent

### v2.0 — Platform Features (Q3 2026)
- Web-based admin dashboard (React) for configuration without code
- Multi-tenant support for agencies managing multiple brands
- VMS integration (Beeline, SAP Fieldglass)
- Advanced analytics with predictive revenue forecasting
- Mobile app for recruiter daily briefing

### v2.5 — AI Native (Q4 2026)
- Fine-tuned models for staffing-specific tasks
- Voice AI for candidate phone screening
- Computer use for ATS automation (for ATS systems without APIs)
- Real-time market pricing engine

---

## Appendix A: Environment Variable Reference

| Variable | Required | Description |
|----------|----------|-------------|
| `ANTHROPIC_API_KEY` | Yes | Claude API key |
| `BULLHORN_CLIENT_ID` | If using Bullhorn | Bullhorn OAuth client ID |
| `BULLHORN_CLIENT_SECRET` | If using Bullhorn | Bullhorn OAuth client secret |
| `LEVER_API_KEY` | If using Lever | Lever API key |
| `GREENHOUSE_API_KEY` | If using Greenhouse | Greenhouse Harvest API key |
| `JOBADDER_CLIENT_ID` | If using JobAdder | JobAdder OAuth client ID |
| `JOBADDER_CLIENT_SECRET` | If using JobAdder | JobAdder OAuth client secret |
| `SMTP_HOST` | Yes | Email server hostname |
| `SMTP_PORT` | Yes | Email server port |
| `SMTP_USER` | Yes | Email username |
| `SMTP_PASS` | Yes | Email password or app password |
| `INSTANTLY_API_KEY` | If using Instantly | For cold email campaigns |
| `GOOGLE_CLIENT_ID` | If using Google | For Calendar + Gmail |
| `GOOGLE_CLIENT_SECRET` | If using Google | For Calendar + Gmail |
| `SLACK_BOT_TOKEN` | Recommended | For notifications |
| `APIFY_API_KEY` | If using scraping | For LinkedIn/job board data |
| `POSTGRES_PASSWORD` | Yes | Database password |
| `N8N_BASIC_AUTH_PASSWORD` | Yes | n8n admin password |
| `N8N_ENCRYPTION_KEY` | Yes | n8n credential encryption |
| `WEBHOOK_URL` | Yes | Public URL for webhooks |

## Appendix B: Glossary

| Term | Definition |
|------|-----------|
| **ATS** | Applicant Tracking System — the core database for candidates, jobs, and placements |
| **Bill Rate** | The hourly rate charged to the client |
| **Pay Rate** | The hourly rate paid to the contractor |
| **Spread/Margin** | Bill Rate minus Pay Rate (and burden costs) |
| **Burden Rate** | Employer costs beyond pay: FICA, FUTA, SUTA, workers comp, benefits (typically 22-30%) |
| **Time-to-Fill** | Days from job order creation to candidate start date |
| **Submittal** | Presenting a candidate to the client for consideration |
| **Submission-to-Interview Ratio** | % of submitted candidates who get interviews |
| **Fill Rate** | % of job orders successfully filled |
| **VMS** | Vendor Management System — enterprise software managing contingent labor suppliers |
| **MSP** | Managed Service Provider — outsourced management of an employer's contingent workforce |
| **QBR** | Quarterly Business Review — strategic meeting with client |
| **NPS** | Net Promoter Score — client/candidate satisfaction metric |
| **I-9** | Employment Eligibility Verification form (US) |
| **E-Verify** | Electronic system to verify employment eligibility against government databases |
| **EEOC** | Equal Employment Opportunity Commission |
| **Four-Fifths Rule** | Selection rate for any group must be at least 80% of the highest group's rate |

---

*Built with care by [Using AI to Scale](https://usingaitoscale.com). Contributions welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).*
