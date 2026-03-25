# AI Integrator Solutions by Agency Type

> **Universal AI Systems (UAIS) -- Staffing AI Vertical**
> This index maps each of the 17 staffing agency business models to the specific AI Integrator solutions that would be deployed, including recommended tiers, deployment focus areas, and expected ROI.

---

## How to Use This Guide

Every staffing agency operates under one (or more) of the business models below. The AI Integrator program deploys a tailored automation stack for each model — built with **Claude Code** (custom agents and direct API integrations), powered by **OpenClaw** (autonomous 24/7 operations), orchestrated through **n8n** (scheduling, monitoring, webhooks), and connected to **ATS APIs** (Bullhorn, Lever, Greenhouse, JobAdder) directly with no middleware. Click through to the detailed deployment guide for each type.

---

## 1. Contingency Recruiting

**[Detailed Deployment Guide](contingency-recruiting.md)**

- **Model:** Agency submits candidates to open roles; fee (15-25% of first-year salary) is paid only upon successful hire. No placement, no payment.
- **Revenue:** One-time placement fees. Gross margins 40-70%, net margins 3-8%.
- **Top 5 Pain Points:**
  1. Speed-to-submit -- first agency with qualified candidates wins the placement
  2. Candidate ghosting -- 40%+ of candidates go silent during the process
  3. Resume screening bottleneck -- 7-10 minutes per resume, hundreds per role
  4. Database staleness -- 30-40% of records become outdated within 12 months
  5. Client acquisition -- 53% of agencies say connecting with decision-makers is their top challenge
- **AI Integrator 30-Day Focus:** Resume screening agent (Claude Code agent + direct ATS API integration), candidate sourcing pipeline (Claude Code + Apify + n8n orchestration), automated follow-up sequences, cold outreach engine for business development.
- **Recommended Tier:** Professional ($2,000)
- **Expected ROI:** 13x in month 1. Two additional placements per recruiter per month at $15,000 average fee = $30,000 incremental revenue against $2,300 first-month cost.

---

## 2. Retained Search

- **Model:** Client pays an upfront retainer (typically in three installments) for an exclusive search engagement. Fee is 25-35% of total first-year compensation.
- **Revenue:** Predictable, milestone-based payments regardless of placement outcome. Gross margins 50-70%, net margins 15-25%.
- **Top 5 Pain Points:**
  1. Research depth -- exhaustive market mapping required for every engagement
  2. Long search cycles -- average 90-120 days per engagement ties up resources
  3. Candidate assessment quality -- clients expect psychometric-grade evaluations
  4. Status reporting burden -- weekly or biweekly detailed progress reports to clients
  5. Limited throughput -- each recruiter manages only 3-5 active searches simultaneously
- **AI Integrator 30-Day Focus:** Market mapping automation (Claude Code agents + Apify research synthesis), candidate assessment report generation, automated status report compilation from ATS data, talent pipeline intelligence dashboards.
- **Recommended Tier:** Professional ($2,000)
- **Expected ROI:** 8x in first quarter. Reduced research time by 60% allows each partner to manage 2-3 additional concurrent searches. At $50,000+ average fee, one additional completed search per quarter per partner pays for the system many times over.

---

## 3. Recruitment Process Outsourcing (RPO)

- **Model:** Agency takes over all or part of a client's internal recruiting function. Pricing is monthly retainer plus per-hire fees, with contracts running 2-5 years. The global RPO market exceeds $7B.
- **Revenue:** Recurring monthly retainers ($10K-$100K+/month) plus variable per-hire fees. Gross margins 30-50%, net margins 8-15%.
- **Top 5 Pain Points:**
  1. SLA compliance -- contractual obligations for time-to-fill, quality-of-hire, cost-per-hire
  2. Scalability -- volume fluctuates dramatically with client hiring cycles
  3. Reporting complexity -- multi-client dashboards with dozens of KPIs
  4. Process standardization -- maintaining consistent quality across distributed teams
  5. Technology integration -- must work within each client's existing HR tech stack
