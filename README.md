# n8n Powerhouse

**Turn Claude into an n8n automation expert.** Build, validate, and deploy production-ready workflows through conversation.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![n8n](https://img.shields.io/badge/n8n-Automation-orange)](https://n8n.io)
[![Claude Code](https://img.shields.io/badge/Claude-Code-blue)](https://claude.ai/claude-code)

---

## The Problem

Building n8n workflows typically requires:
- Learning n8n's node configurations and quirks
- Understanding which nodes exist and how to configure them
- Knowing the right patterns for webhooks, APIs, databases
- Debugging expression syntax errors
- Figuring out why your workflow isn't validating

**What if Claude just knew all of this?**

---

## The Solution

n8n Powerhouse is a **skills framework** that gives Claude deep expertise in n8n automation. It combines:

1. **8 specialized skills** - Deep knowledge about n8n patterns, syntax, and best practices
2. **n8n MCP integration** - Direct connection to your n8n instance to create/edit/validate workflows
3. **Production guidance** - Built-in knowledge about idempotency, error handling, and operational concerns

---

## Before & After

### Before (without n8n Powerhouse)

```
You: Build me a workflow that syncs Shopify orders to my CRM

Claude: I can help you design a workflow. You would need:
1. A Shopify trigger node
2. Some transformation
3. A CRM node

You'll need to configure these in n8n...
[Generic advice, no actual implementation]
```

### After (with n8n Powerhouse)

```
You: Build me a workflow that syncs Shopify orders to my CRM

Claude: I'll build this for you. Let me:
1. Analyze your stack → Recommending pure n8n (both have OAuth nodes)
2. Apply webhook processing pattern with idempotency
3. Create the workflow in your n8n instance...

✅ Workflow created: "Shopify Order → Zoho CRM + Slack"
   ID: BbRLnEFYH9f4T6Jr
   URL: https://your-instance.app.n8n.cloud/workflow/BbRLnEFYH9f4T6Jr

Next steps:
1. Add your Shopify and Zoho credentials
2. Activate the workflow
```

**Claude actually builds and deploys the workflow.**

---

## What Are Claude Code Skills?

[Claude Code](https://claude.ai/claude-code) is Anthropic's CLI tool for working with Claude in your terminal. **Skills** are collections of markdown files that give Claude specialized knowledge.

When you put skills in `.claude/skills/`, Claude reads them and gains that expertise. n8n Powerhouse provides 8 skills (~25,000 words of n8n knowledge) covering:

- Architecture decisions
- Workflow patterns
- Node configuration
- Code syntax
- Expression debugging
- Validation

---

## What is n8n MCP?

[MCP (Model Context Protocol)](https://modelcontextprotocol.io/) allows Claude to use external tools. The [n8n MCP](https://www.npmjs.com/package/n8n-mcp) gives Claude tools to:

- **Search** for nodes and templates
- **Create** workflows in your n8n instance
- **Update** existing workflows
- **Validate** configurations
- **Trigger** webhook workflows
- **List** executions and debug

Together, skills + MCP = Claude can **know** n8n AND **do** n8n.

---

## Quick Start

### 1. Install n8n MCP

```bash
claude mcp add n8n-mcp-api \
  -e MCP_MODE=stdio \
  -e LOG_LEVEL=error \
  -e DISABLE_CONSOLE_OUTPUT=true \
  -e N8N_API_URL=https://YOUR-INSTANCE.app.n8n.cloud \
  -e N8N_API_KEY=your-api-key-here \
  -- npx -y n8n-mcp
```

**Get your API key:** n8n Dashboard → Settings → API → Create API Key

### 2. Clone This Repo

```bash
git clone https://github.com/promptadvisers/n8n-powerhouse.git
cd n8n-powerhouse
```

### 3. Start Claude Code

```bash
claude
```

That's it. Claude now has full n8n expertise.

---

## Real-World Examples

### Example 1: E-commerce Automation

```
You: I run a Shopify store and use Zoho CRM. When someone places an order,
     I want to sync the customer to Zoho and notify my team on Slack.

Claude: I'll analyze your stack and build this...

[Claude uses n8n-workflow-architect to analyze]
→ Shopify: OAuth, native trigger node ✓
→ Zoho CRM: OAuth, native node ✓
→ Slack: OAuth, native node ✓
→ Recommendation: Pure n8n (all services supported)
→ Pattern: Webhook Processing with idempotency

[Claude uses n8n MCP to create workflow]
→ Created: Shopify Order → Zoho CRM + Slack
→ Includes: Duplicate prevention (check email before create)
→ Includes: Error handling
→ Includes: Slack notification with order details

Here's your workflow: [link]
```

### Example 2: Tool Selection

```
You: Should I use n8n or Python to process 50,000 customer records daily?

Claude: [Uses n8n-workflow-architect/tool-selection-matrix.md]

For 50,000 records, I recommend a **hybrid approach**:

Use n8n for:
- Triggering (schedule trigger)
- Authentication (if APIs need OAuth)
- Notifications (Slack alerts)

Use Python for:
- The actual data processing (>5,000 record threshold)
- Memory-efficient streaming
- Complex transformations

Architecture:
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ n8n         │     │ Python      │     │ n8n         │
│ (Schedule)  │ ──► │ (Process)   │ ──► │ (Notify)    │
└─────────────┘     └─────────────┘     └─────────────┘

Why not pure n8n?
- n8n runs on Node.js with memory limits
- 50k records would likely crash or timeout
- Python can stream/chunk efficiently
```

### Example 3: Debugging Expressions

```
You: My expression isn't working: {{ $json.email }}
     The webhook sends {"body": {"email": "test@example.com"}}

Claude: [Uses n8n-expression-syntax/COMMON_MISTAKES.md]

Found the issue! Webhook data is nested under `body`.

❌ Wrong: {{ $json.email }}
✅ Correct: {{ $json.body.email }}

The webhook node wraps the payload:
{
  "headers": {...},
  "body": {           ← Your data is here
    "email": "test@example.com"
  }
}
```

### Example 4: Production Readiness

```
You: I'm about to deploy my Stripe webhook workflow. What should I check?

Claude: [Uses n8n-workflow-architect/production-readiness.md]

Pre-deployment checklist for payment webhooks:

□ Security
  - Verify Stripe webhook signatures (Code node)
  - Reject invalid signatures immediately

□ Idempotency
  - Store processed event IDs
  - Check before processing: SELECT 1 FROM processed_events WHERE event_id = $1
  - Skip if already processed

□ Error Handling
  - Error Trigger workflow → Slack #alerts
  - Log all failures with context

□ Response Time
  - Use responseMode: "onReceived" (acknowledge Stripe immediately)
  - Process async (Stripe retries on slow responses)

□ Testing
  - Use Stripe test mode
  - Verify with Stripe CLI: stripe trigger payment_intent.succeeded

Want me to add these patterns to your workflow?
```

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           USER REQUEST                                   │
│         "I need to automate X with Shopify, HubSpot, and AI"            │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      PLANNING LAYER                                      │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                    n8n-workflow-architect                          │  │
│  │  • Analyze business stack (Shopify, Zoho, HubSpot, etc.)          │  │
│  │  • Decide: n8n vs Python vs Hybrid                                 │  │
│  │  • Evaluate production readiness requirements                      │  │
│  └───────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    IMPLEMENTATION LAYER                                  │
│  ┌─────────────────────┐ ┌─────────────────────┐ ┌───────────────────┐  │
│  │ n8n-workflow-       │ │ n8n-node-           │ │ n8n-mcp-tools-    │  │
│  │ patterns            │ │ configuration       │ │ expert            │  │
│  │                     │ │                     │ │                   │  │
│  │ • Webhook processing│ │ • Node setup        │ │ • Tool selection  │  │
│  │ • API integration   │ │ • Property deps     │ │ • Parameter help  │  │
│  │ • Database ops      │ │ • Auth config       │ │ • MCP patterns    │  │
│  │ • AI agent workflows│ │ • Operation config  │ │                   │  │
│  │ • Scheduled tasks   │ │                     │ │                   │  │
│  └─────────────────────┘ └─────────────────────┘ └───────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      CODE & SYNTAX LAYER                                 │
│  ┌─────────────────────┐ ┌─────────────────────┐ ┌───────────────────┐  │
│  │ n8n-code-           │ │ n8n-code-           │ │ n8n-expression-   │  │
│  │ javascript          │ │ python              │ │ syntax            │  │
│  │                     │ │                     │ │                   │  │
│  │ • $input/$json      │ │ • _input/_json      │ │ • {{ }} syntax    │  │
│  │ • $helpers usage    │ │ • Standard library  │ │ • $json access    │  │
│  │ • DateTime handling │ │ • Python limits     │ │ • $node refs      │  │
│  └─────────────────────┘ └─────────────────────┘ └───────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      VALIDATION LAYER                                    │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                    n8n-validation-expert                           │  │
│  │  • Interpret validation errors • Fix common issues                 │  │
│  │  • Validation profiles • Pre-deployment checklist                  │  │
│  └───────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         n8n MCP TOOLS                                    │
│                                                                          │
│  Discovery             Workflow Mgmt           Validation                │
│  ─────────────────     ──────────────────     ────────────────────       │
│  • search_nodes        • n8n_create_workflow  • validate_workflow        │
│  • get_node_info       • n8n_update_workflow  • validate_node_operation  │
│  • list_nodes          • n8n_get_workflow     • validate_expressions     │
│  • list_ai_tools       • n8n_list_workflows   • validate_connections     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## The 8 Skills

| Skill | Purpose | Key Decision |
|-------|---------|--------------|
| **n8n-workflow-architect** | Strategic planning | "Should I use n8n or Python?" |
| **n8n-workflow-patterns** | Implementation patterns | "What's the right pattern for webhooks?" |
| **n8n-node-configuration** | Node setup | "What fields does this node need?" |
| **n8n-mcp-tools-expert** | MCP guidance | "Which MCP tool should I use?" |
| **n8n-code-javascript** | JS Code nodes | "How do I access $json in Code?" |
| **n8n-code-python** | Python Code nodes | "What Python modules are available?" |
| **n8n-expression-syntax** | Expression debugging | "Why isn't my expression working?" |
| **n8n-validation-expert** | Error fixing | "What does this validation error mean?" |

### Skill Details

<details>
<summary><strong>n8n-workflow-architect</strong> - Strategic planning</summary>

**When Claude uses this:**
- Starting a new automation project
- Deciding between n8n, Python, or hybrid
- Evaluating production readiness

**Key files:**
- `SKILL.md` - Decision framework
- `tool-selection-matrix.md` - n8n vs Python criteria (OAuth, volume, complexity)
- `business-stack-analysis.md` - SaaS compatibility guides
- `production-readiness.md` - Pre-launch checklist

**Decision criteria:**
| Use n8n when | Use Python when |
|--------------|-----------------|
| OAuth required | >5,000 records |
| Non-tech maintainers | >20MB files |
| Multi-day waits | Complex algorithms |
| Standard integrations | Cutting-edge AI |

</details>

<details>
<summary><strong>n8n-workflow-patterns</strong> - Implementation patterns</summary>

**The 5 core patterns:**

1. **Webhook Processing** - Receive HTTP → Process → Respond
   - Stripe webhooks, form submissions, chat integrations

2. **HTTP API Integration** - Fetch APIs → Transform → Store
   - Data sync, API aggregation, enrichment

3. **Database Operations** - Read/Write/Sync databases
   - ETL, migrations, backups

4. **AI Agent Workflow** - AI with tools and memory
   - Chatbots, content generation, analysis

5. **Scheduled Tasks** - Recurring automation
   - Daily reports, cleanup, monitoring

</details>

<details>
<summary><strong>n8n-node-configuration</strong> - Node setup</summary>

**Key concepts:**
- **Operation-aware:** Different operations need different fields
- **Property dependencies:** Fields show/hide based on other values
- **Progressive discovery:** Start with essentials, add complexity

**Example:** Slack node
```javascript
// For operation='post' (send message)
{ resource: "message", operation: "post", channel: "#general", text: "Hello!" }

// For operation='update' (edit message) - different fields!
{ resource: "message", operation: "update", messageId: "123", text: "Updated!" }
```

</details>

<details>
<summary><strong>n8n-code-javascript</strong> - JS Code nodes</summary>

**Key syntax:**
```javascript
// Access input data
const items = $input.all();
const firstItem = $json;

// Reference other nodes
const prevData = $node["Previous Node"].json;

// HTTP requests (built-in)
const response = await $helpers.httpRequest({
  method: 'GET',
  url: 'https://api.example.com/data'
});

// DateTime (Luxon)
const now = DateTime.now();
const formatted = now.toFormat('yyyy-MM-dd');
```

</details>

<details>
<summary><strong>n8n-expression-syntax</strong> - Expression debugging</summary>

**Common mistakes:**

| Wrong | Correct | Why |
|-------|---------|-----|
| `{{ $json.email }}` | `{{ $json.body.email }}` | Webhook data is under `.body` |
| `{{ json.field }}` | `{{ $json.field }}` | Missing `$` prefix |
| `$json.field` | `{{ $json.field }}` | Missing `{{ }}` wrapper |

**Useful patterns:**
```javascript
// Conditional
{{ $json.status === 'active' ? 'Yes' : 'No' }}

// Default value
{{ $json.name || 'Unknown' }}

// Reference other nodes
{{ $node["Extract Data"].json.email }}
```

</details>

---

## File Structure

```
n8n-powerhouse/
├── README.md                          # This file
├── INSTALL.md                         # Detailed setup guide
├── examples/
│   └── README.md                      # Example prompts
└── .claude/
    ├── CLAUDE.md                      # Main configuration
    └── skills/
        ├── n8n-workflow-architect/    # 4 files - Planning
        ├── n8n-workflow-patterns/     # 6 files - Patterns
        ├── n8n-node-configuration/    # 4 files - Node setup
        ├── n8n-mcp-tools-expert/      # 5 files - MCP guidance
        ├── n8n-code-javascript/       # 6 files - JS code
        ├── n8n-code-python/           # 6 files - Python code
        ├── n8n-expression-syntax/     # 4 files - Expressions
        └── n8n-validation-expert/     # 4 files - Validation
```

**Total:** 46 files, ~25,000 words of n8n expertise

---

## FAQ

### Do I need n8n Cloud or can I use self-hosted?

Both work. Just change `N8N_API_URL` to your instance URL.

### Does this work with n8n Community Edition?

Yes! You need API access enabled. In your n8n environment variables:
```
N8N_PUBLIC_API_ENABLED=true
```

### Can Claude activate workflows?

No. n8n's API doesn't support activation. Claude will create the workflow and tell you to activate it in the UI.

### What if Claude makes a mistake?

Workflows are created inactive. Always review before activating. Claude also validates before creating.

### How is this different from n8n's AI features?

n8n has built-in AI for generating workflows. n8n Powerhouse gives Claude **deep expertise** plus **direct API access**. Claude can:
- Make architectural decisions (n8n vs Python)
- Apply production patterns (idempotency, error handling)
- Debug complex issues
- Create workflows that follow best practices

### Can I add my own skills?

Yes! Add `.md` files to `.claude/skills/your-skill-name/` and update `.claude/CLAUDE.md`.

---

## MCP Tools Reference

<details>
<summary><strong>Discovery Tools</strong></summary>

```bash
search_nodes          # Find nodes by keyword
get_node_essentials   # Quick node overview (start here)
get_node_info         # Full documentation
list_nodes            # List by category/package
list_ai_tools         # AI-capable nodes
```

</details>

<details>
<summary><strong>Workflow Management</strong></summary>

```bash
n8n_create_workflow          # Create new workflow
n8n_get_workflow             # Get by ID
n8n_update_partial_workflow  # Incremental updates (add/remove nodes)
n8n_update_full_workflow     # Complete replacement
n8n_list_workflows           # List all workflows
n8n_delete_workflow          # Delete workflow
```

</details>

<details>
<summary><strong>Validation Tools</strong></summary>

```bash
validate_workflow             # Full workflow validation
validate_node_operation       # Single node validation
validate_workflow_connections # Check connections only
validate_workflow_expressions # Check expressions only
n8n_validate_workflow         # Validate by workflow ID
```

</details>

<details>
<summary><strong>Template Tools</strong></summary>

```bash
search_templates       # Search by keyword
list_node_templates    # Find templates using specific nodes
get_template           # Get full workflow JSON
get_templates_for_task # Curated templates by task type
```

</details>

<details>
<summary><strong>Execution Tools</strong></summary>

```bash
n8n_trigger_webhook_workflow  # Trigger via webhook
n8n_list_executions           # Execution history
n8n_get_execution             # Execution details
n8n_health_check              # Check n8n connectivity
```

</details>

---

## Production Readiness

The skills include production guidance for:

| Area | Coverage |
|------|----------|
| **Observability** | Error workflows, execution logging, health checks |
| **Idempotency** | Duplicate handling, check-before-create, idempotency keys |
| **Cost Management** | AI API costs, caching, model selection |
| **Operational Control** | Kill switches, approval queues, audit trails |
| **Security** | Webhook signatures, credential handling, input validation |

See [production-readiness.md](.claude/skills/n8n-workflow-architect/production-readiness.md) for the full checklist.

---

## Contributing

1. Fork this repo
2. Add/edit files in `.claude/skills/`
3. Update skill descriptions in `.claude/CLAUDE.md`
4. Submit a PR

### Skill Structure

Each skill needs:
- `SKILL.md` - Main file (loaded when skill is invoked)
- `README.md` - Human-readable overview
- Additional `.md` files for detailed topics

---

## Links

- [n8n Documentation](https://docs.n8n.io/)
- [n8n MCP on npm](https://www.npmjs.com/package/n8n-mcp)
- [Claude Code](https://claude.ai/claude-code)
- [Claude Code Skills Docs](https://docs.anthropic.com/en/docs/claude-code/skills)
- [Model Context Protocol](https://modelcontextprotocol.io/)

---

## License

MIT License - Use freely for your automation projects.

---

<p align="center">
  <strong>Built with Claude Code + n8n MCP</strong><br>
  <em>Turn conversations into production-ready automation</em>
</p>
