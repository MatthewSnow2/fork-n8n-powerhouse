# IdeaForge — Automated Startup Idea Discovery Pipeline

A 5-workflow n8n pipeline that automatically discovers, synthesizes, scores, and publishes startup ideas from community signals.

## Pipeline Overview

```
Schedule (2AM UTC)
    │
    ▼
┌─────────────────────┐     ┌─────────────────────┐     ┌─────────────────────┐
│ WF-01: Signal       │────▶│ WF-02: Idea         │────▶│ WF-03: Idea         │
│ Harvester           │     │ Synthesizer         │     │ Scorer              │
│ (Reddit → Notion)   │     │ (Cluster → Claude)  │     │ (5-dim scoring)     │
└─────────────────────┘     └─────────────────────┘     └─────────────────────┘
                                                              │
Schedule (11AM UTC)                                           │
    │                                                         ▼
    ▼                                                   ┌─────────────────────┐
┌─────────────────────┐                                 │ Ideas DB (Notion)   │
│ WF-04: Daily        │◀────────────────────────────────│ status: scored      │
│ Publisher           │                                 └─────────────────────┘
│ (Email + LinkedIn)  │
└─────────────────────┘

On-demand (POST webhook):
┌─────────────────────┐
│ WF-05: Deep         │
│ Research Agent      │
│ (Full market study) │
└─────────────────────┘
```

## Workflows

| # | Workflow | Trigger | What It Does |
|---|----------|---------|--------------|
| 01 | Signal Harvester | Schedule (2AM) | Scrapes 6 subreddits for pain points, stores in Notion Raw Signals DB |
| 02 | Idea Synthesizer | Webhook (from WF-01) | Clusters signals by community, uses Claude to extract product ideas |
| 03 | Idea Scorer | Webhook (from WF-02) | Scores ideas on 5 dimensions using Claude, computes composite score |
| 04 | Daily Publisher | Schedule (11AM) + Manual webhook | Picks top scored ideas, generates email newsletter + LinkedIn post |
| 05 | Deep Research Agent | Manual webhook | Full market research: competitors, TAM, revenue model, GTM, tech assessment |

## Scoring Dimensions (WF-03)

| Dimension | Weight | What It Measures |
|-----------|--------|-----------------|
| `opportunity_score` | 0.25 | Market size and growth potential |
| `problem_score` | 0.30 | Pain severity and urgency |
| `feasibility_score` | 0.20 | Solo dev MVP feasibility |
| `why_now_score` | 0.10 | Timing advantage |
| `competition_score` | 0.15 | Differentiation potential |
| `composite_score` | — | Weighted average of above |

## Setup

### Prerequisites

- n8n Cloud instance (or self-hosted n8n v1.x+)
- Notion workspace with databases (see schema below)
- Anthropic API key (Claude access)
- Gmail/Google Workspace account (for WF-04 email sending)

### Required Credentials

| Credential | Used By | Type | Required? |
|-----------|---------|------|-----------|
| **Notion API** | WF-01, WF-02, WF-03, WF-04, WF-05 | Notion Internal Integration | Yes |
| **Anthropic API** | WF-02, WF-03, WF-04, WF-05 | API Key | Yes |
| **Google/Gmail** | WF-04 | Service Account | Yes (for email) |
| **Perplexity API** | WF-05 | API Key | Only for Deep Research |
| **Firecrawl API** | WF-05 | API Key | Only for Deep Research |
| **PDFShift API** | WF-05 | API Key | Only for Deep Research |
| **Google Drive** | WF-05 | Service Account | Only for Deep Research |

### Notion Database Schema

You need 4 Notion databases:

#### Raw Signals DB
| Property | Type | Notes |
|----------|------|-------|
| Title | Title | `signal_id` (e.g., `sig_nursing_abc123`) |
| `source` | Select | Options: `reddit` |
| `source_url` | URL | Link to original post |
| `raw_text` | Rich Text | Post title + body (500 char max) |
| `community` | Rich Text | Subreddit name |
| `engagement_score` | Number | `score + (num_comments * 2)` |
| `signal_type` | Select | `pain_point`, `wish`, `solution_request`, `complaint` |
| `harvested_at` | Date | ISO timestamp |
| `processed` | Checkbox | Set to `true` after idea extraction |
| `idea` | Relation | Links to Ideas DB |

