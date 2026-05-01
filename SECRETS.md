# Secrets Required for Routines

These are configured in Claude Code Routines settings, not in the repo. Do NOT commit actual values.

## Per-routine requirements

### blog-publish
- `WP_APP_PASSWORD` — WordPress application password for thomasclarkadvisor.com (Users → Profile → Application Passwords)
- `WP_USERNAME` — WordPress username
- `OPENAI_API_KEY` — for GPT Image 2 image generation
- `GHL_API_KEY` — GoHighLevel API key (for newsletter contact updates if needed)
- `NOTION_MCP_TOKEN` — for logging to Operations Hub

### social-distribute
- `META_ACCESS_TOKEN` — Facebook + Instagram (Graph API token from Meta Business)
- `LINKEDIN_ACCESS_TOKEN` — LinkedIn API token
- `PINTEREST_ACCESS_TOKEN` — Pinterest API token
- `OPENAI_API_KEY` — image generation
- `NOTION_MCP_TOKEN` — logging

### pinterest-queue-flush
- `PINTEREST_ACCESS_TOKEN`
- `NOTION_MCP_TOKEN` — logging

### weekly-newsletter-digest
- `GHL_API_KEY` — GoHighLevel for sending the broadcast
- `GHL_LOCATION_ID` — TCA sub-account ID
- `OPENAI_API_KEY` — if any image generation needed
- `NOTION_MCP_TOKEN` — logging

## Where to get each

| Secret | Source |
|---|---|
| WP_APP_PASSWORD | thomasclarkadvisor.com → wp-admin → Users → Profile → Application Passwords |
| WP_USERNAME | thomasclarkadvisor.com → wp-admin → Users → All Users (Thomas's username) |
| OPENAI_API_KEY | platform.openai.com → API keys |
| GHL_API_KEY | GHL → Sub-Account Settings → API Keys (use TCA sub-account) |
| GHL_LOCATION_ID | GHL → Sub-Account Settings → URL contains the location ID |
| META_ACCESS_TOKEN | developers.facebook.com → app dashboard → Tools → Graph API Explorer |
| LINKEDIN_ACCESS_TOKEN | linkedin.com/developers → app → Auth → access token |
| PINTEREST_ACCESS_TOKEN | developers.pinterest.com → apps → access token |
| NOTION_MCP_TOKEN | Already configured in Notion MCP — pull from existing setup |

## Configuration order

1. WordPress (test publish first)
2. OpenAI (test image generation)
3. Notion MCP (test logging)
4. GHL (test newsletter send to a test contact first)
5. Meta, LinkedIn, Pinterest (last — these have rate limits and reauth windows)

Each routine should be run manually once after secrets are configured, before activating its schedule.
