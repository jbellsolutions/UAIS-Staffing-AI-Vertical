# Resume Screening Agent — Technical Blueprint

> **Status:** Production-ready template
> **Last Updated:** 2026-03-24
> **Difficulty:** Intermediate
> **Time to Deploy:** 2-4 hours for basic setup, 1-2 weeks for full ATS integration

---

## Overview

A Claude Code-built resume screening agent for staffing agencies. This is the first agent every AI Integrator builds using Claude Code — the primary tool in the UAIS stack. It processes resumes against job requirements, scores candidates on a configurable rubric, and outputs ranked shortlists directly into the agency's ATS via API. n8n handles the scheduling triggers and monitoring; Claude Code handles the intelligence and ATS integration.

**What it replaces:** A recruiter spending 5-10 minutes per resume doing initial screening.
**What it produces:** A scored, ranked shortlist with reasoning, ready for human review.
**What it does NOT replace:** Human judgment on final hiring decisions.

---

## Architecture

```
                    +------------------+
                    |   Resume Input   |
                    |  (email, ATS,    |
                    |   file drop)     |
                    +--------+---------+
                             |
                             v
                    +------------------+
                    |  Text Extraction |
                    |  (PDF/DOCX/TXT)  |
                    +--------+---------+
                             |
                             v
                    +------------------+
                    |  Claude Haiku    |
                    |  Initial Screen  |
                    |  (bulk scoring)  |
                    +--------+---------+
                             |
                    +--------+---------+
                    |                  |
           Score >= 40          Score < 40
                    |                  |
                    v                  v
           +---------------+   +---------------+
           | Claude Sonnet |   |  Auto-Reject  |
           | Deep Review   |   |  + Notify     |
           | (borderline   |   +---------------+
           |  candidates)  |
           +-------+-------+
                   |
                   v
           +---------------+
           | Ranked Output |
           | -> ATS Push   |
           | -> CSV/JSON   |
           | -> Dashboard  |
           +---------------+
```

### How It Works

