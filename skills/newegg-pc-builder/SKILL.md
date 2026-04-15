---
name: newegg-pc-builder
description: >
  Connect to the Newegg PC Builder MCP service to retrieve PC build
  configurations, component compatibility checks, and build recommendations.
  Use this skill whenever the user asks about PC builds, custom PC
  configurations, component compatibility, budget builds, gaming rigs,
  workstation setups, or anything related to selecting or validating PC
  components on Newegg — even if they don't mention "PC Builder" by name.
  Also trigger when the user says things like "help me build a PC", "is this
  GPU compatible with my motherboard", "what parts do I need for a $1500
  gaming build", or "show me Newegg build configs".
---

# Newegg PC Builder MCP Skill

Connects Claude to the Newegg PC Builder MCP service. This skill is
**fully dynamic**: it discovers available tools at runtime and lets the LLM
decide which tool to call and how to fill its parameters. No tool names or
parameter names are hard-coded, so the skill continues to work even after
the MCP server updates its API.

**MCP Endpoint**: `https://apis.newegg.com/ex-mcp/endpoint/pcbuilder`
**Script**: `scripts/mcp_client.py`

---

## Core Workflow (always follow this order)

### Step 1 — Discover tools

Always start by listing available tools. Never assume tool names or parameters
from previous runs or documentation.

```bash
python scripts/mcp_client.py list_tools
```

The output contains, for each tool:
- `name` — identifier to use when calling
- `description` — what it does (may be empty; infer from name + schema)
- `inputSchema.properties` — available parameters with types and descriptions
- `inputSchema.required` — mandatory parameters

### Step 2 — Select tool and map parameters

Read the `list_tools` output and decide:

1. **Which tool best matches the user's intent?**
   - Match on description first; fall back to inferring from name + param names
   - If multiple tools apply, prefer the most specific one
   - If still ambiguous, pick the first one and note the assumption

2. **How does user intent map to parameters?**
   - Only use parameters present in `inputSchema.properties`
   - Free-text params (e.g. `question`, `query`, `text`): pass the user's
     request as a natural-language string describing their need
   - Typed/enum params: map user intent to the closest valid value
   - Leave optional params unset unless you have a clear value
   - Never invent parameters not present in the schema

### Step 3 — Call the tool

```bash
python scripts/mcp_client.py call <tool_name> '<json_arguments>'
```

Tool name and arguments are determined at runtime from Step 2. Example:
```bash
python scripts/mcp_client.py call v2allin '{"question": "gaming PC RTX 5090 9800X3D best price"}'
```

### Step 4 — Interpret the response

Parse the JSON output. Common response shapes:

```json
{ "result": { "summary": "...", "popular": [...], "valued": [...] } }
```

- `summary` non-null → use as the primary answer text
- `popular` / `valued` arrays present → list the builds
- All result fields null → service returned no data; tell the user and offer
  to answer from general knowledge instead
- `isError: true` → report the error and suggest retry

### Step 5 — Present results

- **Build list**: name, total price, key components, brief description
- **Compatibility check**: clear yes/no with reasoning
- **Component details**: specs, price, compatibility notes
- **No results from API**: explain clearly, then offer a manual recommendation

---

## Edge case handling

| Situation | Action |
|-----------|--------|
| `list_tools` returns 0 tools | Report service unavailable; answer from training knowledge |
| Tool description is empty | Infer purpose from name + parameter names |
| Schema has no `required` | Treat all params as optional; pass what you have |
| All response fields null | No data for this query; say so and fall back to general knowledge |
| HTTP error on `list_tools` | Report error; do not proceed to call step |
| HTTP error on `call` | Retry once; if still failing, fall back to general knowledge |
| Multiple tools match | Pick most specific; briefly note the choice |

---

## Script reference

`scripts/mcp_client.py` uses Python standard library only (3.6+, no pip needed).
Handles both `application/json` and `text/event-stream` responses automatically.

```
python scripts/mcp_client.py list_tools
    → prints all tools with full parameter schemas

python scripts/mcp_client.py call <tool_name> '<json_args>'
    → calls the tool and prints the JSON response
```

Errors go to stderr with a non-zero exit code.