- **AI Integrator 30-Day Focus:** SLA monitoring dashboard with automated alerts, bulk resume screening pipeline, standardized workflow templates across client accounts, automated reporting engine that pulls data from multiple ATS platforms.
- **Recommended Tier:** Enterprise ($5,000)
- **Expected ROI:** 10x over first contract year. Automation reduces cost-per-hire by 25-40%, improving margins on fixed-fee arrangements and enabling the RPO to take on additional client accounts without proportional headcount increases.

---

## 4. Temporary/Contract Staffing

- **Model:** Agency employs workers and assigns them to client sites for defined periods. Revenue comes from a markup (25-75%) on the worker's hourly bill rate. This is the largest segment at $150B+ market size.
- **Revenue:** Ongoing bill-rate markup for the duration of each assignment. Gross margins 25-75% on bill rate, but net margins are thin at 3-7% after payroll burden, insurance, and overhead.
- **Top 5 Pain Points:**
  1. Fill rate pressure -- unfilled orders are lost revenue every single day
  2. Compliance and payroll complexity -- W-2, benefits, workers' comp across jurisdictions
  3. Redeployment gaps -- downtime between assignments erodes margins
  4. Timesheet and invoicing errors -- manual processes create billing disputes
  5. Worker retention -- high turnover (300%+ annualized in some segments) drives constant recruiting
- **AI Integrator 30-Day Focus:** Real-time job-to-candidate matching engine, automated compliance document collection and verification, redeployment alert system (flags workers approaching assignment end), timesheet anomaly detection, candidate reactivation campaigns.
- **Recommended Tier:** Professional ($2,000)
- **Expected ROI:** 15x in first quarter. Improving fill rate by even 5% on a $5M annual revenue temp book translates to $250K+ in additional gross profit. Reducing redeployment gaps by 2 days per assignment compounds across every active worker.

---

## 5. Temp-to-Perm

- **Model:** Workers start as temporary employees with the option to convert to permanent hires after a trial period (typically 90-180 days). Agency earns dual revenue: temp markup during the assignment plus a conversion fee upon permanent hire.
- **Revenue:** Temp-period markup (25-50%) plus conversion fee (typically prorated based on hours worked). Combined gross margins 30-60%.
- **Top 5 Pain Points:**
  1. Conversion prediction -- difficulty knowing which placements will convert
  2. Pricing complexity -- calculating conversion fees against hours already billed
  3. Candidate expectation management -- workers uncertain about permanent offer
  4. Client decision delays -- managers postpone conversion decisions, extending temp periods
  5. Dual-pipeline management -- running temp fulfillment and perm placement processes simultaneously
- **AI Integrator 30-Day Focus:** Conversion likelihood scoring model (analyzes manager feedback, assignment duration, performance signals), automated conversion fee calculations and client notifications, candidate engagement sequences during trial period, client nudge workflows when conversion windows approach.
- **Recommended Tier:** Starter ($1,000)
- **Expected ROI:** 6x in first quarter. Increasing conversion rate from 30% to 45% on existing temp placements captures conversion fees that would otherwise be lost. Each conversion fee ($3,000-$8,000) is nearly pure margin.

---

## 6. Direct Hire / Permanent Placement

- **Model:** Agency sources and presents candidates for permanent roles. Fee (15-25% of first-year salary) is a one-time payment upon successful hire. Similar to contingency but may include some exclusivity or project-based arrangements.
- **Revenue:** One-time placement fees. Gross margins 45-65%, net margins 5-12%.
- **Top 5 Pain Points:**
  1. Candidate quality expectations -- clients expect pre-vetted, interview-ready candidates
  2. Offer acceptance rates -- candidates decline offers after lengthy processes
  3. Guarantee period risk -- placements that leave within 60-90 days require free replacement
  4. Job order prioritization -- too many open roles, not enough recruiters
  5. Passive candidate engagement -- best candidates are employed and not actively looking
- **AI Integrator 30-Day Focus:** Candidate pre-qualification bot (Claude-powered screening calls via voice or chat), offer acceptance prediction scoring, guarantee period health monitoring (automated check-ins with placed candidates), job order prioritization algorithm, passive candidate nurture campaigns.
- **Recommended Tier:** Starter ($1,000)
- **Expected ROI:** 8x in first quarter. Reducing falloffs during the offer stage and minimizing guarantee-period replacements directly protects revenue. Each saved placement is worth $12,000-$20,000.

