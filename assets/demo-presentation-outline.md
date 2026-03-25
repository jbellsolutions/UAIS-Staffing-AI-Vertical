# UAIS Staffing AI Demo — Presentation Outline

## Purpose

A 15-minute live demo showing staffing agency owners exactly what an AI Integrator builds in their first 30 days. This is the presentation used on sales calls after the discovery phase. The goal is to make the prospect feel the gap between their current manual workflows and what AI-powered recruiting looks like — then close.

---

## Slide Flow (15 Minutes)

### Opening (2 min)

**Slide 1: "Your Agency, Powered by AI — What 30 Days Looks Like"**
- Set the frame: this is not a pitch deck. This is a preview of what their agency will look like in one month.
- Personalize with their agency name and logo if available.

**Slide 2: The Problem**
- Display the 13 pain categories from the problem map.
- Highlight the specific ones they mentioned on the discovery call (circle or bold them).
- Key stat: "Your recruiters spend 60% of their day on tasks AI can handle in seconds."
- Transition: "Let me show you what that looks like in practice."

---

### The Demo (8 min)

**Slide 3: Resume Screening Agent — LIVE CLAUDE CODE SESSION**
- Open a live Claude Code session on screen. This is the demo — not an n8n workflow builder.
- Show Claude Code agent processing a stack of 50 resumes in real-time against job requirements.
- Show the direct ATS API connection — no middleware, no drag-and-drop. Real code, real intelligence.
- Output: ranked shortlist with match scores and reasoning for each score, pushed directly to ATS.
- Talking point: "This took your recruiter 6 hours. It just took 45 seconds. And it's connected directly to your Bullhorn — no middleware."
- Show the reasoning output so they see it is not a black box.

**Slide 4: Candidate Sourcing Pipeline — LIVE CLAUDE CODE SESSION**
- Show Claude Code building and running a multi-source search simultaneously (LinkedIn, GitHub, job boards).
- Claude Code enriches profiles with skills, experience, and contact information via direct API calls.
- AI scores relevance against the open role with complex reasoning — not keyword matching.
- Output: 25 qualified candidates with contact info, ready for outreach.
- Talking point: "Your recruiter found 3 candidates in a day. Claude Code found 25 in 10 minutes. This isn't a drag-and-drop workflow — this is AI that thinks."
- Emphasize quality: show the match reasoning, not just the count.

**Slide 5: Cold Outreach Personalization — LIVE CLAUDE CODE SESSION**
- Show Claude Code researching a prospect company in real-time.
- Claude Code pulls recent job postings, company news, leadership changes, and funding rounds.
- Claude Code crafts a personalized email referencing specific details.
- Talking point: "This is the email that got you on this call. Same system, pointed at your prospects."
- Show 2-3 variations so they see it is not a template.

**Slide 6: The Operations Hub — OpenClaw (Tier 2+ Only)**
- Show the OpenClaw autonomous layer: "This runs 24/7 on its own. The AI that never sleeps."
- Show the Slack or SMS interface: type "Follow up with every candidate who hasn't responded in 3 days" and watch it execute.
- Show the daily briefing: a prioritized task list generated for each recruiter every morning.
- Show credential monitoring: automatic expiration alerts, renewal tracking, compliance dashboards.
- Show n8n monitoring dashboard: visual overview of all scheduled jobs, execution logs, system health.
- Talking point: "Claude Code builds it. OpenClaw runs it. n8n schedules it. Your ATS connects it. That's the stack."

---

### The Offer (3 min)

**Slide 7: What You Get**
- Certified AI Integrator matched to your specific agency type — trained on Claude Code, not just drag-and-drop workflows.
- 30-day bootcamp building these exact systems around YOUR workflows.
- Custom Claude Code agents integrated directly with YOUR ATS APIs, YOUR processes, YOUR team — no middleware.
- OpenClaw Operations Hub running 24/7 on dedicated cloud infrastructure (Tier 2+).
- Results: 30-40 hours per week saved, 2x placements per recruiter.
- Not a software subscription. Not an "n8n agency." A trained person who builds Claude Code agents inside your agency.

**Slide 8: The Investment**
- $1,000 to start + $300/month.
- Show the ROI calculation: "Average agency sees $30K+ in additional revenue per recruiter per month. Your investment: $1,300."
- Frame it as a no-brainer: "You are paying $1,300 for a system that generates $30K+ per recruiter."
- Compare to the cost of one bad hire or one lost placement.

**Slide 9: The Guarantee**
- "We agree on metrics BEFORE we start. If we don't double them in 30 days, you get every dollar back."
- "Month-to-month after that. No contracts. We earn your business every month."
- "You keep everything we build, even if you cancel."
- This eliminates all risk from the prospect's perspective.

---

### Close (2 min)

**Slide 10: "Two Options"**
- Option A: Keep doing it the old way. Recruiters spending 60% of their time on manual tasks. Losing placements to faster agencies.
- Option B: Let us place an AI Integrator next week. Systems live in 30 days. Results measured from day one.
- Next step: "We match you with an integrator. They start within 7 days."
- Close: "What questions do you have before we get started?"

---

## Demo Requirements

| Requirement | Details |
|-------------|---------|
| Live Claude Code session | **PRIMARY DEMO** — Resume screening agent running against real (anonymized) resume data. Show the Claude Code environment, not an n8n canvas. |
| Direct ATS API demo | Show Claude Code connecting directly to a Bullhorn/Lever sandbox — no middleware layer |
| Pre-built sourcing pipeline | Claude Code-built candidate sourcing with real output from multiple sources |
| Cold email generation | Claude Code generating personalized outreach live during the call |
| OpenClaw Operations Hub | Slack channel or web dashboard showing the autonomous 24/7 operations layer |
| n8n monitoring dashboard | Visual overview of scheduled jobs and orchestration — positioned as supporting infrastructure, not the main event |
| Case study metrics | Before/after numbers from completed engagements |
| Backup recordings | Pre-recorded demos in case of technical issues during the live call |

---

## Customization by Agency Type

Tailor the demo emphasis based on the prospect's agency model:

| Agency Type | Demo Focus | Key Pain to Hit |
|-------------|-----------|-----------------|
| **Contingency Recruiting** | Speed-to-submit and sourcing volume | "First to submit wins. AI gets you there in minutes, not days." |
| **Healthcare Staffing** | Credential monitoring and compliance tracking | "One expired credential can cost you a contract. AI never forgets a renewal date." |
| **IT Staffing** | Semantic skill matching and technical assessment | "Your recruiters can't tell Java from JavaScript. AI matches on actual skill depth." |
| **Light Industrial** | No-show reduction and shift filling automation | "A 3 AM no-show call used to mean scrambling. Now AI has a replacement confirmed before your phone rings." |
| **Executive Search** | Market mapping and research automation | "AI builds the target company list, maps the org chart, and identifies the 12 people worth calling — in 20 minutes." |

---

## Presenter Notes

- Never read from slides. The slides are visual anchors; you are telling a story.
- Pause after each demo output. Let them absorb the speed and quality.
- Ask "Does that match what your team does today?" after each demo section. Get them nodding.
- If they ask about a feature you have not built yet, say "That's exactly the kind of custom workflow your integrator builds in the first 30 days."
- The close is not aggressive. It is a natural next step. If they are on this call, they are already interested.
- Always end with a specific next step and a date: "I'll match you with an integrator by Thursday. Does that work?"
