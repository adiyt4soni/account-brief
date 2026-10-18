# Account brief

Builds a one-page, fully sourced account brief on a UK multi-site employer that an SDR can trust before reaching out.

**Type:** Claude Code skill · **Area:** Sales research

## What it does

- Looks the company up on the Companies House public register by number (short trading names often match the wrong company) and refuses dissolved, dormant or non-trading entities.
- Reads one primary results or trading-update page and runs one search for the incumbent competitor.
- Writes a structured `brief.json`: facts (each with an https source, date read and confidence), recent signals, a labelled hypothesis on where the problem the product solves is probably costing money, roles to approach, one short opener, and open items.
- Runs a code checker that fails the brief if any number in the hypothesis, signals, roles or opener is not in a cited fact, if a source is a search page, or if any personal name, email or phone number slipped in. Then renders it as HTML.

## Guardrails

- No names of people anywhere, even public ones: roles only.
- Never briefs an existing customer of the vendor (blocked by both the fetcher and the checker).
- A figure that could not be read goes in open items. It is never guessed.
- The checker says plainly what it does not check: whether a source says what the fact says. A person does that.

## When it triggers

> Build a sourced one-page account brief on a UK multi-site employer for an SDR at a B2B software vendor (Companies House public register, the company's own results, an incumbent-system search), check it against code guardrails, and render it as HTML. Use when asked for an account brief, account research or a hypothesis on a named company.

## What's in this repo

- `SKILL.md`: the skill itself, the file Claude or Codex reads.
- Not included: The helper scripts it drives (`ch_fetch.py`, `check_brief.py`), the company pick list (`panel_picks.md`) and the pre-run backup briefs live in the original project and are not published here.
- Written to be generic: where the skill relies on the user's own files, it names them by role, not by content.

## Install

```bash
mkdir -p ~/.claude/skills/account-brief
curl -fsSL https://raw.githubusercontent.com/adiyt4soni/account-brief/main/SKILL.md -o ~/.claude/skills/account-brief/SKILL.md
```