---

## 7. Executive Search

- **Model:** High-touch, relationship-driven search for C-suite and senior leadership positions. Fee is 25-35% of total compensation (base + bonus + equity), often $75,000-$250,000+ per engagement. Most engagements are retained or partially retained.
- **Revenue:** Large per-engagement fees with milestone payments. Gross margins 60-80%, net margins 20-35%.
- **Top 5 Pain Points:**
  1. Market intelligence depth -- boards expect exhaustive knowledge of the leadership landscape
  2. Confidentiality management -- many searches are highly confidential
  3. Assessment rigor -- psychometric evaluations, reference checks, board-ready candidate presentations
  4. Long sales cycles -- 6-12 months to win a new executive search client relationship
  5. Thought leadership pressure -- partners must publish, speak, and maintain visible industry presence
- **AI Integrator 30-Day Focus:** Executive landscape mapping (Apify + Claude research on public filings, board compositions, leadership changes), confidential candidate outreach sequences with careful brand positioning, AI-assisted candidate assessment report generation, thought leadership content engine (Claude drafts articles, speaking proposals, market commentary).
- **Recommended Tier:** Professional ($2,000)
- **Expected ROI:** 5x in first quarter. Even one additional completed search per quarter per partner (at $100,000+ average fee) delivers massive returns. The real leverage is in research time reduction: cutting 40 hours of manual market mapping to 4 hours per engagement.

---

## 8. MSP / Managed Service Provider

- **Model:** Agency manages the entire contingent workforce program for a large enterprise, including vendor management, compliance, reporting, and rate card management. Revenue comes from management fees (percentage of total spend) or per-transaction fees.
- **Revenue:** Management fees (2-5% of total contingent spend managed) or monthly retainers. Gross margins 15-30%, net margins 5-12%. Revenue is highly predictable.
- **Top 5 Pain Points:**
  1. Vendor performance management -- tracking and scoring dozens of staffing suppliers
  2. Spend visibility -- aggregating data across multiple VMS platforms and vendors
  3. Compliance reporting -- regulatory requirements across states and countries
  4. Rate card management -- ensuring competitive and consistent pricing
  5. Client stakeholder management -- satisfying multiple hiring managers with different needs
- **AI Integrator 30-Day Focus:** Vendor scorecard automation (pulls data from VMS, calculates performance metrics, generates rankings), spend analytics dashboard, compliance audit automation, rate benchmarking engine (scrapes market rate data, compares to current cards), automated requisition routing to top-performing vendors.
- **Recommended Tier:** Enterprise ($5,000)
- **Expected ROI:** 7x in first year. Automation enables the MSP to manage 30-50% more contingent spend without proportional headcount. On a $50M managed program, a 1% efficiency gain in vendor management translates to $500K in value delivered to the client.

---

## 9. SOW / Statement of Work

- **Model:** Agency scopes and delivers project-based work with defined deliverables, timelines, and outcomes. Billing is milestone-based or fixed-price rather than hourly. This model is growing as enterprises shift from time-and-materials to outcome-based engagements.
- **Revenue:** Project fees based on deliverables. Gross margins 20-40%, net margins 8-15%. Higher margins possible on well-scoped projects.
- **Top 5 Pain Points:**
  1. Scope creep -- clients expand requirements without corresponding price increases
  2. Resource planning -- matching available talent to project requirements and timelines
  3. Milestone tracking -- monitoring progress across multiple concurrent projects
  4. Profitability visibility -- real-time margin tracking per project is difficult
  5. SOW creation -- drafting detailed, legally sound statements of work is time-consuming
- **AI Integrator 30-Day Focus:** SOW generation engine (Claude drafts statements of work from intake forms), project margin tracker (real-time cost vs. budget monitoring), scope change detection alerts, resource availability matching, milestone progress automation.
- **Recommended Tier:** Professional ($2,000)
- **Expected ROI:** 6x in first quarter. Preventing even one scope creep incident per quarter saves $20,000-$50,000 in unbilled work. Faster SOW creation (from 8 hours to 1 hour) accelerates deal velocity.

