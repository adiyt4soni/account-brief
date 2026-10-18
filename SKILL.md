---
name: account-brief
description: Build a sourced one-page account brief on a UK multi-site employer for an SDR at a B2B software vendor (Companies House public register, the company's own results, an incumbent-system search), check it against code guardrails, and render it as HTML. Use when asked for an account brief, account research or a hypothesis on a named company.
---

# Account brief

Goal: a one-page brief an SDR can trust. Every fact sourced and dated, a labelled hypothesis on where the problem the
vendor solves is probably costing the company money, the current incumbent system if one is public, the roles to approach, one draft
opener, and what couldn't be found. Paths below are relative to the project folder.

**Modes.**
- `live` (default): Companies House, **one** primary results or RNS fetch, and **one** incumbent
  search. No accounts images. Target about 3 minutes; the slow part is web search and page fetching (Companies
  House takes about 3 seconds, the checker 1 second). If the results search turns up no readable primary source,
  stop searching and record it as an open item rather than searching again.
- `full` (refreshing a backup): also read the filed accounts images.

## Steps

1. **Which company.** Ask for the **company number** or the **full legal name**. Short trading names often resolve to
   the wrong company (a famous brand name can match an unrelated small company or a non-trading shell).
   Keep a pick list (`demos/data/panel_picks.md`) of trading entities by number for likely targets (it also lists
   the vendor's customers, which are never run).
2. **Folder.** Make a short lowercase slug and write to a new folder `demos/out/live-<slug>-<HHMM>/` (HHMM = the time
   now). Never write into `demos/out/prerun/`: those are the pre-run backups.
3. **Companies House.** Run:
   `python3 -I demos/account_brief/ch_fetch.py "<number or full legal name>" --out demos/out/live-<slug>-<HHMM> --no-accounts`
   (drop `--no-accounts` in full mode). The first line says which company, its STATUS and its SIC. Then:
   - **exit 3, "EXISTING CUSTOMER":** stop. Say it is a published customer of the vendor and is not briefed.
     This check runs before any network call.
   - **exit 2, "NO EXACT NAME MATCH" or "NOT A TRADING COMPANY":** nothing was written. Read the candidate list it
     printed (number, status, SIC) or `panel_picks.md`, and re-run **by number** with the trading company that
     employs the store staff. Never pick a dissolved, dormant (99999) or non-trading (74990) company. If the
     company itself is dissolved, in liquidation or in administration, stop and say so: not a prospect.
   - **exit 1:** Companies House could not be read. Say so and show a pre-run brief.
   - A "NOTE: SIC says holding or head-office company" line means the store staff may sit in a subsidiary: check
     `panel_picks.md` before going on.
   Read `ch.json`. The register lists each officer's role (Director, Secretary) and dates only, **not their job
   function**; a board change is "a director was appointed on this date", nothing more.
4. **Accounts (full mode only).** If `accounts_reading.text_layer` is false, open the contact sheets in
   `accounts_reading.contact_sheets` in order with the Read tool until you have found:
   - the strategic report KPIs (revenue, store or site count, openings and closures, cost commentary, any mention
     of labour costs, wages, National Living Wage, employer NICs, distribution centre, productivity);
   - the note on employees (average number, and split by stores, distribution and head office if given) and the
     staff costs (wages and salaries, total).

   When a figure is small on the sheet, open the single page from `accounts_reading.pages_dir`
   (`page%03d.png`) to read it exactly. Stop opening sheets once you have those. Cite each figure as the
   Companies House filing URL (`latest_accounts.pdf`) with "accounts p.N" in `source_name`, where N is the PDF page.
   **Never guess a figure you couldn't read.** Put it in `open_items`. The accounts name the directors: never copy
   a name into the brief.
5. **Results (one fetch).** Find the latest trading update, results or RNS (Investegate or the London Stock
   Exchange for listed companies; the company's own investor page otherwise). Fetch the page and cite the page, not
   the search result. Log the search and the fetch in `source_log` with the number of useful results (0 if none).
6. **Incumbent (one search).** Search:
   `"<company>" <product-category keywords> software` (for example the category your product competes in)
   and check whether the company appears on a competitor's customer or case-study page among the results (keep
   the competitor list in `demos/data/competitors.md`). If one does, fetch it and add a fact whose text
   starts **"Incumbent:"** (vendor, what the page says the company uses, the page's date if shown; confidence
   `likely`, vendor-reported). If none, add the open item
   **"Incumbent search done (<query>, <YYYY-MM-DD>): none found."** The checker warns if neither is there.
7. **Write `demos/out/live-<slug>-<HHMM>/brief.json`** in exactly this shape:
   ```json
   {
     "company": {"name": "...", "number": "...", "sector": "...", "domain": "company's own web domain, optional"},
     "generated_at": "YYYY-MM-DD HH:MM",
     "mode": "live|full",
     "facts": [{"id": "F1", "text": "...", "source_url": "https://...", "source_name": "...",
                "date_read": "YYYY-MM-DD", "confidence": "certain|likely"}],
     "signals": [{"date": "Mon YYYY", "text": "...", "fact_ids": ["F1"]}],
     "hypothesis": {"text": "...", "fact_ids": ["F1", "F2"]},
     "roles": [{"title": "Chief Operating Officer", "why": "...", "fact_ids": ["F3"]}],
     "opener": {"subject": "...", "body": "...", "fact_ids": ["F1"]},
     "open_items": ["..."],
     "source_log": [{"source": "...", "query_or_url": "...", "items_returned": 0}]
   }
   ```
   Merge `ch.json`'s `source_log` into `source_log`. In live mode add the open item
   **"Accounts not read in live mode."**
8. **Run the checks:** `python3 -I demos/account_brief/check_brief.py demos/out/live-<slug>-<HHMM>/brief.json`.
   `ch.json` must sit next to `brief.json` (it holds the officer-name hashes). Fix every FAIL by correcting the
   brief, never by weakening a fact or rewriting a name to dodge a rule. Re-run until it passes. Then open
   `brief.html` (`open` on macOS) and give a two-line summary, including any warnings.

## What the checker does and does not do

It fails a brief when a fact has no https source or cites a search page, a date read is not a real date, a number
in the hypothesis, a signal, a role or the opener is not in a fact that part cites (the company number and SIC codes
do not count; £50k = £50,000 and normal rounding are fine), the company is a published customer of the vendor (by number
or by name), or the brief has an email, a phone number, a LinkedIn profile link, a titled name or an officer name
from the register. It **does not check that a source says what the fact says**: a person does. Say that when
asked.

## Writing rules

- **Facts:** one checkable statement each. `certain` = the company's own filing or announcement; `likely` = press,
  a vendor's case study or another third party. Today's date in `date_read`, written YYYY-MM-DD.
- **Numbers:** write each figure as its fact gives it, or a plain rounding of it (21% for 21.3%). Every number in
  a signal, the hypothesis, a role's "why" or the opener must be in a fact that part cites. Say what a percentage
  is of ("21.3% of continuing revenue (group results)").
- **Signals:** what changed recently and why it matters for the problem the product solves, newest first.
- **Hypothesis:** two or three sentences, framed as a guess: where the problem probably costs them money (for
  a workforce-scheduling product, for example: overstaffed quiet hours, overtime or agency at peaks, rota-building time for store managers, turnover,
  labour cost rising faster than sales). It must follow from the cited facts. If an incumbent is known, frame it
  as a question about the incumbent (what it covers, what it doesn't, is it under review), not as a greenfield
  pitch. Don't claim the vendor would fix it.
- **Roles:** titles only (COO or Retail Operations Director, People Director, CFO or Finance Director, CIO or
  Head of Data), each with one line on why. **No names of people anywhere**, even when public. The register gives
  roles and dates, not job functions: a leader's function comes only from the company's own announcement.
- **Opener:** under 120 words. Open with their situation, not the vendor. One sourced observation, one question.
  Use only numbers that appear in the facts it cites. Plain English, no hype.
- **Open items:** what you looked for and couldn't confirm, for example staff costs not read, or store count not
  found for this year.
- **Never** brief an existing customer of the vendor (the fetcher and the checker both block it); never include email
  addresses or phone numbers.