#### Ideas DB
| Property | Type | Notes |
|----------|------|-------|
| Title | Title | `idea_name` from Claude |
| `Status` | Status | `synthesized` → `scored` → `published` → `deep_research` |
| `Description` | Rich Text | One-liner pitch |
| `Problem` | Rich Text | Problem narrative |
| `Solution` | Rich Text | Solution description |
| `Target Audience` | Rich Text | Who buys this |
| `Value Proposition` | Rich Text | Revenue model + price |
| `Source Signal IDs` | Rich Text | Comma-separated signal page IDs |
| `opportunity_score` | Number | 1-10 |
| `problem_score` | Number | 1-10 |
| `feasibility_score` | Number | 1-10 |
| `why_now_score` | Number | 1-10 |
| `competition_score` | Number | 1-10 |
| `composite_score` | Number | Weighted average |
| `publish_date` | Date | When published in newsletter |
| `featured_as` | Rich Text | `idea_of_the_day` or `also_today` |
| `report_url` | URL | Google Drive link (WF-05) |

#### Config DB
General configuration storage (not actively used by MVP workflows).

#### Errors DB
Error logging storage (not actively used by MVP workflows).

### Import Instructions

1. **Import each workflow JSON** into your n8n instance via Settings → Import Workflow
2. **Replace placeholders** in each workflow:
   - `<YOUR_RAW_SIGNALS_DB_ID>` → Your Notion Raw Signals database ID
   - `<YOUR_IDEAS_DB_ID>` → Your Notion Ideas database ID
   - `<YOUR_N8N_INSTANCE>` → Your n8n cloud URL (e.g., `https://your-instance.app.n8n.cloud`)
   - `<YOUR_EMAIL>` → Your email address for daily newsletter
   - `<YOUR_NOTION_CREDENTIAL_ID>` → Your n8n Notion credential ID
   - `<YOUR_ANTHROPIC_CREDENTIAL_ID>` → Your n8n Anthropic credential ID
   - `<YOUR_GOOGLE_CREDENTIAL_ID>` → Your n8n Google/Gmail credential ID
   - `<YOUR_PERPLEXITY_CREDENTIAL_ID>` → (WF-05 only) Perplexity credential ID
   - `<YOUR_FIRECRAWL_API_KEY>` → (WF-05 only) Firecrawl API key
   - `<YOUR_PDFSHIFT_API_KEY>` → (WF-05 only) PDFShift API key
   - `<YOUR_GOOGLE_DRIVE_FOLDER_ID>` → (WF-05 only) Google Drive folder for reports
3. **Activate workflows** in order: WF-03 → WF-02 → WF-01 → WF-04
4. **WF-05** should remain inactive until Firecrawl/PDFShift keys are configured

### Activation Order

Activate in reverse dependency order so webhooks are ready when upstream workflows call them:

1. **WF-03** (Idea Scorer) — webhook listener
2. **WF-02** (Idea Synthesizer) — webhook listener, calls WF-03
3. **WF-01** (Signal Harvester) — schedule trigger, calls WF-02
4. **WF-04** (Daily Publisher) — schedule trigger, reads from Ideas DB

## Customization

### Adding Subreddits
Edit the `Build Subreddit List` Code node in WF-01:
```javascript
const subreddits = [
  { subreddit: 'nursing', category: 'Healthcare' },
  { subreddit: 'healthIT', category: 'Healthcare Technology' },
  // Add more here
];
```

### Changing Schedule
- WF-01: Modify `Schedule Trigger` node (default: 2AM UTC daily)
- WF-04: Modify `Schedule Daily` node (default: 11AM UTC daily)

### Adjusting Signal Filters
Edit `Filter and Normalize Signals` in WF-01:
- `post.score >= 5` — minimum upvotes
- `post.num_comments >= 2` — minimum comments

### Modifying Scoring Weights
Edit the system prompt in WF-03's `Claude: Score Idea` node:
```
composite_score = (opportunity * 0.25) + (problem * 0.30) + ...
```

## MVP Limitations

- **Reddit only** — Trends API and YouTube branches were removed for MVP simplicity
- **Keyword clustering** — Uses community-based grouping instead of semantic embeddings (no Voyage AI dependency)
- **WF-05 partially functional** — Requires Firecrawl and PDFShift API keys for full operation
- **No error recovery** — Workflows don't automatically retry on failure
- **Rate limiting** — No explicit wait between Reddit requests (sufficient for 6 subreddits)

## Workflow IDs (Live Instance)

For reference, the live workflow IDs on `im4tlai.app.n8n.cloud`:

| Workflow | ID |
|----------|-----|
| Signal Harvester | `O6i7jU8u1px98mIU` |
| Idea Synthesizer | `iWtub08wZh3yrliO` |
| Idea Scorer | `NrCHkJpBV8Zhkrav` |
| Daily Publisher | `dU02BjoaZVTNpigV` |
| Deep Research Agent | `3Nqs3Kvgr1GEd168` |
