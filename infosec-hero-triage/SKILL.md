---
name: infosec-hero-triage
description: >
  Run the daily InfoSec Hero escalation triage for Qualia's security team. ALWAYS use this
  skill when the user says "Avengers Assemble" (any capitalization), or asks to "run hero
  triage", "pull the security queue", "check my hero channels", "what's in the security
  inbox", "triage today's escalations", or anything similar about the InfoSec Hero rotation,
  security@ email queue, #security, or #support-security. The goal: every inbound escalation
  is looked at and acknowledged within 2 business hours, with each item linked back to its
  source so it's traceable.
---

# InfoSec Hero Daily Triage ("Avengers Assemble")

Pull the live escalation queue across all Hero channels, classify every item per the
[InfoSec Hero Guidelines](https://docs.google.com/document/d/1pt0KOg27TEVSv_V14zZx-OOupNWcYf-gwg8r4WJcWqI/edit),
and produce a triage board where every item links back to its source. **Never send any
reply, email, or Slack message automatically** — recommend and draft only; the Hero sends.
The single exception: a **Slack DM to the Hero themself** as an alert channel (see
off-hours discipline below) — that is a notification to the user, not a reply to anyone.

## Step 0 — Arm the recurring watch (on "Avengers Assemble" only)

When the user opens with **"Avengers Assemble"** (any capitalization), they are starting a
Hero shift, not asking for a one-off sweep. Do both, in this order:

1. **Run the full triage now** (Steps 1–5 below) so they start the shift with a current board.
2. **Arm the business-hours watch** with `CronCreate`:
   - `cron`: `7,22,37,52 9-16 * * 1-5` — every 15 minutes, 9:07am through 4:52pm, weekdays.
     The off-minutes are deliberate (they keep the job off the :00/:15/:30/:45 marks every
     other scheduled job in the world lands on); do not "tidy" them.
   - `recurring`: `true`
   - `prompt`: `Run the infosec-hero-triage skill as a scheduled Hero check. Sweep all three
     channels. Notify only on CHANGE per the notification discipline in the skill — a new
     unacked item, an item newly inside 30 minutes of its 2-business-hour deadline, an item
     newly breaching it, or an apparent active incident. Do not re-notify about items already
     reported earlier this shift. If nothing changed, print the board and stay silent.`
3. **Arm the off-hours watch** (nights + weekends, every 30 minutes) with two more
   `CronCreate` jobs so the windows don't overlap:
   - Nights, all days: `cron`: `9,39 0-8,17-23 * * *`
   - Weekend daytime: `cron`: `9,39 9-16 * * 0,6`
   - `recurring`: `true` on both, and the same prompt for both:
     `Run the infosec-hero-triage skill as an OFF-HOURS Hero check. Narrow sweep of all
     three channels. Follow the off-hours discipline in the skill: alert ONLY on a genuinely
     new inbound item or an apparent active incident — no SLA ladder, no clocks. On an alert,
     send the macOS banner AND a Slack DM to the Hero themself. If nothing new, print the
     one-line quiet board and stay silent.`
4. Report all returned job IDs back to the user and keep them for Step 0b.
5. **Tell them plainly**: the watch is session-only (dies when this Claude Code session
   closes — re-arm each morning with "Avengers Assemble"), and overnight coverage also
   requires the Mac to stay awake: plugged in, with System Settings → Battery →
   "Prevent automatic sleeping on power adapter when the display is off" enabled
   (or `caffeinate -is` running). Remind them of this once per shift, not every run.

Any other trigger phrase ("run hero triage", "pull the security queue", etc.) means a one-off
sweep — run Steps 1–5 and do **not** arm anything.

### Step 0b — Stopping

When the user says **stop**, **stand down**, **Avengers disassemble**, **end shift**, or
otherwise indicates the shift is over: call `CronList` to find **all** hero jobs (business
hours + both off-hours jobs), `CronDelete` each by ID, and confirm in one line that the
watch is off. If no jobs are armed, say so rather than pretending to cancel one.

Then **print the EOD closeout checklist** (doc step 5) from the shift ledger — this is
automatic on stand-down, not optional:

```markdown
**Watch off** (job deleted).
# 📋 EOD closeout
- [ ] Jira → Done: <every ticket the ledger marked ready to close>
- [ ] <open containment / follow-up actions on in-motion items>
- [ ] Unanswered requesters: <anyone the team asked a question and never heard back from>
- [ ] kb: add lessons for <today's new SEC tickets>, rebuild
Then: archive? (y/n)
```

If the user says yes to archiving (or asked to "back up" in the same breath) **and** an
archive/backup skill is available in this session (e.g. archive-chat), run it. If no such
skill is installed, skip the archive offer entirely — don't ask.

### Notification discipline on scheduled runs

The watch runs every 15 minutes — roughly 32 times a day. **Notify on change, never on state.**
An item that was already reported is not news 15 minutes later; re-pinging it would train the
Hero to ignore the notifications entirely, which defeats the whole point.

**Keep a running shift ledger** in the conversation: for each open item, its source link and
which of these notification stages it has already triggered. Fire a `PushNotification` only
when an item crosses into a stage it has not yet triggered:

1. **New** — an unacked item appeared that was not in the previous run's board.
2. **Approaching** — newly within 30 minutes of its 2-business-hour deadline.
3. **Breached** — newly past the deadline.
4. **Incident** — looks like an active incident (executed wire fraud, confirmed compromise,
   ongoing attack). Always notify, immediately, regardless of clock.

**Classify before you start a clock.** #security especially is high-noise. Before treating
a new item as stage-1, actually look at what was posted (open the attachment/screenshot if
that's where the substance is). If it clearly asks for help or reports a problem → full
stage ladder. If it reads as awareness-share, chatter, or is genuinely ambiguous → mark it
**tentative**: one soft ping/banner asking the Hero to classify it ("escalation" or
"informational"), track it on the board, but do **not** run the approaching/breached ladder
until the Hero confirms it's an escalation. A user reclassification ("that's informational")
immediately cancels the clock and any pending pings for that item.

An item sitting unacked at stage 2 for an hour produces **one** notification, not four. When
several items cross in the same run, send **one** notification covering them, not one each.

Stay silent — print the board only — when: nothing changed, everything open has already been
reported at its current stage, or the queue is clear. **Never** notify to say the queue is
clear or that a run completed.

Keep the message under 200 characters, one line, no markdown, leading with the thing to act on:
`2 unacked, oldest 1h50m: ORTC wire question in #support-security`. Not `triage complete`.

### Off-hours discipline (nights and weekends)

Outside business hours the SLA clock is not running — an item that lands at 11pm has its
2-business-hour clock start at 9am the next business day. So off-hours runs are simpler
and stricter:

- Alert **only** on: a genuinely new inbound item, or an apparent active incident.
  No approaching/breached stages, no tentative pings — chatter waits for morning.
- Each alert states the real deadline: `clock starts 9:00am → ack by 11:00am`.
- Alert delivery off-hours = macOS banner **plus a Slack DM to the Hero via their alert
  webhook** (the phone channel). A plain Slack DM-to-self does NOT work — Slack never
  push-notifies you about your own messages — so alerts go through a Slack Workflow
  webhook whose bot message does notify. The Hero's webhook URL lives in
  `~/.claude/hero-slack-webhook.url` (chmod 600; it is a capability URL — treat it as a
  credential, never commit or print it). Send with:

  ```bash
  curl -s -X POST "$(cat ~/.claude/hero-slack-webhook.url)" \
    -H 'Content-Type: application/json' -d '{"text":"<one-line alert with source link>"}'
  ```

  If the file is missing, fall back to emailing the user at their own address via Gmail
  `send_message` (subject = the alert line), and tell them once how to set up the webhook
  (Slack Workflow Builder → trigger "From a webhook" with a `text` variable → step "Send a
  message" to themselves → publish → save the URL to that path). Never message anyone
  else, never post in a channel.
- Quiet off-hours runs print the one-line 🟢 board and send nothing anywhere.
- Active incidents (executed wire fraud, confirmed compromise, ongoing attack) ignore all
  of this: banner + DM + push immediately, any hour.
- Slack-DM alerts also apply during business hours for **incident-stage** items only.

**Every push also posts a native macOS banner** so it lands even when the terminal is
buried. Run via Bash, same discipline (only on stage crossings, never on quiet runs):

```bash
osascript -e 'display notification "<same one-line message>" with title "InfoSec Hero" subtitle "<stage: New unacked | Under 30 min | BREACHED | INCIDENT>" sound name "Ping"'
```

Escape any double quotes in the message. For a breach or incident use sound name "Sosumi"
instead of "Ping" so it's audibly different. If osascript errors or the banner doesn't
appear, mention once that the terminal app may need Notification permission
(System Settings → Notifications) — don't retry repeatedly.

### Scheduled-run output format (the board in the terminal)

The terminal renders markdown, not ANSI color — emoji + headers are the color. Visual
hierarchy: 🟢 quiet · 🔵/✅ FYI change · 🟠 tentative or warning · 🔴/⛔ act now. Anything
red/stop also gets a push; everything else is board-only. Use exactly these shapes:

**Quiet run (nothing changed)** — the entire output is ONE line, no headers, no sections:

```markdown
🟢 **12:22pm** — no change · 0 unacked · in motion: SEC-1515 (you), Grace L. (Daniel)
```

If a clock is ticking, keep it visible in the same line:
`🟢 12:22pm — no change · ⏱ Marlon unacked, 1h20m left (1:42pm)`

**New escalation (confirmed)** — banner first, everything else below a divider:

```markdown
# 🚨🔴 NEW — <short title>
**<who> · <channel> · <time>** → [jump to thread](<link>)
<1–2 lines: what it is, why it matters>. **Ack by <deadline>.**
**Do:** <first action> → <second> → <third>. Draft: *"<copy-paste ack>"*

---
Rest of board: 🟡 <in-motion, names only> · ✅ EOD: <tickets>
```

**New but tentative** (borderline / #security chatter — see classification rule above):

```markdown
# 🟠❓ NEW (tentative) — <short title>
**<who> · <channel> · <time>** → [jump](<link>) · <attachment note>
<why it reads as informational>. **Reply "escalation" or "informational" to set/skip the
2h clock.** Until then it's tracked without warning pings.
```

**30-minute warning:**

```markdown
# ⏰🟠 UNDER 30 MIN — <item>
Still zero replies since <arrival>. **Deadline <time>.** One-line thread ack stops the
clock → [jump](<link>)
```

**Breach:**

```markdown
# 🚨⛔ BREACHED — <item>
2h SLA passed at <time>, no team reply. **Ack it or hand off in #infosec-private now**
→ [jump](<link>)
Final ping for this item — it stays pinned at the top of every board until owned.
```

**Change that isn't an alert** (pickup, resolution, reply on an owned item) — no banner,
one or two ✅/🔵 lines, then the quiet line:

```markdown
✅ **2:44pm — SEC-1515 thread closed by Marlon.** Ticket stays open: containment pending.
🟢 Everything else unchanged.
```

The **full three-section board** (Step 5) is printed only at shift start, at stand-down,
when the user asks ("show board"), or when a run has multiple simultaneous changes that
don't fit the shapes above. Never print it just because a scheduled run fired.

## Authoritative references

- Triage rules: [InfoSec Hero Guidelines (Google Doc)](https://docs.google.com/document/d/1pt0KOg27TEVSv_V14zZx-OOupNWcYf-gwg8r4WJcWqI/edit) — re-read via Google Drive `read_file_content` (fileId `1pt0KOg27TEVSv_V14zZx-OOupNWcYf-gwg8r4WJcWqI`) if rules seem to have changed.
- Weekly tasks: [Information Security Runbook](https://qualialabs.atlassian.net/wiki/spaces/SEC/pages/1073315853/Information+Security+Runbook)
- Incidents: [Security Incident Response Process](https://qualialabs.atlassian.net/wiki/spaces/SEC/pages/223412333/Security+Incident+Response+Process)
- Tracking: [ISEC Jira board](https://qualialabs.atlassian.net/jira/software/c/projects/ISEC/boards/48)
- Anything not covered → point the requester to the Runbook; don't guess.

## Step 1 — Determine the triage window

The Hero role is not on-call. Compute the window from today's date:

- **Tue–Fri:** everything since ~17:00 local on the previous business day, plus anything
  older that is still unacknowledged.
- **Monday:** everything since Friday ~17:00 local, **including the whole weekend** (the
  new week's Hero owns weekend arrivals).
- Always also sweep for older items with **no pickup signal** (see Step 3) — those have
  fallen through the cracks and outrank new arrivals.

## Step 2 — Pull all three channels (live data, every run)

**Opening sweep** (shift start, or any one-off run):

| Channel | How to pull |
|---|---|
| Email security@qualia.com | Gmail `search_threads` with query `to:security@qualia.com newer_than:3d` (use `newer_than:5d` on Mondays). For any thread needing classification detail, `get_message` with `PLAIN_TEXT`. |
| Slack #security (public) | `slack_read_channel` on channel ID `C0G8KAJQY` (verify with `slack_search_channels` if not found). |
| Slack #support-security (private) | `slack_read_channel` on channel ID `C030HNH43UN`. |

**Scheduled runs after the opening sweep — narrow the pull.** The full 3-day window is
mostly identical data every 15 minutes. Instead:

- Gmail: `to:security@qualia.com newer_than:1d`.
- Slack: same channel reads but with `oldest` set to the previous run's check time minus
  ~5 minutes of slack (Slack ts format).
- The shift ledger carries every older item that is still open — the narrowed sweep only
  needs to detect *changes*; open items never fall off the board because they aged out of
  the query window. To re-check an older item's thread for replies, `slack_read_thread`
  it directly by its stored ts.

Use `detailed` response format on Slack reads so you get `Message TS` and thread reply
counts. Read the threads (`slack_read_thread`) of every message in the window whose reply
count changed — pickup status lives in the replies.

## Step 3 — Determine pickup status per item

An item is **picked up** only on a positive signal:

- A substantive reply from an InfoSec team member in the email thread or Slack thread, or
- A Jira ticket (SEC-/ISEC-) created from the message with an assignee.

A 👀 reaction, a "thanks", or the requester's own follow-ups do **not** count. Per the
Guidelines doc, the Hero retains ownership until handoff is positively confirmed.

Compute **age against the 2-business-hour clock** (clock starts when the message landed,
or at start of business day if it landed after hours / on a weekend). Flag anything
unacknowledged and past — or within 30 minutes of — the deadline.

## Step 4 — Classify each open item per the Guidelines doc

Check escalation triggers first: abusive language or legal threats → escalate to manager
immediately (doc step 2.1.2), regardless of channel. Then:

| Channel | Type | Action (doc ref) |
|---|---|---|
| Email | Due diligence / vendor review | Forward to compliance@qualia.com, original recipients moved to **BCC** (2.1.3) |
| Email | Confidently answerable | Reply with the answer (2.1.4) |
| Email | Needs investigation | Reply, original list **CC'd**, confirming receipt + being looked into (2.1.5) |
| Slack | Due diligence / vendor review | Tell user to email compliance@qualia.com (2.2.1) |
| Slack | Confidently answerable | Answer **in a thread** (2.2.2) |
| Slack | Needs investigation | Ack **in a thread**: received + being looked into (2.2.3) |

Also flag, per step 3–4 of the doc:
- Looks like an **active incident** (executed wire fraud, confirmed compromise, ongoing
  attack) → route to the Security Incident Response Process, not routine triage.
- Will consume significant time → recommend logging on the ISEC board.
- Belongs elsewhere → legal@qualia.com or #ask-infra; teammate help → #infosec-private.
- Suspicious wires, compromised credentials, executed malware, exfiltration, lost devices
  → these are active-incident territory per org policy (security@qualia.com owns them).
- If anyone asks whether a message is a **phishing simulation**: never confirm or deny;
  direct them to report via the Phish Alert Button (orange hook icon in Gmail).

## Step 5 — Output the triage board

Render three sections, every item carrying a **source link**:

1. **🔴 Needs Hero action now** — unacknowledged or awaiting a Hero reply. For each:
   who/what/when, classification + doc step, time remaining on (or overrun of) the
   2-business-hour clock, recommended action, and a suggested draft reply the Hero can
   copy. Order by clock urgency.
2. **🟡 In motion — monitor** — picked up by a named owner. Show owner, Jira link, and
   what the next checkpoint is.
3. **✅ Closed / verify at end-of-day** — resolved items to confirm during the daily
   end-of-day sweep (doc step 5).

Then a one-line **bottom line**: what needs the Hero within the next hour.

**Source link formats:**
- Slack message: `https://goqualia.slack.com/archives/<CHANNEL_ID>/p<TS-with-dot-removed>`
  (e.g. ts `1787352135.175549` → `p1787352135175549`)
- Gmail thread: `https://mail.google.com/mail/u/0/#all/<threadId>`
- Jira: `https://qualialabs.atlassian.net/browse/<KEY>`
- Also carry through any Zendesk/deployment links the reporter included.

## Step 6 — Offer, don't act

End by offering to (a) draft the pending replies (Gmail draft / Slack thread draft) for
review, and (b) create ISEC/SEC Jira tickets for anything sizable and untracked. Only do
these on explicit confirmation. If the user says a different teammate is Hero today,
frame recommendations for handoff (post in #infosec-private) rather than direct action.

## Edge cases

- A Slack channel read fails with `channel_not_found` → re-resolve the ID via
  `slack_search_channels` (names: `security`, `support-security`) before giving up.
- Zero new items → still run the unacknowledged-item sweep and say explicitly that the
  queue is clear, listing the newest item checked per channel with its link.
- #security is high-noise (articles, chatter): only treat messages as escalations if they
  ask for help, report a problem, or request something of the team.
- Sensitive data (account numbers, wire details, credentials) may appear in escalations:
  summarize it, never repeat full account/routing numbers in the board, and never move it
  to unapproved tools.