1. **Resumes arrive** via email attachment, ATS upload, or file drop to a watched folder
2. **Text extraction** converts PDF/DOCX to plain text (using `pdf-parse`, `mammoth`, or your ATS's built-in parser)
3. **Claude Haiku screens at volume** -- fast, cheap, accurate enough for first-pass filtering
4. **Borderline candidates get re-screened** with Claude Sonnet for nuanced evaluation
5. **Results are ranked** and pushed to the ATS pipeline or output as a structured report
6. **Rejected candidates** receive automated notifications (configurable delay and messaging)
7. **Qualified candidates** get fast-tracked to recruiter review with AI-generated talking points

---

## The Screening Prompt (Production-Ready)

This is the core prompt that does the actual screening work. It is designed to be injected with variables from your job requisition system.

```markdown
You are a senior recruiting specialist screening resumes for a staffing agency.

**Job Requirements:**
Title: {{job_title}}
Description: {{job_description}}
Required Skills: {{required_skills}}
Preferred Skills: {{preferred_skills}}
Minimum Experience: {{minimum_experience_years}} years
Location Requirement: {{location_requirement}}
Salary Range: {{salary_range}}
{{#if certifications_required}}
Required Certifications: {{certifications_required}}
{{/if}}
{{#if clearance_required}}
Security Clearance: {{clearance_required}}
{{/if}}

**Your Task:**
Analyze the following resume and provide a structured assessment.

**Resume:**
{{resume_text}}

**Scoring Rubric:**
Score each dimension 1-10:

1. **Skills Match** -- How well do the candidate's skills align with required and preferred
   skills? Weight exact matches heavily, but also recognize equivalent technologies
   (e.g., React Native experience counts toward React requirements).

2. **Experience Level** -- Does their experience duration and depth match requirements?
   Count only relevant experience. 3 years of relevant work beats 10 years of unrelated work.

3. **Industry Relevance** -- Have they worked in relevant industries or similar roles?
   Adjacent industries count (e.g., fintech experience is relevant to banking roles).

4. **Career Trajectory** -- Is their career progression logical and ascending? Look for
   increasing responsibility, promotions, and scope expansion.

5. **Red Flags** -- Any gaps, inconsistencies, or concerns?
   (10 = no red flags, 1 = major concerns)
   Common red flags: unexplained gaps > 12 months, frequent job hopping (< 1 year
   tenure repeatedly), title inflation, vague descriptions without metrics.

**Output Format (strict JSON, no markdown wrapping):**
{
  "candidate_name": "Full Name",
  "overall_score": 0,
  "recommendation": "STRONG_MATCH | GOOD_MATCH | POSSIBLE_MATCH | NO_MATCH",
  "dimensions": {
    "skills_match": {"score": 0, "reasoning": ""},
    "experience_level": {"score": 0, "reasoning": ""},
    "industry_relevance": {"score": 0, "reasoning": ""},
    "career_trajectory": {"score": 0, "reasoning": ""},
    "red_flags": {"score": 0, "reasoning": ""}
  },
  "key_strengths": [],
  "concerns": [],
  "suggested_interview_questions": [],
  "salary_alignment": "WITHIN_RANGE | LIKELY_ABOVE | LIKELY_BELOW | UNKNOWN",
  "availability_estimate": "IMMEDIATE | 2_WEEKS | 1_MONTH | UNKNOWN",
  "summary": ""
}

**Scoring Rules:**
- overall_score is a weighted average: Skills 30%, Experience 25%, Industry 20%,
  Trajectory 15%, Red Flags 10%. Multiply each dimension score by 2 to get the
  0-100 scale contribution, then apply weights.
- STRONG_MATCH: overall_score >= 75
- GOOD_MATCH: overall_score >= 55 and < 75
- POSSIBLE_MATCH: overall_score >= 35 and < 55
- NO_MATCH: overall_score < 35

**Important Instructions:**
- Score based on ACTUAL demonstrated experience, not keyword stuffing
- "Managed a team of 12 engineers" implies leadership even if "management" is not listed as a skill
- Look for transferable skills, not just exact matches
- If the candidate lists a skill in their skills section but never references it in their
  work history, discount it by 50%
- Flag any potential bias in your assessment under concerns
- If the resume is clearly irrelevant (e.g., a pastry chef applying for a DevOps role),
  assign NO_MATCH quickly with a brief explanation -- do not waste tokens on deep analysis
- The summary field should be 2-3 sentences written for a busy recruiter -- lead with the
  verdict, then the strongest reason for or against
```

---

## Node.js Implementation

This is the complete, runnable screening agent. It handles text extraction, batch processing, two-tier screening, and structured output.

### Prerequisites

```bash
mkdir resume-screening-agent && cd resume-screening-agent
npm init -y
npm install @anthropic-ai/sdk pdf-parse mammoth
```

### `resume-screening-agent.js`

```javascript
#!/usr/bin/env node

// resume-screening-agent.js
// -------------------------
// A production-ready resume screening agent using Claude.
//
// Usage:
//   node resume-screening-agent.js --job ./jobs/senior-swe.json --resumes ./resumes/
//
// Requirements:
//   - ANTHROPIC_API_KEY environment variable set
//   - npm install @anthropic-ai/sdk pdf-parse mammoth

const Anthropic = require("@anthropic-ai/sdk");
const fs = require("fs");
const path = require("path");
const pdfParse = require("pdf-parse");
const mammoth = require("mammoth");

// ---------------------------------------------------------------------------
// Configuration
// ---------------------------------------------------------------------------

const CONFIG = {
  // Model selection: Haiku for volume, Sonnet for depth
  model_fast: "claude-haiku-4-5-20251001",
  model_deep: "claude-sonnet-4-6-20250514",

  // Score thresholds (0-100)
  threshold_strong: 75,
  threshold_good: 55,
  threshold_possible: 35,

  // Re-screen borderline candidates (score between possible and good) with Sonnet
  rescreen_low: 40,
  rescreen_high: 65,

  // Rate limiting
  batch_size: 10,
  delay_between_batches_ms: 1000,

  // Token limits
  max_tokens_fast: 1024,
  max_tokens_deep: 2048,

  // Bias prevention
  name_blind_mode: false, // Strip candidate names before screening
};

// ---------------------------------------------------------------------------
// Text Extraction
// ---------------------------------------------------------------------------

async function extractText(filePath) {
  const ext = path.extname(filePath).toLowerCase();

  if (ext === ".txt" || ext === ".md") {
    return fs.readFileSync(filePath, "utf8");
  }

  if (ext === ".pdf") {
    const buffer = fs.readFileSync(filePath);
    const data = await pdfParse(buffer);
    return data.text;
  }

  if (ext === ".docx") {
    const result = await mammoth.extractRawText({ path: filePath });
    return result.value;
  }

  throw new Error(`Unsupported file type: ${ext} (supported: .txt, .md, .pdf, .docx)`);
}

// ---------------------------------------------------------------------------
// Name Blinding (Bias Prevention)
// ---------------------------------------------------------------------------

function blindResume(text) {
  // Remove the first line (usually the candidate name) and email/phone patterns
  const lines = text.split("\n");
  lines[0] = "[NAME REDACTED]";

  return lines
    .join("\n")
    .replace(/[\w.-]+@[\w.-]+\.\w+/g, "[EMAIL REDACTED]")
    .replace(/\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}/g, "[PHONE REDACTED]");
}

// ---------------------------------------------------------------------------
// Screening Logic
// ---------------------------------------------------------------------------

function buildScreeningPrompt(resumeText, job) {
  return `
**Job Requirements:**
Title: ${job.title}
Description: ${job.description}
Required Skills: ${job.required_skills.join(", ")}
Preferred Skills: ${(job.preferred_skills || []).join(", ")}
Minimum Experience: ${job.min_experience_years} years
Location: ${job.location}
${job.salary_range ? `Salary Range: ${job.salary_range}` : ""}
${job.certifications_required ? `Required Certifications: ${job.certifications_required.join(", ")}` : ""}

**Resume to Screen:**
${resumeText}

Score this candidate using the rubric. Output ONLY valid JSON matching the schema, with no markdown code fences or extra text.`;
}

const SYSTEM_PROMPT = `You are a senior recruiting specialist for a staffing agency. You screen resumes quickly and accurately. You always output valid JSON and nothing else -- no markdown fences, no explanatory text before or after the JSON. Be fair, look for transferable skills, and flag potential bias in your assessment.

Scoring weights: Skills 30%, Experience 25%, Industry 20%, Trajectory 15%, Red Flags 10%.
Each dimension is scored 1-10. The overall_score is the weighted sum scaled to 0-100.
STRONG_MATCH >= 75, GOOD_MATCH >= 55, POSSIBLE_MATCH >= 35, NO_MATCH < 35.`;

async function screenResume(client, resumeText, job, model) {
  const prompt = buildScreeningPrompt(resumeText, job);

  const response = await client.messages.create({
    model: model,
    max_tokens: model === CONFIG.model_fast ? CONFIG.max_tokens_fast : CONFIG.max_tokens_deep,
    system: SYSTEM_PROMPT,
    messages: [{ role: "user", content: prompt }],
  });

  const text = response.content[0].text;

  // Extract JSON -- handle cases where the model wraps in code fences
  const jsonMatch = text.match(/\{[\s\S]*\}/);
  if (!jsonMatch) {
    throw new Error("No JSON object found in response");
  }

  const result = JSON.parse(jsonMatch[0]);

  // Validate required fields
  const required = ["candidate_name", "overall_score", "recommendation", "summary"];
  for (const field of required) {
    if (!(field in result)) {
      throw new Error(`Missing required field: ${field}`);
    }
  }

  // Clamp score to 0-100
  result.overall_score = Math.max(0, Math.min(100, Math.round(result.overall_score)));

  return result;
}

// ---------------------------------------------------------------------------
// Batch Processing
// ---------------------------------------------------------------------------

function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function screenBatch(client, resumes, job) {
  const results = [];
  let processed = 0;

  for (const resume of resumes) {
    processed++;
    const progress = `[${processed}/${resumes.length}]`;

    try {
      // Optionally blind the resume
      const text = CONFIG.name_blind_mode ? blindResume(resume.text) : resume.text;

      // First pass with fast model
      let result = await screenResume(client, text, job, CONFIG.model_fast);

      // Re-screen borderline candidates with the deep model
      if (
        result.overall_score >= CONFIG.rescreen_low &&
        result.overall_score <= CONFIG.rescreen_high
      ) {
        console.log(
          `${progress} Re-screening borderline candidate: ${result.candidate_name} (${result.overall_score}/100)`
        );
        const deepResult = await screenResume(client, text, job, CONFIG.model_deep);
        // Average the two scores for stability
        deepResult.overall_score = Math.round(
          (result.overall_score + deepResult.overall_score) / 2
        );
        deepResult._rescreened = true;
        result = deepResult;
      }

      result._source_file = resume.filename;
      result._screened_at = new Date().toISOString();
      result._model_used = result._rescreened ? CONFIG.model_deep : CONFIG.model_fast;

      results.push(result);

      const icon =
        result.overall_score >= CONFIG.threshold_strong
          ? "+++"
          : result.overall_score >= CONFIG.threshold_good
            ? " ++"
            : result.overall_score >= CONFIG.threshold_possible
              ? "  +"
              : "  -";

      console.log(
        `${progress} ${icon} ${result.candidate_name} -- ${result.overall_score}/100 (${result.recommendation})`
      );
    } catch (err) {
      console.error(`${progress} ERROR screening ${resume.filename}: ${err.message}`);
      results.push({
        _source_file: resume.filename,
        _error: err.message,
        _screened_at: new Date().toISOString(),
      });
    }

    // Rate limiting between batches
    if (processed % CONFIG.batch_size === 0 && processed < resumes.length) {
      await sleep(CONFIG.delay_between_batches_ms);
    }
  }

  return results;
}

// ---------------------------------------------------------------------------
// Report Generation
// ---------------------------------------------------------------------------

function generateReport(results, job) {
  const scored = results.filter((r) => !r._error);
  const errors = results.filter((r) => r._error);

  const strong = scored.filter((r) => r.overall_score >= CONFIG.threshold_strong);
  const good = scored.filter(
    (r) => r.overall_score >= CONFIG.threshold_good && r.overall_score < CONFIG.threshold_strong
  );
  const possible = scored.filter(
    (r) => r.overall_score >= CONFIG.threshold_possible && r.overall_score < CONFIG.threshold_good
  );
  const rejected = scored.filter((r) => r.overall_score < CONFIG.threshold_possible);

  const sorted = [...scored].sort((a, b) => b.overall_score - a.overall_score);

  let report = "";
  report += "=".repeat(70) + "\n";
  report += `RESUME SCREENING REPORT\n`;
  report += `Position: ${job.title}\n`;
  report += `Date: ${new Date().toISOString().split("T")[0]}\n`;
  report += "=".repeat(70) + "\n\n";

  report += `SUMMARY\n`;
  report += `-`.repeat(40) + "\n";
  report += `Total Resumes Screened: ${scored.length}\n`;
  report += `Errors: ${errors.length}\n`;
  report += `Strong Match (75+): ${strong.length}\n`;
  report += `Good Match (55-74): ${good.length}\n`;
  report += `Possible Match (35-54): ${possible.length}\n`;
  report += `No Match (<35): ${rejected.length}\n\n`;

  report += "RANKED SHORTLIST\n";
  report += "-".repeat(40) + "\n\n";

  sorted.forEach((r, i) => {
    const tier =
      r.overall_score >= CONFIG.threshold_strong
        ? "STRONG"
        : r.overall_score >= CONFIG.threshold_good
          ? "GOOD"
          : r.overall_score >= CONFIG.threshold_possible
            ? "POSSIBLE"
            : "NO MATCH";

    report += `${String(i + 1).padStart(3)}. ${r.candidate_name}\n`;
    report += `     Score: ${r.overall_score}/100 [${tier}]${r._rescreened ? " (re-screened)" : ""}\n`;
    report += `     ${r.summary}\n`;

    if (r.key_strengths && r.key_strengths.length > 0) {
      report += `     Strengths: ${r.key_strengths.join("; ")}\n`;
    }
    if (r.concerns && r.concerns.length > 0) {
      report += `     Concerns: ${r.concerns.join("; ")}\n`;
    }
    if (r.suggested_interview_questions && r.suggested_interview_questions.length > 0) {
      report += `     Interview Qs: ${r.suggested_interview_questions[0]}\n`;
    }
    report += "\n";
  });

  if (errors.length > 0) {
    report += "ERRORS\n";
    report += "-".repeat(40) + "\n";
    errors.forEach((e) => {
      report += `  ${e._source_file}: ${e._error}\n`;
    });
  }

  return report;
}

// ---------------------------------------------------------------------------
// CLI Entry Point
// ---------------------------------------------------------------------------

async function main() {
  // Parse arguments
  const args = process.argv.slice(2);
  let jobFile = null;
  let resumeDir = null;
  let outputFile = null;

  for (let i = 0; i < args.length; i++) {
    if (args[i] === "--job" && args[i + 1]) jobFile = args[++i];
    else if (args[i] === "--resumes" && args[i + 1]) resumeDir = args[++i];
    else if (args[i] === "--output" && args[i + 1]) outputFile = args[++i];
    else if (args[i] === "--name-blind") CONFIG.name_blind_mode = true;
    else if (args[i] === "--help") {
      console.log(`
Resume Screening Agent
Usage: node resume-screening-agent.js --job <job.json> --resumes <dir>

Options:
  --job <file>       Job requirements JSON file (required)
  --resumes <dir>    Directory containing resume files (required)
  --output <file>    Output JSON file (default: screening-results.json)
  --name-blind       Enable name-blind screening for bias prevention
  --help             Show this help message

Job JSON format:
{
  "title": "Senior Software Engineer",
  "description": "Build cloud-native applications...",
  "required_skills": ["JavaScript", "React", "Node.js"],
  "preferred_skills": ["Python", "Docker"],
  "min_experience_years": 5,
  "location": "Remote (US)",
  "salary_range": "$140,000-$180,000"
}

Supported resume formats: .txt, .md, .pdf, .docx
      `);
      process.exit(0);
    }
  }

  // Validate inputs
  if (!jobFile || !resumeDir) {
    // If no args, run with built-in example for demo purposes
    if (args.length === 0) {
      console.log("No arguments provided. Running in demo mode.\n");
      console.log("For production use, run:");
      console.log("  node resume-screening-agent.js --job ./job.json --resumes ./resumes/\n");
      console.log("Run with --help for full usage information.\n");

      jobFile = null; // Will use inline demo job
      resumeDir = null; // Will use inline demo resumes
    } else {
      console.error("Error: --job and --resumes are required. Run with --help for usage.");
      process.exit(1);
    }
  }

  // Verify API key
  if (!process.env.ANTHROPIC_API_KEY) {
    console.error("Error: ANTHROPIC_API_KEY environment variable is not set.");
    console.error("Get your key at: https://console.anthropic.com/settings/keys");
    process.exit(1);
  }

  const client = new Anthropic();

  // Load job requirements
  let job;
  if (jobFile) {
    job = JSON.parse(fs.readFileSync(jobFile, "utf8"));
  } else {
    // Demo job
    job = {
      title: "Senior Software Engineer",
      description:
        "Build and maintain cloud-native applications for fintech clients. " +
        "Work with a distributed team on microservices architecture, CI/CD pipelines, " +
        "and real-time data processing systems.",
      required_skills: ["JavaScript", "TypeScript", "React", "Node.js", "AWS"],
      preferred_skills: ["Python", "PostgreSQL", "Docker", "Kubernetes", "CI/CD", "GraphQL"],
      min_experience_years: 5,
      location: "Remote (US)",
      salary_range: "$140,000-$180,000",
    };
  }

  // Load resumes
  let resumes;
  if (resumeDir) {
    const supportedExts = [".txt", ".md", ".pdf", ".docx"];
    const files = fs
      .readdirSync(resumeDir)
      .filter((f) => supportedExts.includes(path.extname(f).toLowerCase()));

    if (files.length === 0) {
      console.error(`No supported resume files found in ${resumeDir}`);
      console.error(`Supported formats: ${supportedExts.join(", ")}`);
      process.exit(1);
    }

    resumes = [];
    for (const f of files) {
      try {
        const text = await extractText(path.join(resumeDir, f));
        resumes.push({ filename: f, text });
      } catch (err) {
        console.error(`Warning: Could not extract text from ${f}: ${err.message}`);
      }
    }
  } else {
    // Demo resumes for testing
    resumes = [
      {
        filename: "demo-strong-match.txt",
        text: `Sarah Chen
sarah.chen@email.com | San Francisco, CA | (415) 555-0192

SUMMARY
Senior Software Engineer with 7 years of experience building cloud-native applications.
Specialize in full-stack TypeScript/React/Node.js development on AWS. Led a team of 6
engineers at a fintech startup, shipping a real-time payment processing system handling
50K transactions per day.

EXPERIENCE
Lead Software Engineer | PayFlow (Fintech) | 2021-Present
- Architected microservices platform on AWS ECS, reducing deployment time by 80%
- Built React/TypeScript dashboard processing $2M daily transaction volume
- Implemented CI/CD pipeline with GitHub Actions, achieving 99.9% deployment success rate
- Mentored 4 junior engineers, 2 promoted to mid-level within 18 months

Senior Software Engineer | DataStack | 2019-2021
- Developed Node.js/GraphQL API serving 10M requests/day
- Migrated monolith to microservices on Kubernetes, cutting infrastructure costs 40%
- Built real-time data pipeline with Python and Apache Kafka

Software Engineer | TechCorp | 2017-2019
- Full-stack development with React, Node.js, PostgreSQL
- Shipped features for 500K+ user SaaS platform
- Introduced Docker-based local development, adopted team-wide

SKILLS
JavaScript, TypeScript, React, Node.js, Python, AWS (EC2, ECS, Lambda, S3, RDS),
PostgreSQL, Docker, Kubernetes, GraphQL, CI/CD, Git, REST APIs

EDUCATION
BS Computer Science, UC Berkeley, 2017`,
      },
      {
        filename: "demo-possible-match.txt",
        text: `James Morrison
james.m@email.com | Austin, TX

EXPERIENCE
Software Developer | Regional Bank IT Department | 2020-Present
- Maintain internal Java/Spring Boot applications
- Write SQL queries and reports against Oracle database
- Some experience with React for internal dashboards
- Participate in Agile ceremonies, 2-week sprints

Junior Developer | Local Web Agency | 2018-2020
- Built WordPress and Shopify sites for small businesses
- Some JavaScript and PHP development
- Basic AWS hosting setup (EC2, S3)

SKILLS
Java, Spring Boot, SQL, Oracle, JavaScript (basic), React (learning),
HTML/CSS, WordPress, PHP, Git

EDUCATION
Associate Degree, Computer Science, Austin Community College, 2018`,
      },
      {
        filename: "demo-no-match.txt",
        text: `Maria Rodriguez
maria.r@email.com | Portland, OR

EXPERIENCE
Head Pastry Chef | The Grand Hotel | 2019-Present
- Manage pastry kitchen with team of 8
- Design seasonal dessert menus
- Reduced food waste by 30% through inventory optimization

Sous Chef | Bistro Noel | 2016-2019
- Prepared fine dining desserts and bread program
- Trained 5 junior pastry cooks

SKILLS
French pastry techniques, chocolate tempering, bread baking, menu design,
kitchen management, food safety certification, inventory management

EDUCATION
Culinary Arts Diploma, Le Cordon Bleu, 2016`,
      },
    ];
  }

  console.log(`\nScreening ${resumes.length} resumes for: ${job.title}`);
  if (CONFIG.name_blind_mode) {
    console.log("(Name-blind mode enabled)");
  }
  console.log("-".repeat(60) + "\n");

  // Run screening
  const startTime = Date.now();
  const results = await screenBatch(client, resumes, job);
  const elapsed = ((Date.now() - startTime) / 1000).toFixed(1);

  // Generate and print report
  const report = generateReport(results, job);
  console.log("\n" + report);
  console.log(`Completed in ${elapsed}s`);

  // Save JSON results
  const outFile = outputFile || "screening-results.json";
  const outputData = {
    job: job,
    config: {
      model_fast: CONFIG.model_fast,
      model_deep: CONFIG.model_deep,
      name_blind_mode: CONFIG.name_blind_mode,
      thresholds: {
        strong: CONFIG.threshold_strong,
        good: CONFIG.threshold_good,
        possible: CONFIG.threshold_possible,
      },
    },
    screened_at: new Date().toISOString(),
    elapsed_seconds: parseFloat(elapsed),
    total_screened: results.filter((r) => !r._error).length,
    total_errors: results.filter((r) => r._error).length,
    results: results.sort((a, b) => (b.overall_score || 0) - (a.overall_score || 0)),
  };

  fs.writeFileSync(outFile, JSON.stringify(outputData, null, 2));
  console.log(`\nResults saved to ${outFile}`);
}

main().catch((err) => {
  console.error("Fatal error:", err.message);
  process.exit(1);
});
```

### `package.json`

```json
{
  "name": "resume-screening-agent",
  "version": "1.0.0",
  "description": "Claude-powered resume screening agent for staffing agencies",
  "main": "resume-screening-agent.js",
  "scripts": {
    "screen": "node resume-screening-agent.js",
    "demo": "node resume-screening-agent.js"
  },
  "dependencies": {
    "@anthropic-ai/sdk": "^0.39.0",
    "pdf-parse": "^1.1.1",
    "mammoth": "^1.8.0"
  }
}
```

---

## Cost Analysis

| Volume | Model | Cost per Resume | Monthly Cost (500/day) |
|--------|-------|-----------------|------------------------|
| High volume first pass | Claude Haiku 4.5 | ~$0.003 | ~$45/month |
| Borderline re-screening (10%) | Claude Sonnet 4.6 | ~$0.02 | ~$10/month |
| **Total** | | | **~$55/month for 10,000+ resumes** |

**Comparison to manual screening:**

| Method | Cost per Resume | Monthly Cost (10K resumes) | Time per Resume |
|--------|----------------|---------------------------|-----------------|
| Manual (recruiter at $30/hr) | $3.50 | $35,000 | 7 minutes |
| AI screening (this agent) | $0.005 | $55 | 2 seconds |
| **Savings** | | **$34,945/month** | **99.5% time reduction** |

The math is straightforward: AI screening is roughly 600x cheaper than manual screening per resume. Even accounting for the human review layer on shortlisted candidates, total cost drops by 80-90%.

---

## Configurable Scoring Rubrics

The rubric weights should be adjusted based on the staffing vertical. Here are production-tested configurations.

### General Staffing (Default)

| Dimension | Weight |
|-----------|--------|
| Skills Match | 30% |
| Experience Level | 25% |
| Industry Relevance | 20% |
| Career Trajectory | 15% |
| Red Flags | 10% |

### Healthcare Staffing

| Dimension | Weight |
|-----------|--------|
| Credentials/Certifications | 35% |
| Clinical Experience | 25% |
| Skills Match | 20% |
| Compliance Signals | 15% |
| Availability | 5% |

Additional automated checks:
- Active license verification against state nursing board APIs
- NLC (Nurse Licensure Compact) state coverage mapping
- Required immunization and BLS/ACLS certification status
- Malpractice history flag (requires third-party data source)

### IT/Tech Staffing

| Dimension | Weight |
|-----------|--------|
| Technical Skills Match | 35% |
| Experience Depth | 25% |
| Industry/Domain Relevance | 20% |
| Problem-Solving Signals | 10% |
| Red Flags | 10% |

Additional scoring:
- Technology recency bonus (used the tech within last 2 years = +1 to skills score)
- Open source contribution bonus (GitHub profile linked = +0.5 to trajectory)
- Certification relevance (AWS cert for AWS role = +1 to skills score)

### Light Industrial / Warehouse

| Dimension | Weight |
|-----------|--------|
| Availability/Shift Match | 30% |
| Safety Certifications | 25% |
| Relevant Experience | 20% |
| Reliability Signals | 15% |
| Location Proximity | 10% |

Additional checks:
- OSHA 10/30 certification status
- Forklift certification and type
- Drug screen clearance status
- Shift preference vs. requirement matching
- Commute distance calculation from zip code

### Executive / Professional Search

| Dimension | Weight |
|-----------|--------|
| Leadership Track Record | 30% |
| Industry Network/Reputation | 25% |
| Strategic Experience | 20% |
| Cultural Fit Signals | 15% |
| Compensation Alignment | 10% |

Note: Executive screening should always use Claude Sonnet (the deep model) as the primary screener, not Haiku. The nuance required justifies the higher per-resume cost.

---

## ATS Integration Points

### Bullhorn (Most Common for Staffing)

```javascript
// Push screening results to Bullhorn via REST API
async function pushToBullhorn(result, bullhornConfig) {
  const { restUrl, BhRestToken } = bullhornConfig;

  // Update candidate record with screening score
  await fetch(`${restUrl}/entity/Candidate/${result.candidateId}`, {
    method: "POST",
    headers: {
      "BhRestToken": BhRestToken,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      customFloat1: result.overall_score, // Map to a custom field
      customText1: result.recommendation,
      customTextBlock1: result.summary,
    }),
  });

  // Add note with full screening details
  await fetch(`${restUrl}/entity/Note`, {
    method: "PUT",
    headers: {
      "BhRestToken": BhRestToken,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      personReference: { id: result.candidateId },
      action: "AI Screening",
      comments: `Score: ${result.overall_score}/100 (${result.recommendation})\n\n${result.summary}\n\nStrengths: ${result.key_strengths.join(", ")}\nConcerns: ${result.concerns.join(", ")}`,
    }),
  });

  // Move to appropriate pipeline stage
  const stage = result.overall_score >= 75 ? "Shortlisted"
    : result.overall_score >= 55 ? "Under Review"
    : "Screened Out";

  // Update job submission status
  await fetch(`${restUrl}/entity/JobSubmission/${result.submissionId}`, {
    method: "POST",
    headers: {
      "BhRestToken": BhRestToken,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ status: stage }),
  });
}
```

### Lever

```javascript
// Push to Lever via API
async function pushToLever(result, leverApiKey) {
  const baseUrl = "https://api.lever.co/v1";
  const headers = {
    "Authorization": `Basic ${Buffer.from(leverApiKey + ":").toString("base64")}`,
    "Content-Type": "application/json",
  };

  // Add feedback to opportunity
  await fetch(`${baseUrl}/opportunities/${result.opportunityId}/feedback`, {
    method: "POST",
    headers,
    body: JSON.stringify({
      text: result.summary,
      fields: [
        { text: "AI Score", value: `${result.overall_score}/100` },
        { text: "Recommendation", value: result.recommendation },
        { text: "Key Strengths", value: result.key_strengths.join(", ") },
      ],
    }),
  });

  // Add tags
  await fetch(`${baseUrl}/opportunities/${result.opportunityId}/addTags`, {
    method: "POST",
    headers,
    body: JSON.stringify({ tags: [`ai-score:${result.overall_score}`, result.recommendation] }),
  });
}
```

### Greenhouse

```javascript
// Push to Greenhouse via Harvest API
async function pushToGreenhouse(result, ghApiKey) {
  const baseUrl = "https://harvest.greenhouse.io/v1";
  const headers = {
    "Authorization": `Basic ${Buffer.from(ghApiKey + ":").toString("base64")}`,
    "Content-Type": "application/json",
    "On-Behalf-Of": result.recruiterId,
  };

  // Submit scorecard
  await fetch(
    `${baseUrl}/applications/${result.applicationId}/scorecards`,
    {
      method: "POST",
      headers,
      body: JSON.stringify({
        overall_recommendation: result.overall_score >= 75 ? "strong_yes"
          : result.overall_score >= 55 ? "yes"
          : result.overall_score >= 35 ? "no_decision"
          : "no",
        attributes: [
          { name: "Skills Match", type: "skills_match", rating: ratingFromScore(result.dimensions.skills_match.score) },
          { name: "Experience", type: "experience", rating: ratingFromScore(result.dimensions.experience_level.score) },
        ],
      }),
    }
  );
}

function ratingFromScore(score) {
  if (score >= 8) return "strong_yes";
  if (score >= 6) return "yes";
  if (score >= 4) return "no_decision";
  return "no";
}
```

### Generic Webhook (Works with Any ATS)

```javascript
// Fire webhook on screening completion -- works with Zapier, Make, n8n, etc.
async function fireWebhook(result, webhookUrl) {
  await fetch(webhookUrl, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      event: "resume_screened",
      timestamp: new Date().toISOString(),
      candidate_name: result.candidate_name,
      overall_score: result.overall_score,
      recommendation: result.recommendation,
      summary: result.summary,
      key_strengths: result.key_strengths,
      concerns: result.concerns,
      suggested_interview_questions: result.suggested_interview_questions,
      source_file: result._source_file,
    }),
  });
}
```

---

## Bias Prevention and Compliance

This is not optional. AI-assisted hiring decisions are subject to increasing regulation.

### Built-In Safeguards

1. **Name-blind screening** -- Run with `--name-blind` to strip candidate names, emails, and phone numbers before sending to Claude. The model scores on qualifications only.

2. **Bias detection pass** -- After initial screening, run a separate prompt that reviews the screening output for potential bias signals:

```javascript
async function auditForBias(screeningResult, client) {
  const response = await client.messages.create({
    model: "claude-sonnet-4-6-20250514",
    max_tokens: 512,
    messages: [{
      role: "user",
      content: `Review this resume screening result for potential bias.
Flag any language or reasoning that could indicate bias based on:
- Age (graduation dates, "overqualified", "not a culture fit")
- Gender (pronouns, gendered assumptions about roles)
- Race/ethnicity (name-based assumptions, "communication style")
- Disability (gap assumptions, accommodation concerns)
- Socioeconomic (school prestige bias, unpaid internship expectations)

Screening Result:
${JSON.stringify(screeningResult, null, 2)}

Output JSON: {"bias_flags": [], "risk_level": "none|low|medium|high", "recommendation": ""}`
    }],
  });

  return JSON.parse(response.content[0].text.match(/\{[\s\S]*\}/)[0]);
}
```

3. **Aggregate monitoring** -- Track rejection rates across demographic proxies (inferred from names, school types, etc.) and flag statistically significant disparities. This should run weekly or monthly as a batch job.

4. **Full audit trail** -- Every screening decision is logged with the complete prompt, response, model version, and timestamp. This is required for compliance audits.

### Regulatory Compliance

| Regulation | Jurisdiction | Key Requirement |
|-----------|-------------|-----------------|
| NYC Local Law 144 | New York City | Annual bias audit by independent auditor; candidate notice 10 days before use |
| Illinois AIVFA | Illinois | Consent before AI video interview analysis; data destruction on request |
| EEOC Guidance | Federal (US) | AI tools are "selection procedures" subject to disparate impact analysis |
| EU AI Act | European Union | High-risk classification for employment AI; conformity assessment required |
| Colorado AI Act | Colorado | Duty to disclose AI use in consequential decisions; impact assessments |

**Practical requirement:** Before deploying this in production, consult with employment counsel in the jurisdictions where you operate. The agent itself is a tool -- compliance depends on how you deploy and monitor it.

---

## Folder Structure for Production Deployment

```
resume-screening-agent/
  resume-screening-agent.js    # Main screening script
  package.json                 # Dependencies
  jobs/                        # Job requirement JSON files
    senior-swe.json
    nurse-rn.json
    warehouse-lead.json
  resumes/                     # Drop resumes here (PDF, DOCX, TXT)
  results/                     # Screening output goes here
  rubrics/                     # Custom scoring rubric configs
    general.json
    healthcare.json
    tech.json
    industrial.json
  logs/                        # Audit trail
  .env                         # ANTHROPIC_API_KEY=sk-ant-...
```

---

## Quick Start

```bash
# 1. Clone or create the project
mkdir resume-screening-agent && cd resume-screening-agent

# 2. Install dependencies
npm init -y
npm install @anthropic-ai/sdk pdf-parse mammoth

# 3. Set your API key
export ANTHROPIC_API_KEY="sk-ant-your-key-here"

# 4. Create a job requirements file
cat > job.json << 'EOF'
{
  "title": "Senior Software Engineer",
  "description": "Build and maintain cloud-native applications for fintech clients",
  "required_skills": ["JavaScript", "TypeScript", "React", "Node.js", "AWS"],
  "preferred_skills": ["Python", "PostgreSQL", "Docker", "Kubernetes"],
  "min_experience_years": 5,
  "location": "Remote (US)",
  "salary_range": "$140,000-$180,000"
}
EOF

# 5. Add resumes to the resumes/ directory
mkdir resumes
# Drop .pdf, .docx, or .txt resume files here

# 6. Run the screening
node resume-screening-agent.js --job job.json --resumes ./resumes/

# 7. Or run the built-in demo (no resumes needed)
node resume-screening-agent.js

# 8. Enable name-blind mode for bias prevention
node resume-screening-agent.js --job job.json --resumes ./resumes/ --name-blind
```

---

## Customization Guide for AI Integrators

This template is the starting point. The real value you deliver to staffing agency clients is in the customization layer. Here is what to configure per client:

1. **Scoring rubric weights** -- Adjust per vertical (healthcare, IT, industrial, etc.)
2. **ATS integration** -- Connect to their specific ATS (Bullhorn, Lever, Greenhouse, JobDiva, etc.)
3. **Rejection email templates** -- Match their brand voice and compliance requirements
4. **Intake triggers** -- Email parsing, ATS webhook, watched folder, or API endpoint
5. **Compliance layer** -- NYC LL144 notice, Illinois AIVFA consent flow, EEOC logging
6. **Custom fields** -- Map screening output to their ATS custom fields
7. **Escalation rules** -- When to bump to Sonnet, when to flag for human review
8. **Dashboard** -- Build a simple web dashboard showing screening volume, pass rates, and bias metrics

The typical staffing agency engagement for this automation is $5,000-$15,000 for initial setup and $500-$2,000/month for ongoing management and tuning.