---

## 10. Payrolling / Employer of Record (EOR)

- **Model:** Agency serves as the legal employer for workers identified by the client. The agency handles payroll, benefits, tax withholding, workers' compensation, and compliance. The client directs the work; the agency manages the employment relationship.
- **Revenue:** Per-employee-per-month fees ($100-$500) or percentage of payroll (2-8%). Gross margins 15-30%, net margins 5-10%. Highly scalable with volume.
- **Top 5 Pain Points:**
  1. Multi-jurisdiction compliance -- employment law varies by state, country, and municipality
  2. Onboarding volume -- processing hundreds of new hires per month with full documentation
  3. Benefits administration complexity -- managing multiple plan options across populations
  4. Payroll accuracy -- errors create compliance risk and worker dissatisfaction
  5. Client billing reconciliation -- matching timesheets to invoices across large worker populations
- **AI Integrator 30-Day Focus:** Automated onboarding document collection and verification, compliance rule engine (flags jurisdiction-specific requirements), payroll exception detection, benefits eligibility automation, invoice reconciliation bot.
- **Recommended Tier:** Starter ($1,000)
- **Expected ROI:** 10x in first year. Reducing onboarding processing time from 45 minutes to 10 minutes per worker, multiplied across hundreds of workers per month, delivers massive labor savings. Compliance error prevention avoids $10,000+ per incident in penalties.

---

## 11. On-Demand / Gig Platforms

- **Model:** Technology-enabled marketplace connecting workers to short-duration assignments (shifts, gigs, projects) in real time. Revenue comes from platform fees (10-20% of transaction value) or subscription models. Highly tech-dependent with mobile-first experiences.
- **Revenue:** Platform/transaction fees. Gross margins 40-60%, net margins vary widely (-20% to +15% depending on scale and growth stage).
- **Top 5 Pain Points:**
  1. Liquidity -- maintaining enough workers and jobs to create a functioning marketplace
  2. No-show rates -- gig workers have 15-30% no-show rates on accepted shifts
  3. Worker quality control -- ensuring consistent quality without traditional interviewing
  4. Price optimization -- dynamic pricing that balances worker supply and client demand
  5. Platform technology costs -- engineering and infrastructure are major expenses
- **AI Integrator 30-Day Focus:** No-show prediction model (scores likelihood of worker completing shift based on behavioral data), dynamic pricing engine, automated worker quality scoring from client feedback, supply-demand matching optimization, churn prediction for both workers and clients.
- **Recommended Tier:** Enterprise ($5,000)
- **Expected ROI:** 4x in first quarter. Reducing no-show rates by 5 percentage points on a platform processing 10,000 shifts/month at $200 average value translates to $100,000+ in recovered revenue. Price optimization adds another 3-5% to gross margins.

---

## 12. Hybrid Agencies

- **Model:** Agency operates across multiple business models simultaneously -- for example, offering contingency search, temp staffing, and RPO services from the same organization. This is increasingly common among mid-size and large firms seeking diversified revenue.
- **Revenue:** Blended across models. Overall gross margins 30-55%, net margins 5-12%. Revenue diversification reduces cyclical risk.
- **Top 5 Pain Points:**
  1. Operational complexity -- different processes, metrics, and systems for each service line
  2. Cross-selling inefficiency -- teams don't share client intelligence across divisions
  3. Reporting fragmentation -- no unified view of client relationships and revenue
  4. Technology sprawl -- multiple ATS/CRM platforms for different business lines
  5. Inconsistent client experience -- different service levels across divisions
- **AI Integrator 30-Day Focus:** Unified client intelligence dashboard (aggregates data across all business lines), cross-sell opportunity detection engine, standardized reporting layer across platforms, client health scoring that spans all engagement types, workflow templates for each service line.
- **Recommended Tier:** Enterprise ($5,000)
- **Expected ROI:** 8x in first year. Cross-sell detection alone typically uncovers 15-25% revenue growth opportunities from existing clients. A hybrid agency with $10M revenue finding even 10% cross-sell potential identifies $1M in pipeline.

---

## 13. Niche / Boutique Agencies

