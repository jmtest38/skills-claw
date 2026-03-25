---
name: pica-gateway
description: use when chatgpt needs to decide autonomously how to access and act on third-party apps or websites, especially notion, telegram, crms, saas tools, and any integration that may be reachable via existing secrets, pica passthrough api, pica toolkit/mcp, or browser automation. triggers include requests like "use pica for notion", "connect autonomously", "fill notion", "contact x", "sync this tool", or any task where chatgpt should first check bitwarden secrets, then use pica if available, then fall back to browser/search without exposing any api key, password, token, or secret value.
---

# Pica Gateway

Route external-service work through the most direct safe path:
1. Existing credentials from Bitwarden Secrets Manager
2. Pica (Passthrough API first, ToolKit/MCP if relevant, web app only as last resort)
3. Browser/search fallback

Never ask the user to choose between these paths unless a real blocker remains after trying them in order.

## Core rules

- Never print or quote any secret value.
- Confirm only presence, absence, name, availability, action success, or failure reason.
- Prefer direct authenticated API access over browser automation.
- Use Bitwarden first whenever a task mentions a service, app, site, SaaS tool, CRM, or connector.
- If Bitwarden does not provide a usable credential, try Pica automatically.
- Use browser automation only when Bitwarden and Pica do not cover the action.
- If browser automation fails because of SPA rendering, selectors, or UI complexity, prefer an existing visual/browser skill if available.
- If the task still cannot be completed, explain the blocker briefly and name the next best path.

## Decision workflow

### Step 1: Infer the target service and likely action
Infer the service from the task itself.

Examples:
- “utilise Pica pour remplir Notion” → service = Notion, action = create/update pages or database rows
- “contacte azer de manière autonome” → infer likely CRM, outreach tool, email, linkedin, telegram, or other contact system from context
- “utilise Telegram pour m’envoyer le résultat” → service = Telegram, action = send message

Infer likely credential types from the service:
- API work: `API_KEY`, `TOKEN`, `SECRET`, `ACCESS_TOKEN`, `BOT_TOKEN`
- Account login: `LOGIN`, `USERNAME`, `EMAIL`, `PASSWORD`
- Vendor-specific names when obvious

### Step 2: Check Bitwarden first
Use the Bitwarden skill first.

Requirements:
- use `bws`, never `bw`
- search flexibly with name variants
- prefer exact names starting with `API_KEY_`
- allow underscore and hyphen variants
- detect duplicates and choose the most specific result

Priority rules:
1. Exact match beginning with `API_KEY_`
2. Exact match with the normalized service name
3. Closest equivalent token/key name
4. Login/password pair if API access is unavailable

Examples for Notion:
- `API_KEY_NOTION`
- `API_KEY-NOTION`
- `NOTION_API_KEY`
- `NOTION_TOKEN`
- `NOTION_KEY`

Examples for Telegram:
- `API_KEY_TELEGRAM`
- `TELEGRAM_API_KEY`
- `TELEGRAM_BOT_TOKEN`
- `BOT_TOKEN_TELEGRAM`

Examples for BeatStars:
- `BEATSTARS_LOGIN`
- `BEATSTARS_PASSWORD`
- `BEATSTARS_USERNAME`
- `BEATSTARS_EMAIL`

When secrets are found:
- state only that the required credential is available
- use it directly in the workflow
- do not ask the user to copy/paste it

## Step 3: Use Pica automatically when direct credentials are missing or less suitable
If Bitwarden does not provide a usable route, try Pica automatically.

Preferred order inside Pica:
1. Passthrough API
2. ToolKit or MCP if Passthrough is not the best fit
3. Pica web app only if API-style access is not available

For Pica access, look for likely credentials such as:
- `API_KEY_PICAOS`
- `PICA_API_KEY`
- `PICA_SECRET_KEY`
- user-provided Pica UUID or connection identifier when relevant

Use Pica to:
- inspect available integrations
- determine whether the target service is supported
- execute the action directly when possible
- report only availability and action outcome, never secret values

When Pica is available but the requested action is unsupported:
- say the integration exists but the action appears unavailable
- fall through to browser/search if that remains viable

## Step 4: Browser/search fallback
Use browser/search only after Bitwarden and Pica have been tried or ruled out.

Preferred fallback order:
1. Existing browser automation skill
2. Web search skill such as Brave-based search if browsing is enough
3. Visual/web-app fallback only if actual page interaction is required

Use browser/search for:
- public discovery
- sites that require rendered SPA pages
- cases where the user wants public results gathered then written into another tool

Do not default to browsing when direct APIs or Pica can do the job.

## Autonomy rules

Act autonomously by default for:
- checking Bitwarden
- checking Pica
- choosing the best route
- executing the action
- sending the result through the requested channel

Pause only for real blockers such as:
- 2FA or captcha that cannot be solved by the available tools
- no credential and no supported integration path
- ambiguous destructive action
- missing destination details that cannot be inferred safely

## Output rules

When reporting progress or final status:
- never reveal secret values
- be brief
- say what route was chosen
- say whether the action completed
- mention missing pieces only if they truly blocked execution

Good examples:
- “Notion API credential found and used successfully.”
- “Pica integration for Notion is available and the page was created.”
- “No usable BeatStars API credential was found; browser fallback used instead.”
- “Telegram bot token found and message sent.”

Bad examples:
- printing any api key
- pasting passwords
- asking the user to choose between bitwarden and pica before trying both

## Default behavior examples

### Example: Notion write request
User: “fais de la prospection d'artistes rap français et remplis Notion”

Behavior:
1. infer Notion is a target service
2. search Bitwarden for Notion credentials
3. use direct Notion API if credential exists
4. otherwise try Pica’s Notion integration
5. otherwise use browser/search to collect data and the best remaining write path
6. optionally send results to Telegram if requested

### Example: BeatStars discovery then CRM update
User: “cherche sur BeatStars 10 artistes et remplis Notion”

Behavior:
1. search Bitwarden for BeatStars credentials and Notion credentials
2. use direct APIs if available
3. if BeatStars is not directly accessible, try Pica support
4. if still blocked, use browser/search for discovery
5. write structured results into Notion

### Example: service availability check
User: “utilise Pica pour remplir Notion”

Behavior:
1. check Bitwarden first for direct Notion access
2. if no direct path, inspect Pica integration availability
3. execute directly if available
4. reply with route used and outcome only

## Reference
See `references/pica-routing.md` for compact routing guidance and naming conventions.
