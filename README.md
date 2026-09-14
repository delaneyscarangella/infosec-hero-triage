# InfoSec Hero Triage — Claude Code skill ("Avengers Assemble")

A Claude Code skill that runs the daily InfoSec Hero escalation triage: it sweeps the
security@ mailbox, #security, #support-security, and #idr-alerts (Rapid7 SIEM
investigations posted by InsightConnect), builds a triage board where every item links
back to its source, tracks the 2-business-hour ack SLA per the
[InfoSec Hero Guidelines](https://docs.google.com/document/d/1pt0KOg27TEVSv_V14zZx-OOupNWcYf-gwg8r4WJcWqI/edit),
and — when you start a shift with **"Avengers Assemble"** — arms a recurring watch
(every 15 min, weekdays, until you say **disassemble**) that notifies
**only on change** (new unacked item, 30-min SLA warning, breach,
apparent incident). Alerts arrive three ways: terminal push, a banner-first board, and a
native macOS Notification Center banner with distinct sounds for normal vs. breach.
Incident-stage items also arrive as a Slack DM on your phone — via a personal Slack
Workflow webhook, because a plain DM-to-self never triggers a Slack notification.

The shift is what you say it is. Per the Guidelines the Hero is not on-call, so there are
no night or weekend jobs: if you stay assembled past 5pm the watch just keeps running
normally; once you disassemble nothing runs, and anything that lands overnight or over the
weekend gets its 2-business-hour clock at 9am the next working day.

It never sends anything on your behalf — it drafts replies and recommends actions; the
Hero sends.

## Install

```bash
mkdir -p ~/.claude/skills
cp -R infosec-hero-triage ~/.claude/skills/
```

Restart (or start) a Claude Code session and say **"Avengers Assemble"**.

## Prerequisites

- **Claude Code** with the claude.ai **Gmail** and **Slack** connectors enabled
  (Settings → Connectors), signed into your Qualia Google account and the goqualia
  Slack workspace.
- **Delegated access to the security@qualia.com mailbox** — the Gmail sweep runs as you,
  so mail to security@ must be visible in your own Gmail (it is for InfoSec team members).
- **Membership in #support-security and #idr-alerts** (both private) — the Slack
  connector can only read channels your account is in.
- For phone alerts: a personal Slack Workflow webhook — in Slack: Tools → Workflow
  Builder → New Workflow → trigger "From a webhook" (add a Text variable named `text`) →
  step "Send a message" to yourself → Publish. Save the generated
  `hooks.slack.com/triggers/...` URL to `~/.claude/hero-slack-webhook.url` and
  `chmod 600` it. Treat the URL like a credential (anyone holding it can DM you); never
  commit it. Without it, alerts fall back to email-to-self.
- For a long or late shift: the Mac must stay awake or the watch silently stops firing —
  keep the lid open, or run `caffeinate -is`.
- macOS for the Notification Center banners (`osascript` — built in, nothing to install).
  On first use, allow notifications for your terminal app if prompted
  (System Settings → Notifications).

## Usage

| You say | It does |
|---|---|
| `Avengers Assemble` | Full triage board + arms the watch (every 15 min, weekdays, until you disassemble) |
| `run hero triage` / `pull the security queue` | One-off sweep, no watch |
| `show board` | Reprints the full three-section board mid-shift |
| `that's informational` (about an item) | Reclassifies it, cancels its SLA clock and pending pings |
| `Avengers disassemble` / `stand down` | Kills the watch + prints the EOD closeout checklist |

**The watch is session-only.** It lives in your Claude Code session's memory — closing the
session kills it silently. Re-arm each shift with "Avengers Assemble". Notifications only
fire while the session is open.

## What the notifications look like

- Quiet runs print one line (`🟢 12:22pm — no change …`) and never notify.
- Stage crossings notify once each, at most three times per item
  (new → under-30-min → breached), plus immediately for anything that looks like an
  active incident. Borderline #security posts are marked 🟠 tentative and don't start an
  SLA clock until you confirm they're real escalations.
- #idr-alerts works differently: there is no requester to reply to, so per the Guidelines
  the ack is an **emoji reaction** on the alert once you've looked at it. A bare alert is
  unacked. CRITICAL alerts and host/data detections (Defender, GuardDuty, malware,
  exfiltration, tunnels) are full-volume; the known-noisy "Ingress Auth From Outside The
  US" alerts still get the ladder but with low-key wording unless the evidence shows a
  Red-list country or a non-employee account.

## Cautions

- Escalations can contain customer data. The skill instructs Claude to summarize, never
  repeat full account/routing numbers, and never move content to unapproved tools — but
  you're still the human in the loop.
- The Guidelines doc, Runbook, and IR process links inside SKILL.md are the source of
  truth for triage rules; the skill re-reads the Guidelines doc when rules seem stale.

Questions / improvements: open an issue or PR, or ping the InfoSec team.
