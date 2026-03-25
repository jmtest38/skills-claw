# Pica routing reference

## Routing order
1. Bitwarden Secrets Manager
2. Pica Passthrough API
3. Pica ToolKit or MCP
4. Pica web app
5. Browser automation or search fallback

## Secret naming heuristics
Normalize service names to uppercase and try both underscore and hyphen variants.

For API-first services, test:
- `API_KEY_<SERVICE>`
- `API_KEY-<SERVICE>`
- `<SERVICE>_API_KEY`
- `<SERVICE>_TOKEN`
- `<SERVICE>_KEY`
- `<SERVICE>_ACCESS_TOKEN`

For login-based services, test:
- `<SERVICE>_LOGIN`
- `<SERVICE>_PASSWORD`
- `<SERVICE>_USERNAME`
- `<SERVICE>_EMAIL`

## Duplicate resolution
Prefer in this order:
1. exact `API_KEY_` prefix
2. exact normalized service name
3. token/key variant
4. login/password pair

If multiple plausible matches remain, choose the most specific one and state that multiple matches existed without revealing values.

## Pica-specific notes
Use Pica when:
- direct service credentials are missing
- the target integration exists in Pica
- the task is better handled through a managed integration path

Likely Pica credential names:
- `API_KEY_PICAOS`
- `PICA_API_KEY`
- `PICA_SECRET_KEY`
- connection UUID if already known

## Reporting style
Allowed:
- “credential found”
- “integration available”
- “action completed”
- “fallback used”

Forbidden:
- secret values
- passwords
- full token strings