- **Model:** Agency specializes deeply in one vertical (healthcare, cybersecurity, fintech) or one function (CFOs, data engineers, DevOps). Competitive advantage comes from deep domain expertise, exclusive candidate networks, and market intelligence. Premium pricing is justified by specialization.
- **Revenue:** Premium placement fees (20-30% for perm, higher markups for temp). Gross margins 50-75%, net margins 10-20%.
- **Top 5 Pain Points:**
  1. Talent pool limitations -- specialized markets have finite candidate pools
  2. Knowledge management -- institutional expertise lives in individual recruiters' heads
  3. Market intelligence monetization -- deep knowledge is underutilized as a revenue stream
  4. Scaling without diluting expertise -- growth risks commoditizing the brand
  5. Competitive differentiation -- proving depth of expertise to clients
- **AI Integrator 30-Day Focus:** Knowledge base creation (Claude extracts and structures institutional expertise from emails, notes, and conversations), market intelligence report automation, candidate universe mapping for the niche, content engine for thought leadership in the vertical, competitive intelligence monitoring.
- **Recommended Tier:** Professional ($2,000)
- **Expected ROI:** 7x in first quarter. Monetizing market intelligence through automated reports adds a new revenue stream ($2,000-$5,000/report). Knowledge capture prevents catastrophic loss when senior recruiters depart (each departing recruiter costs $300,000+ in replacement and lost relationships).

---

## 14. Offshore / Nearshore Staffing

- **Model:** Agency sources talent from lower-cost geographies (India, Eastern Europe, Latin America) for clients in higher-cost markets. Revenue comes from the cost arbitrage between local wages and client bill rates. Growing rapidly in technology roles.
- **Revenue:** Bill rate markup (50-200%+). Gross margins 40-65%, net margins 10-20%. Margins are higher than domestic staffing due to wage arbitrage.
- **Top 5 Pain Points:**
  1. Cross-border compliance -- work permits, tax treaties, data privacy regulations (GDPR)
  2. Time zone coordination -- managing distributed teams across 6-12 hour time differences
  3. Quality perception -- overcoming client concerns about offshore talent quality
  4. Cultural fit assessment -- evaluating soft skills and communication across cultures
  5. Attrition in low-cost markets -- workers in talent-scarce markets get poached frequently
- **AI Integrator 30-Day Focus:** Compliance rule engine for cross-border employment requirements, cultural fit and communication assessment tools (Claude-powered interview analysis), time zone overlap optimizer, candidate retention risk scoring, client-facing talent quality dashboards with skills verification data.
- **Recommended Tier:** Professional ($2,000)
- **Expected ROI:** 9x in first quarter. Reducing attrition-driven replacement costs (each replacement costs $5,000-$15,000 in recruiting, onboarding, and client disruption) and accelerating compliance clearance by 50% both directly impact profitability.

---

## 15. Outplacement Services

- **Model:** Agency provides career transition support to employees being laid off or restructured. Services include resume writing, interview coaching, job search assistance, and emotional support. The departing employer pays for services (not the individual).
- **Revenue:** Per-participant fees ($1,000-$15,000 for individual programs, $200-$500 per person for group programs) or enterprise contracts. Gross margins 55-75%, net margins 15-25%.
- **Top 5 Pain Points:**
  1. Scale during layoff events -- hundreds of participants onboard simultaneously during RIF events
  2. Personalization at volume -- each participant needs individualized career guidance
  3. Outcome tracking -- measuring and reporting on job landing rates and time-to-land
  4. Participant engagement -- many participants are emotionally distressed and disengage
  5. Program content freshness -- job market advice and resources become outdated quickly
- **AI Integrator 30-Day Focus:** AI-powered resume rewriting at scale (Claude rewrites resumes for each participant based on target roles), personalized job search strategy generator, automated weekly check-in sequences, outcome tracking dashboard, market-specific job search content engine.
- **Recommended Tier:** Starter ($1,000)
- **Expected ROI:** 12x in first quarter. Handling a 200-person outplacement event that would normally require 5 career coaches with 2 coaches plus AI assistance saves $30,000+ in labor costs per event while improving participant outcomes (faster job landing times increase client satisfaction and referrals).

