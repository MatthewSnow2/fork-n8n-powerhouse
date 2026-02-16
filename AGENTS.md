# n8n-powerhouse - Operational Guide

## How to Build/Run

This is an MCP configuration project — no build step required.

### Start
```bash
cd ~/projects/n8n-powerhouse
claude
```

### Verify MCP Server
Once in Claude Code, the n8n-mcp server should auto-start. Verify by listing workflows.

## Testing
- Verify API connectivity by listing workflows
- Test with `n8n_get_workflow` on a known workflow ID

## Common Operations

### Troubleshoot a workflow
1. `n8n_list_workflows` to find the workflow
2. `n8n_get_workflow` to inspect nodes and connections
3. `n8n_list_executions` to check recent execution history
4. `n8n_get_execution` to view error details

### Build a new workflow
1. `search_nodes` to find needed nodes
2. `get_node_essentials` for configuration details
3. `n8n_create_workflow` to create
4. `n8n_validate_workflow` to verify
5. `n8n_update_partial_workflow` to iterate

### Validate existing workflow
1. `n8n_validate_workflow` with workflow ID
2. Review errors and warnings
3. `n8n_update_partial_workflow` to fix issues
