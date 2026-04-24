# Deploy Deal Pulse

End-to-end skill: pulls fresh pipeline data, writes the JSON, builds the Next.js dashboard, and deploys to Railway.

**Dashboard project**: `/Users/teresa/claude class/deal-pulse-dashboard/`
**Data file (root)**: `/Users/teresa/claude class/deal-pulse-data.json`
**Data file (dashboard)**: `/Users/teresa/claude class/deal-pulse-dashboard/public/deal-pulse-data.json`

---

## Workflow

### Step 1 — Pull fresh deal data

Execute the full deal-pulse workflow exactly as defined in `.claude/skills/deal-pulse/SKILL.md`:
- Search Slack (prospect channels, AE DMs, war room, contracting)
- Search Gmail for Gong call summaries (last 14 days)
- Check Google Calendar for upcoming deal meetings (next 14 days)
- Gmail fallback for thin deals

Synthesize and rank all deals. Then write the output to `/Users/teresa/claude class/deal-pulse-data.json` using this exact JSON schema:

```json
{
  "generated_at": "ISO 8601 timestamp",
  "summary": {
    "total": <number>,
    "sign_ready_closing": <number>,
    "high_momentum": <number>,
    "progressing": <number>,
    "stalling_needs_intervention": <number>
  },
  "deals": [
    {
      "account": "string",
      "ae": "string or null",
      "category": "sign_ready_closing | high_momentum | progressing | stalling_needs_intervention",
      "stage": "string — one-line description of where the deal is",
      "estimated_mrr": <number or null>,
      "last_touch": {
        "date": "YYYY-MM-DD",
        "source": "Slack / Gong / Email / Calendar",
        "detail": "verbatim quote or key detail from that touch"
      },
      "next_meeting": { "date": "YYYY-MM-DD", "title": "string" } | null,
      "deal_signals": ["string", ...],
      "blockers": ["string", ...],
      "recommended_actions": ["string", ...]
    }
  ],
  "priority_actions_this_week": [
    {
      "priority": <number>,
      "account": "string",
      "action": "string",
      "owner": "string",
      "urgency": "today | tomorrow | this_week",
      "why": "one sentence explaining the urgency"
    }
  ]
}
```

Rules:
- New logos only. Exclude existing Solv partners: NextCare, Carbon Health, MultiCare, ZoomCare, CCP, Exer, Bon Secours, GoHealth.
- Use `null` for unknown fields. No markdown or commentary in JSON values.
- `generated_at` = current timestamp in ISO 8601 UTC format.
- Sort deals by category order: sign_ready_closing → high_momentum → progressing → stalling_needs_intervention.

### Step 2 — Sync data to dashboard

Copy the updated JSON into the dashboard's public directory:

```bash
cp "/Users/teresa/claude class/deal-pulse-data.json" "/Users/teresa/claude class/deal-pulse-dashboard/public/deal-pulse-data.json"
```

Confirm the copy succeeded before proceeding.

### Step 3 — Check Railway is linked

```bash
cd "/Users/teresa/claude class/deal-pulse-dashboard" && railway status
```

If the output contains "No linked project found", stop and tell the user:

> Railway is not linked. Run this in your terminal to connect to a project:
> `! railway link`
> Then re-run `/deploy-deal-pulse`.

If linked, continue.

### Step 4 — Build the dashboard

```bash
cd "/Users/teresa/claude class/deal-pulse-dashboard" && npm run build 2>&1
```

If the build exits with a non-zero code or prints errors, stop and report the error. Do not deploy broken code.

### Step 5 — Deploy to Railway

```bash
cd "/Users/teresa/claude class/deal-pulse-dashboard" && railway up --service deal-pulse-dashboard --detach 2>&1
```

`--detach` returns immediately after upload without waiting for the build to finish on Railway's side. Railway will build and deploy the app in the background.

### Step 6 — Report

Output a summary in this format:

```
## Deal Pulse Deployed

**Data as of**: <generated_at formatted as human-readable>
**Railway project**: <project name from railway status>
**URL**: https://deal-pulse-dashboard-production.up.railway.app

### Pipeline snapshot
- 🔥 Sign-Ready: <n>
- 🟠 High Momentum: <n>
- 🟡 Progressing: <n>
- 🔴 Stalling: <n>
- **Total**: <n>

### Priority actions
1. <account> — <action> (<owner>, <urgency>)
...
```

---

## Error handling

| Situation | What to do |
|---|---|
| Slack / Gong returns no results for a deal | Use last known data from existing JSON for that deal; note it |
| Build fails with type error | Fix the type error in the dashboard source, then retry build |
| `railway up` fails with auth error | Tell user to run `! railway login` |
| `railway up` fails with no project | Tell user to run `! railway link` |
| `railway up` says "Multiple services found" | Always pass `--service deal-pulse-dashboard` flag |
| Railway deployment URL not yet available | Provide the Railway dashboard URL: https://railway.app/dashboard |
