# InfoSec Hero Triage — Claude Code skill ("Avengers Assemble")

A Claude Code skill that runs the daily InfoSec Hero escalation triage: it sweeps the
security@ mailbox, #security, and #support-security, builds a triage board where every
item links back to its source, tracks the 2-business-hour ack SLA per the
[InfoSec Hero Guidelines](https://docs.google.com/document/d/1pt0KOg27TEVSv_V14zZx-OOupNWcYf-gwg8r4WJcWqI/edit),
and — when you start a shift with **"Avengers Assemble"** — arms a 15-minute recurring
watch that notifies **only on change** (new unacked item, 30-min SLA warning, breach,
apparent incident). Alerts arrive three ways: terminal push, a banner-first board, and a
native macOS Notification Center banner with distinct sounds for normal vs. breach.

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
- **Membership in #support-security** (private channel) — the Slack connector can only
  read channels your account is in.
- macOS for the Notification Center banners (`osascript` — built in, nothing to install).
  On first use, allow notifications for your terminal app if prompted
  (System Settings → Notifications).

## Usage

| You say | It does |
|---|---|
| `Avengers Assemble` | Full triage board + arms the 15-min watch (weekdays 9:07am–4:52pm) |
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

## Cautions

- Escalations can contain customer data. The skill instructs Claude to summarize, never
  repeat full account/routing numbers, and never move content to unapproved tools — but
  you're still the human in the loop.
- The Guidelines doc, Runbook, and IR process links inside SKILL.md are the source of
  truth for triage rules; the skill re-reads the Guidelines doc when rules seem stale.

Questions / improvements: open an issue or PR, or ping the InfoSec team.
