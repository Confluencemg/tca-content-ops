# Secrets Required for Routines

Credentials live in a private Notion page titled exactly **`Routine Vault — Credentials`**. Routines fetch them at runtime via Notion search + page fetch, parse the fenced code block, and hold the values in memory only. **Do NOT commit actual values to this repo** and **do NOT add them to the routine's environment variables** — the vault is the single source of truth.

The Notion connector must be attached to every routine that needs credentials. The fetch protocol is documented in `CLAUDE.md` → Routine architecture.

---

## Per-routine requirements

### blog-publish
- `WP_BASE_URL` — `https://thomasclarkadvisor.com`
- `WP_USERNAME` — WordPress username
- `WP_APP_PASSWORD` — WordPress application password (Users → Profile → Application Passwords)
- `OPENAI_API_KEY` — primary image gen (`gpt-image-2`)
- `OPENROUTER_API_KEY` — fallback image gen (`openai/gpt-5.4-image-2`)

### social-distribute
- `META_ACCESS_TOKEN` — Facebook + Instagram (Graph API token from Meta Business)
- `LINKEDIN_ACCESS_TOKEN` — LinkedIn API token
- `PINTEREST_ACCESS_TOKEN` — Pinterest API token
- `OPENAI_API_KEY` — primary image gen
- `OPENROUTER_API_KEY` — fallback image gen

### pinterest-queue-flush
- `PINTEREST_ACCESS_TOKEN`

### weekly-newsletter-digest
- `GHL_API_KEY` — GoHighLevel for sending the broadcast
- `GHL_LOCATION_ID` — TCA sub-account ID
- `OPENAI_API_KEY` — if any image generation is needed
- `OPENROUTER_API_KEY` — fallback image gen

Notion is reached via MCP connector, not via a stored token in the vault.

---

## Where to get each

| Secret | Source |
|---|---|
| `WP_BASE_URL` | Always `https://thomasclarkadvisor.com` |
| `WP_USERNAME` | thomasclarkadvisor.com → wp-admin → Users → All Users (Thomas's username) |
| `WP_APP_PASSWORD` | thomasclarkadvisor.com → wp-admin → Users → Profile → Application Passwords |
| `OPENAI_API_KEY` | platform.openai.com → API keys (org must be verified for `gpt-image-2`) |
| `OPENROUTER_API_KEY` | openrouter.ai → Keys |
| `GHL_API_KEY` | GHL → Sub-Account Settings → API Keys (TCA sub-account) |
| `GHL_LOCATION_ID` | GHL → Sub-Account Settings → URL contains the location ID |
| `META_ACCESS_TOKEN` | developers.facebook.com → app dashboard → Tools → Graph API Explorer |
| `LINKEDIN_ACCESS_TOKEN` | linkedin.com/developers → app → Auth → access token |
| `PINTEREST_ACCESS_TOKEN` | developers.pinterest.com → apps → access token |

---

## Updating the vault

To add or rotate a credential:

1. Open the `Routine Vault — Credentials` page in Notion.
2. Edit the fenced code block in place — keep the `KEY = value` format, one per line.
3. Save. Routines pick up the new value on their next run; no routine config change needed.
4. If a key is renamed, update the relevant `routines/*.md` files in the same commit.

---

## Bootstrap order for a new routine

1. Add any new keys to the vault.
2. Verify the vault parse works by manually running the routine once and confirming Step 1 succeeds.
3. Run the routine end-to-end against a test target (test WordPress draft, test GHL contact, etc.) before flipping the cron live.
