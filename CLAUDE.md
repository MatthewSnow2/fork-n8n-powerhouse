# n8n-powerhouse - Project Instructions

## Project Overview
Centralized n8n workflow management via Claude Code. Uses the n8n-mcp MCP server for full API access to all workflows.

## MCP Server
- **n8n-mcp**: Full n8n REST API access (list, get, create, update, validate workflows)
- Configured in `.mcp.json` with `N8N_API_URL` and `N8N_API_KEY`

## Key Tools
- `n8n_list_workflows` — list all workflows (with optional filters)
- `n8n_get_workflow` — get full workflow JSON
- `n8n_get_workflow_structure` — nodes + connections only
- `n8n_validate_workflow` — validate by workflow ID
- `n8n_list_executions` — view execution history
- `n8n_get_execution` — get execution details/errors
- `n8n_create_workflow` — create new workflows
- `n8n_update_partial_workflow` — incremental edits (15 operation types)
- `search_nodes` — search 537 node types
- `get_node_essentials` — node operation details

## Conventions
- Use `nodes-base.*` prefix for search/validate tools
- Use `n8n-nodes-base.*` prefix for workflow creation tools
- Build workflows iteratively, not in one shot
- Validate after every significant change
- Use smart parameters (branch="true", case=0) for IF/Switch connections

## n8n MCP Connectivity

### Two n8n MCP Servers
This project has access to two different n8n connectors:

| Server | Prefix | Pagination | Use For |
|--------|--------|-----------|---------|
| **Local n8n-mcp** | `mcp__n8n-mcp__*` | Full (cursor-based, all results) | All workflow operations (preferred) |
| **Anthropic-hosted** | `mcp__claude_ai_n8n__*` | Limited (~12 results, no pagination param) | Quick lookups only |

**Always prefer the local `n8n-mcp` server** for workflow operations. The Anthropic-hosted connector silently truncates results on instances with >12 workflows.

### Workflow Not Found? Fallback Steps
If a workflow search returns no results:
1. **Try the local n8n-mcp** (`n8n_list_workflows`) — it supports full pagination
2. **Direct REST API** as last resort:
   ```bash
   # List all workflows (paginated)
   curl -s -H "X-N8N-API-KEY: $N8N_API_KEY" "$N8N_API_URL/api/v1/workflows?limit=200"

   # Paginate with cursor
   curl -s -H "X-N8N-API-KEY: $N8N_API_KEY" "$N8N_API_URL/api/v1/workflows?limit=200&cursor=<nextCursor>"
   ```
3. **Never assume a workflow doesn't exist** just because one connector returned empty results

### .mcp.json Credential Management
- `.mcp.json` is **not committed** to the repo (see `.gitignore`) — it contains API keys
- After `git clone` or `git checkout`, verify `.mcp.json` still has `N8N_API_URL` and `N8N_API_KEY`
- Source of truth for credentials: `~/.env.shared`
- Template with placeholder values: `.mcp.json.example`
- The correct API URL: `https://im4tlai.app.n8n.cloud`

## Related Skills
The n8n-mcp-skills plugin provides expert guidance:
- n8n Expression Syntax
- n8n MCP Tools Expert
- n8n Workflow Patterns
- n8n Validation Expert
- n8n Node Configuration
