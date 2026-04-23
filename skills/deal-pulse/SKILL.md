---
name: deal-pulse
description: Surface the status of net-new (new logo) deals in progress. Searches prospect Slack channels, AE direct messages, Gong call summaries, and calendar to identify what's making progress, what's stalling, and the most important next steps per deal. Use when the user asks about pipeline, new logo deals, what deals need attention, or what's happening in sales.
---

# Deal Pulse

A skill for surfacing the current state of net-new (new logo) deals at Solv Health.

**Primary sources (in priority order):**
1. Slack — prospect channels, AE DMs, and war room
2. Gong — call summaries delivered via Gmail
3. Google Calendar — upcoming and recent deal-related meetings
4. Gmail — light fallback for follow-up threads not captured above

---

## Context

- **Company**: Solv Health — sells Experience, Connect, Pay, ClearPay, and Maya to urgent care chains and health systems
- **AE team**: Lyndsey Gootee, Danny Cavic (Daniel Cavic), Jennifer Condit, Brantley Sickeler, Andy
- **Gong calls**: Delivered to teresa@solvhealth.com from do-not-reply@gong.io; subject "Call recording and analysis is ready"; AI key points are in the email snippet/body

---

## Workflow

### Step 1 — Slack: Prospect Channels

Search each of these channel types. Use the Slack search tool with `in:#channel-name` to scope results. Run searches in parallel batches.

**Prospect-specific channels** (dedicated per account, highest signal):
- Search: `in:#prosp-` — this prefix pattern covers all active prospect channels
- Also search these explicitly: `in:#gsd-war-room-2026`, `in:#big-deal-pricing`
- For each active prospect channel found, use `slack_read_channel` to read the last 20–30 messages for full context

**AE direct messages** (where deal details, blockers, and strategy live):
Search DMs with each AE for deal updates from the last 14 days:
- `from:Lyndsey Gootee after:[14 days ago]`
- `from:Danny Cavic after:[14 days ago]` (also try "Daniel Cavic")
- `from:Jennifer Condit after:[14 days ago]`
- `from:Brantley Sickeler after:[14 days ago]`
- `from:Andy after:[14 days ago]` (filter to DMs with Teresa)

Look for: deal names, stage updates, blockers, competitor mentions, next steps, excitement or concern signals.

**War room, pricing, and sales channels**:
- `#gsd-war-room-2026` — AEs post daily deal updates here; read last 7 days
- `#big-deal-pricing` — pricing discussions, deal structures, approvals
- `#sales-only` — internal sales team discussion, strategy, deal flags
- `#sales-strategy-qs` — questions and debates on deal strategy, objection handling
- `#sales-contracting` — contract issues, redlines, legal escalations, DocuSign status

### Step 2 — Gong: Recent Call Summaries

Search Gmail for Gong call recordings from the last 14 days:
```
newer_than:14d from:do-not-reply@gong.io subject:"Call recording"
```

For each call that involves a prospect (not an internal meeting), extract from the snippet:
- Account name
- AE on the call
- Key points (pricing reactions, objections, excitement, competitor mentions, next steps)
- Call length — longer calls (30+ min) usually signal more serious engagement

Skip: daily digest emails ("Your team's opportunities today"), internal-only calls.

### Step 3 — Calendar: Upcoming Deal Meetings

Use `mcp__claude_ai_Google_Calendar__list_events` to pull events for the next 14 days. Look for:
- Meetings with prospect email domains (non-solvhealth.com attendees)
- Titles containing: demo, pilot, review, kickoff, proposal, pricing, contract, sync
- Note which AE owns each meeting

This surfaces what's actively being worked and what's imminent.

Also check the past 7 days for meetings that just happened — these represent recent touches that should have follow-up.

### Step 4 — Gmail (light fallback)

Only if a deal appears in Slack/Gong but context is thin, search for the specific account name in Gmail:
```
newer_than:14d [account name] -from:do-not-reply@gong.io
```

Fetch the most recent thread (`MINIMAL` format) to fill in gaps.

---

## Synthesis & Ranking

Rank deals by a combination of:
- **Recency of touch** — Slack message or Gong call in last 7 days = hot; 7–14 days = warm; 14+ days = stalling
- **Stage signal** — contract/DocuSign > pricing/proposal > ROI/demo > discovery
- **Sentiment** — excitement ("very excited", "asking twice a day") vs. friction ("pricing surprise", "legal review", silence)
- **Deal size** — larger TAM = higher priority when signals are equal
- **Upcoming calendar event** — imminent meeting bumps a deal up in priority

---

## Output Format

```
## Deal Pulse — [Date]

### 🔥 Sign-Ready / Closing
**[Account] | [AE] | [Est. MRR if known]**
- Stage: [what's happening]
- Signal: [key Slack/Gong quote or data point]
- Next meeting: [if on calendar]
- **Action**: [specific next step, owner, urgency]

### 🟠 High Momentum
**[Account] | [AE]**
- Stage: [e.g., ROI finalization / Pilot agreement pending]
- Signal: [what's driving momentum]
- Next meeting: [if on calendar]
- **Action**: [what to do next]

### 🟡 Progressing
**[Account] | [AE]**
- Stage: [e.g., Post-demo / Discovery]
- Signal: [what's encouraging or uncertain]
- Next meeting: [if on calendar]
- **Action**: [keep warm or specific push]

### 🔴 Stalling — Needs Intervention
**[Account] | [AE]**
- Last touch: [X days ago — channel]
- Blocker: [specific reason]
- **Action**: [what needs to happen to unblock, and who owns it]

---
### Priority Actions This Week
1. [Most urgent — account, owner, why now]
2. ...
```

---

## Signal Reference

| What you see | What it means |
|---|---|
| AE posted update in #gsd-war-room-2026 today | Active deal, check for blockers |
| #prosp-[account] channel has recent messages | Deal being actively worked |
| Gong call 30+ min with key points | Serious engagement |
| "Asked for contract" or "DocuSign" in Slack | Close-ready |
| Pricing discussion in #big-deal-pricing | Deal is real, moving |
| AE DM: "stuck", "no response", "waiting on legal" | Stalling — intervene |
| Calendar: prospect meeting in next 7 days | Prep opportunity |
| Calendar: meeting happened, no Slack follow-up posted | Follow-up may be missing |
| Gong key point: competitor mentioned | Flag immediately |
| No Slack, no Gong, no calendar for 14+ days | Dead or dormant |

---

## Notes

- **Slack OR syntax doesn't work** — search one account or channel at a time
- **Gong key points** are usually in the email snippet even for HTML-only emails — the snippet is sufficient
- **New logos only** — exclude existing Solv partners (NextCare, Carbon Health, MultiCare, ZoomCare, CCP, Exer, Bon Secours, GoHealth, ZoomCare). Those are tracked separately
- If fewer than 5 active deals surface, note that and suggest checking Salesforce directly