---

## 16. HR Tech / Platform Companies

- **Model:** Technology-first companies that sell software platforms with staffing-adjacent services. Revenue comes from SaaS subscriptions, usage fees, and sometimes transactional placement fees. Examples include ATS vendors, video interviewing platforms, and skills assessment tools.
- **Revenue:** SaaS subscriptions ($500-$50,000+/month) plus usage-based fees. Gross margins 70-85%, net margins 10-30% (high gross, but heavy R&D and sales costs).
- **Top 5 Pain Points:**
  1. Customer churn -- SaaS platforms face 10-20% annual churn
  2. Feature development velocity -- users expect rapid innovation and updates
  3. Data quality -- platform value depends on clean, enriched data
  4. User adoption -- purchased platforms often see low utilization rates
  5. Competitive differentiation -- crowded market with 500+ HR tech vendors
- **AI Integrator 30-Day Focus:** AI feature layer on top of existing platform (Claude-powered insights, recommendations, natural language queries), data enrichment pipeline, churn prediction model, automated user onboarding sequences, competitive feature monitoring.
- **Recommended Tier:** Enterprise ($5,000)
- **Expected ROI:** 6x in first year. Adding an AI intelligence layer increases platform stickiness, reducing churn by 3-5 percentage points. On a 1,000-customer base at $1,000/month average, a 3% churn reduction preserves $360,000 in annual recurring revenue.

---

## 17. Marketplace / Talent Platform

- **Model:** Two-sided marketplace connecting employers directly with candidates, with the platform facilitating matching, communication, and often payments. Distinguished from gig platforms by focusing on longer-term roles. Revenue from employer subscriptions, per-hire fees, or advertising.
- **Revenue:** Employer subscription ($200-$5,000/month), per-hire fees ($500-$5,000), or advertising. Gross margins 60-80%, net margins vary widely based on growth stage.
- **Top 5 Pain Points:**
  1. Cold start problem -- need both supply (candidates) and demand (employers) simultaneously
  2. Disintermediation risk -- employers and candidates bypass the platform after initial connection
  3. Matching quality -- poor matches erode trust on both sides of the marketplace
  4. Candidate profile freshness -- stale profiles degrade search results and match quality
  5. Revenue model optimization -- finding the right balance between free and paid features
- **AI Integrator 30-Day Focus:** AI matching algorithm enhancement (Claude-powered semantic matching beyond keyword search), profile freshness automation (prompts candidates to update, enriches profiles from public data), anti-disintermediation features (value-add communications through the platform), employer engagement scoring, conversion funnel optimization.
- **Recommended Tier:** Enterprise ($5,000)
- **Expected ROI:** 5x in first year. Improving match quality increases employer conversion from 5% to 12%+. On a platform with 1,000 active employers paying $500/month average, a 7-point conversion improvement adds $3.5M in annual revenue potential. Profile freshness improvements increase candidate-side engagement and reduce employer complaints.

---

## Tier Summary

| Tier | Price | Best For | Includes |
|------|-------|----------|----------|
| **Starter** | $1,000 | Temp-to-Perm, Direct Hire, Payrolling/EOR, Outplacement | Core Claude Code agents, 3 automated workflows, basic training |
| **Professional** | $2,000 | Contingency, Retained Search, Exec Search, SOW, Niche/Boutique, Offshore/Nearshore | Full Operations Hub (OpenClaw), 10+ Claude Code agents with n8n orchestration, 30-day deployment |
| **Enterprise** | $5,000 | RPO, Temp/Contract, MSP, Gig Platforms, Hybrid, HR Tech, Marketplace | Multi-system integration via direct ATS APIs, custom dashboards, advanced Claude Code agents, OpenClaw autonomous operations, dedicated support, 90-day deployment (30-day build + 60-day optimization) |

---

## Next Steps

1. **Identify your primary business model** (or models, if hybrid)
2. **Review the detailed deployment guide** for your type
3. **Schedule a discovery call** to map your specific tech stack and workflows
4. **Begin the 30-day deployment** with your AI Integrator

---

*Last updated: 2026-03-24*
*Part of the UAIS Staffing AI Vertical -- AI Integrator Program*
