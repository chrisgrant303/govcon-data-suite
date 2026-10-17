# GovCon Data Suite: SAM.gov Alerts, Federal Recompetes, Contract Award Winners and SEC Form D Raises

[![SAM.gov Scraper – New Contract Opportunity Alerts](https://img.shields.io/badge/Apify-SAM.gov%20Scraper-00a86b)](https://apify.com/cce_partners/samgov-opportunity-radar)
[![Expiring Federal Contracts – Recompete Radar](https://img.shields.io/badge/Apify-Recompete%20Radar-00a86b)](https://apify.com/cce_partners/govcon-recompete-radar)
[![Federal Contract Awards – New Winners Feed](https://img.shields.io/badge/Apify-Contract%20Award%20Winners-00a86b)](https://apify.com/cce_partners/federal-contract-award-winners-feed)
[![SEC Form D Scraper – New Startup Funding Raises](https://img.shields.io/badge/Apify-Form%20D%20Scraper-00a86b)](https://apify.com/cce_partners/sec-form-d-new-raises-feed)

Four focused data feeds for **federal contractors**, **capture/BD teams** and **B2B sales**, built on official public data from **SAM.gov**, **USAspending.gov** and **SEC EDGAR**. Give a feed a **monitor name** and each run returns **only new or changed results** for that saved search. No SAM.gov, USAspending or SEC API key is needed, and personal contact data is never output. The feeds run on Apify, so you need an **Apify account**. You pay per result, and you can schedule runs and send results to **Google Sheets, Slack, n8n, Make, Zapier** or a webhook.

> This repo contains documentation and examples only. The tools run on Apify; click a badge above to open one.

## Which tool do I need?
| You want to… | Tool | Data source | Price (pay per event, billed to your Apify account) |
|---|---|---|---|
| Get **SAM.gov alerts** for new and amended solicitations, sources sought and presolicitations | [SAM.gov Scraper – New Contract Opportunity Alerts](https://apify.com/cce_partners/samgov-opportunity-radar) | SAM.gov | $0.01/run + $0.004/notice (optional AI tagging: + $0.002/tagged notice) |
| Find **expiring federal contracts** (recompetes) 3–18 months out, and see if they're on SAM.gov yet | [Expiring Federal Contracts – Recompete Radar](https://apify.com/cce_partners/govcon-recompete-radar) | USAspending.gov + SAM.gov | $0.01/run + $0.01/row |
| See **who just won federal contracts** in your NAICS, agency or set-aside, or track competitors' wins | [Federal Contract Awards – New Winners Feed](https://apify.com/cce_partners/federal-contract-award-winners-feed) | USAspending.gov | $0.01/run + $0.005/award row |
| Get a daily list of **recently funded companies** from SEC Form D filings | [SEC Form D Scraper – New Startup Funding Raises](https://apify.com/cce_partners/sec-form-d-new-raises-feed) | SEC EDGAR | $0.01/run + $0.005/filing |

**How they fit together:** Recompete Radar shows what's coming 3–18 months out, the SAM.gov Scraper alerts you the day the solicitation posts, and the Award Winners feed shows who won once the award is published. Form D surfaces newly funded companies as teaming partners or commercial buyers.

**Monitor names:** the field is `monitorName` in the SAM.gov Scraper, Recompete Radar and the Award Winners feed (Recompete and Award Winners also accept the older `stateKey` as an alias), and `stateKey` in Form D. Each name remembers what it has already returned, so later runs with the same name return only what's new (or changed). Use one name per saved search and keep it the same between runs.
Without a monitor name, nothing is saved (unless an older version already saved state for those filters; see the FAQ): each run returns up to 25 results (SAM.gov: the newest notices in the lookback window; Recompete: the soonest-ending contracts; Award Winners: the newest awards updated in the last 14 days; Form D: the latest filings from the last 5 days), every returned row is charged, and the next run can return the same rows again. Set `maxItems` (`maxFilings` in Form D) to change the 25.

---

## SAM.gov Scraper – New Contract Opportunity Alerts: SAM.gov RFP alerts
Finds **federal contract opportunities** (solicitations, presolicitations, sources sought, combined synopses, awards) for your NAICS codes, set-aside (8(a), SDVOSB, WOSB, HUBZone, small business), agency, state or title keywords. With a `monitorName`, each run returns only **new or amended** notices.
- **SAM.gov saved searches vs this feed:** each monitor remembers what it has seen, so you get only new notices plus amended ones (title, solicitation number, notice type, posted date, response deadline, set-aside, NAICS, PSC or award details changed; description-only edits don't count), with full details in the row. You don't need to log in to SAM.gov to read them.
- **SAM.gov API key is optional:** keyless mode reads SAM.gov's official public nightly extract (up to 24h old), so there's no personal-key quota. Add your own free SAM.gov key (`samApiKey`) for real-time results. You still need an Apify account to run the Actor.
- **Never included:** names, emails or phone numbers.
- **Without a `monitorName`:** each run returns the newest matching notices posted in the lookback window (up to 25 unless you set `maxItems`) and nothing is saved. Every returned notice is charged.

**Example: 8(a) IT services, daily**
```json
{
  "naicsCodes": ["541511", "541512", "541519"],
  "setAside": "8A",
  "noticeTypes": ["o", "k"],
  "lookbackDays": 7,
  "monitorName": "8a-it-daily"
}
```
**Example: SDVOSB construction bids and sources sought in Texas**
```json
{
  "naicsCodes": ["236220", "238210"],
  "setAside": "SDVOSBC",
  "state": "TX",
  "noticeTypes": ["o", "p", "r"],
  "monitorName": "sdvosb-tx-construction"
}
```
Notice types: o = Solicitation, p = Presolicitation, k = Combined Synopsis, r = Sources Sought, s = Special Notice, a = Award, u = J&A, g = Sale of Surplus, i = Intent to Bundle. Set-asides: SBA, SBP, 8A, 8AN, HZC, HZS, SDVOSBC, SDVOSBS, WOSB, WOSBSS, EDWOSB, EDWOSBSS, VSA, VSS.

**Output fields include:** `noticeId`, `title`, `solicitationNumber`, `noticeType`, `postedDate`, `responseDeadline`, `department`, `naicsCode`, `setAside`, `popState`, `samUrl`, `changeType` (`new` or `changed`; `unchanged` only if you turn on `emitUnchanged`).

---

## Expiring Federal Contracts – Recompete Radar: recompete tracker
Finds **federal contracts whose period of performance ends in 3–18 months** (you choose the window) from official USAspending.gov data. Each row has the incumbent, UEI, award value, end date, agency, NAICS/PSC, and whether a related **SAM.gov notice** (sources sought, presolicitation, solicitation, bridge J&A) is already posted, with a match confidence:
- `high`: the contract number is referenced in the notice, or a same-agency, same-NAICS notice names the incumbent
- `medium`: the parent vehicle (IDV) is referenced, or the sub-agency, NAICS, PSC and state all match
- `low`: only the sub-agency and NAICS match (often a different requirement)

No SAM.gov API key is needed: the SAM.gov match uses the official public nightly extract unless you add your own key (`samApiKey`).

Most contracts aren't on SAM.gov yet 3–18 months out; that's the point. With a `monitorName`, later runs return only new contracts plus changed ones. A contract comes back as `changed` when its current end date, award amount or incumbent UEI changes (`changeReason: award`), or when SAM.gov gets a new high- or medium-confidence notice for it (`changeReason: samNotice`). Low-confidence matches never trigger it.

Without a `monitorName`, nothing is saved: each run returns the 25 soonest-ending contracts in your window (`maxItems` changes this), and every row is charged.

**Example: VA IT recompetes over $1M**
```json
{
  "naicsCodes": ["541512"],
  "awardingAgencies": ["Department of Veterans Affairs"],
  "minAwardAmount": 1000000,
  "monthsFrom": 3,
  "monthsTo": 18,
  "monitorName": "va-it"
}
```
**Example: watch competitors' expiring contracts (by name or UEI)**
```json
{ "incumbents": ["D84JK2RNL7V4", "Peraton"], "monthsFrom": 3, "monthsTo": 24, "monitorName": "competitors" }
```
**Current vs potential end date:** `endDate` is the current period-of-performance end. Unexercised options aren't visible, so a contract may run longer than shown.

---

## Federal Contract Awards – New Winners Feed: who just won
Returns **federal contract awards** from official USAspending.gov data, newest first: the winner and its UEI, its **current parent company** (USAspending's recipient profile, or a small cited alias map when USAspending's parent is stale after an acquisition), obligated amount and potential value, start and end dates, and the competition details: **set-aside, extent competed, number of offers, solicitation procedures**. Filter by NAICS, PSC, awarding agency, set-aside, extent competed, place-of-performance state, minimum amount or competitor (`recipients`: UEIs or names).

With a `monitorName`, each run returns only awards that monitor hasn't delivered yet. Turn on `includeAmended` to also get awards you already received whose amount, end date or recipient changed. Without a `monitorName`, nothing is saved: each run returns the newest awards updated in the last `lookbackDays` (14 by default), up to 25 unless you set `maxItems`, and every row is charged.

**DoD delay, plainly:** civilian awards reach USAspending within days, but DoD and USACE awards are published about 90 days after award (DoD policy). They're flagged `dataDelay: dod_90_day`. For earlier signals, the SAM.gov Scraper – New Contract Opportunity Alerts Actor returns SAM.gov award notices.

**Example: new VA IT awards over $250K (teaming and subcontracting leads)**
```json
{
  "naicsCodes": ["541511", "541512", "541519"],
  "awardingAgencies": ["Department of Veterans Affairs"],
  "minAwardAmount": 250000,
  "monitorName": "va-it-new-awards"
}
```
**Example: every new win by two competitors, including changes**
```json
{ "recipients": ["Booz Allen Hamilton", "Leidos"], "minAwardAmount": 100000, "includeAmended": true, "monitorName": "rival-wins" }
```
Open it on Apify: [Federal Contract Awards – New Winners Feed](https://apify.com/cce_partners/federal-contract-award-winners-feed).

---

## SEC Form D Scraper – New Startup Funding Raises: recently funded companies list
A daily list of **companies that just raised money**, from **SEC Form D filings** (Regulation D private placements) on official EDGAR. Each row has the issuer, CIK, state, industry group, total offering amount, amount sold and remaining, date of first sale, and links to the filing. Investment funds are excluded by default (excluded filings aren't charged), and **no personal names** (executives, promoters, signers) are ever output.

**Only new filings:** set a `stateKey`, and later runs with the same name return only filings that monitor hasn't returned yet. Without a `stateKey`, each run returns the latest filings from the last 5 days (up to 25 unless you set `maxFilings`), so repeats are possible.

Form D is filed within 15 days of the first sale, so it's often the earliest public signal of a new round, before any press release.

**Example: tech companies in CA/NY raising $2M+**
```json
{
  "states": ["CA", "NY"],
  "industryGroups": ["Other Technology"],
  "minOfferingAmount": 2000000,
  "excludeFunds": true,
  "stateKey": "tech-ca-ny"
}
```
**Example: one-month backfill of biotech raises** (a fixed date window; no monitor name needed)
```json
{ "dateFrom": "2026-09-01", "dateTo": "2026-09-30", "industryGroups": ["Biotechnology"], "maxFilings": 1000 }
```
A Form D filing isn't proof a round closed, and this isn't investment advice.

---

## Ready-made automations
- **n8n:** [Send new SAM.gov contract opportunities to Slack with Apify](https://n8n.io/workflows/20979). More n8n templates (Recompete → Google Sheets, Form D → Google Sheets, SAM.gov → Google Sheets) are on the way.
- **GitHub Actions:** [gh-action-govcon-feeds](https://github.com/chrisgrant303/gh-action-govcon-feeds) runs any of the four feeds from a scheduled workflow (aliases `sam`, `recompete`, `formd`, `awards`) and writes CSV/JSON files and a job summary. Store your Apify token as the `APIFY_TOKEN` secret:
```yaml
- uses: chrisgrant303/gh-action-govcon-feeds@v1
  with:
    apify-token: ${{ secrets.APIFY_TOKEN }}
    actor: sam
    monitor-name: it-8a-daily
    input-json: '{"naicsCodes":["541512","541519"],"setAside":"8A"}'
```
`monitor-name` is written to the right field for each Actor (`monitorName`, or `stateKey` for Form D).
- **Make:** SAM.gov → Slack, Form D → Google Sheets *(links added once published)*
- **Any tool:** call `POST https://api.apify.com/v2/acts/cce_partners~<actor-name>/run-sync-get-dataset-items` with the JSON input as the body and your own Apify API token in the `Authorization: Bearer` header. Include a monitor name in the body if you want only new results.
- **AI agents (MCP):** add `https://mcp.apify.com/?tools=cce_partners/<actor-name>` to your MCP client and connect with your Apify account (OAuth), or send your own Apify API token as `Authorization: Bearer <token>`. It doesn't work without signing in.

## FAQ
**Do I need SAM.gov, USAspending or SEC API keys?** No. A SAM.gov key is optional and gives real-time SAM.gov data.

**Do I need an account?** Yes, an Apify account. The Actors run there and each run is billed to your account at the prices above. AI agents connect through Apify's MCP server with the same account (OAuth).

**Will I get duplicates?** Not if you set a monitor name. Keep the same `monitorName` (SAM.gov, Recompete, Award Winners) or `stateKey` (Form D) between runs, and only new or changed records come back. Without a monitor name, every run returns the newest (SAM.gov, Award Winners, Form D) or soonest-ending (Recompete) results again, and charges them again.

**I ran it on a schedule without a monitor name before. What changes?** For SAM.gov and Form D, runs with filters keep using the saved state that older versions created for those filters, so that schedule still gets only new results (Form D: set `stateKey` to `default` to keep that state explicitly). New schedules, and all Recompete and Award Winners schedules, need a monitor name to avoid repeats.

**Is personal data included?** No. Contact names, emails and phones are never output, and individual-looking recipients are removed.

**Are these official government products?** No. They're independent tools using public data, not affiliated with GSA, SAM.gov, USAspending.gov, the Treasury or the SEC.

---
Built by [CCE Partners](https://apify.com/cce_partners). Questions or bugs: open an issue in this repo.
