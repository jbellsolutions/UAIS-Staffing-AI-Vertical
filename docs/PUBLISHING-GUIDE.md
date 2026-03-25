# Publishing Guide — Public vs Private Content

This guide defines which files from the UAIS Staffing AI Vertical project should be published to a public GitHub repository and which must remain private. Following this split ensures we share enough value to establish authority and generate inbound leads while protecting sensitive business strategy, pricing, and financial details.

---

## What Goes Public (GitHub)

These files are designed for public consumption and serve as lead generation:

| File | Purpose |
|------|---------|
| `README.md` | Project overview, value proposition, first impression |
| `docs/research/industry-complete-guide.md` | Comprehensive industry reference |
| `docs/research/problems-and-automation-guide.md` | The problem map — 13 pain categories across agency types |
| `docs/research/solutions-architecture.md` | Agent architectures and technical blueprints |
| `docs/research/case-studies.md` | Proof of results with real metrics |
| `solutions/by-agency-type/README.md` | Agency type solutions index |
| `solutions/by-agency-type/contingency-recruiting.md` | Deep dive example for one agency type |
| `competitive-analysis/market-landscape.md` | Competitive analysis showing whitespace |
| `LICENSE` | Open-source license |

**Why these are public:** Each document provides genuine, standalone value to staffing agency owners and technology leaders. They demonstrate depth of expertise without revealing how we monetize that expertise.

---

## What Stays Private (Do NOT Publish to GitHub)

These contain sensitive business strategy, pricing details, sales scripts, and financial projections:

| File | Reason to Keep Private |
|------|----------------------|
| `docs/research/business-plan.md` | Full business plan with financials and unit economics |
| `docs/gtm/staffing-vertical-gtm.md` | Sales scripts, email sequences, close rates, funnel metrics |
| `offer-packages/staffing-offer-suite.md` | Full offer details with value stacking and pricing tiers |
| `offer-packages/roi-calculator.md` | Internal ROI math and margin calculations |
| `financial-model/revenue-projections.md` | Revenue targets, margins, unit economics |
| `docs/research/REVIEW-agent1-research-gaps.md` | Internal quality review |
| `docs/research/REVIEW-agent2-solutions-quality.md` | Internal quality review |
| `docs/research/REVIEW-agent3-gtm-launch.md` | Internal quality review |

**Why these are private:** Publishing pricing, sales scripts, or financial projections would undermine competitive advantage and give competitors a playbook to replicate the business model.

---

## How to Split for GitHub

### Option 1: Use `.gitignore` (Recommended)

Add a `.gitignore` file at the project root that excludes all private files. This keeps a single working directory while ensuring `git add .` never stages sensitive content.

```
# Private business documents — DO NOT publish
docs/research/business-plan.md
docs/gtm/
offer-packages/
financial-model/
docs/research/REVIEW-*.md

# OS files
.DS_Store
Thumbs.db

# Environment
.env
.env.local
```

### Option 2: Two-Directory Structure

Maintain separate directories:
- `public/` — mirrors the GitHub repo structure
- `private/` — contains all business-sensitive files

### Option 3: Two Repositories

- **Public repo** (`uais-staffing-ai-vertical`): Contains only the files listed in the public section above.
- **Private repo** (`uais-staffing-internal`): Contains all files, including private strategy documents.

---

## What the Public Repo Accomplishes

1. **Establishes authority** — UAIS becomes THE reference for AI in staffing agencies. No one else has published anything this comprehensive.
2. **Provides genuine free value** — Problem maps, architecture docs, and solutions give agency owners actionable insight without a sales call.
3. **Drives inbound leads** — Agency owners discover the repo, see the depth of research, and reach out because they want someone who understands their world this well.
4. **Builds open-source credibility** — Positions UAIS as a transparent, knowledge-first company in the staffing tech community.
5. **Creates a first-mover moat** — Whoever publishes this comprehensive resource first owns the space. Competitors publishing after will look like they copied.

---

## Publishing Checklist

Before pushing to public GitHub:

- [ ] Verify `.gitignore` is in place and working
- [ ] Run `git status` to confirm no private files are staged
- [ ] Review `README.md` for any pricing or internal references
- [ ] Confirm all public documents are self-contained (no broken links to private files)
- [ ] Add a clear call-to-action in `README.md` directing interested readers to contact UAIS
- [ ] Double-check that no email sequences, close rates, or financial figures appear in public files
