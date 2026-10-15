# GovCon Data Suite: SAM.gov Alerts, Federal Recompetes and SEC Form D Raises

[![SAM.gov Opportunity Radar](https://img.shields.io/badge/Apify-SAM.gov%20Opportunity%20Radar-00a86b)](https://apify.com/cce_partners/samgov-opportunity-radar)
[![GovCon Recompete Radar](https://img.shields.io/badge/Apify-Recompete%20Radar-00a86b)](https://apify.com/cce_partners/govcon-recompete-radar)
[![SEC Form D Feed](https://img.shields.io/badge/Apify-Form%20D%20New%20Raises-00a86b)](https://apify.com/cce_partners/sec-form-d-new-raises-feed)

Three focused data feeds for **federal contractors**, **capture/BD teams** and **B2B sales**, built on official public data from **SAM.gov**, **USAspending.gov** and **SEC EDGAR**. Each feed returns **only new or changed results**, needs **no API key**, and never outputs personal contact data. They run on Apify with pay-per-result pricing, and you can schedule them and send results to **Google Sheets, Slack, n8n, Make, Zapier** or a webhook.

> This repo contains documentation and examples only. The tools run on Apify; click a badge above to open one.

## Which tool do I need?
| You want to… | Tool | Data source | Price (pay per event) |
|---|---|---|---|
| Get **SAM.gov alerts** for new and amended solicitations, sources sought and presolicitations | [SAM.gov Opportunity Radar](https://apify.com/cce_partners/samgov-opportunity-radar) | SAM.gov | $0.01/run + $0.004/notice |
| Find **expiring federal contracts** (recompetes) 3–18 months out, and see if they're on SAM.gov yet | [GovCon Recompete Radar](https://apify.com/cce_partners/govcon-recompete-radar) | USAspending.gov + SAM.gov | $0.01/run + $0.01/row |
| Get a daily list of **recently funded companies** from SEC Form D filings | [SEC Form D New Raises Feed](https://apify.com/cce_partners/sec-form-d-new-raises-feed) | SEC EDGAR | $0.01/run + $0.005/filing |

**How they fit together:** Recompete Radar shows what's coming 3–18 months out, and Opportunity Radar alerts you the day the solicitation posts. Form D surfaces newly funded companies as teaming partners or commercial buyers.

---

## SAM.gov Opportunity Radar: SAM.gov scraper and RFP alerts
Returns only **new or amended federal contract opportunities** (solicitations, presolicitations, sources sought, combined synopses, awards) for your NAICS codes, set-aside (8(a), SDVOSB, WOSB, HUBZone, small business), agency, state or title keywords.
- **SAM.gov saved searches vs this feed:** each monitor remembers what it has seen, so you get only new notices plus amended ones (deadline or description changes), with full details in the row. You don't need to log in to SAM.gov to read them.
- **SAM.gov API rate limits:** keyless mode reads SAM.gov's official public nightly extract (up to 24h old), so there's no personal-key quota. Add your own free key for real-time results.
- **Never included:** names, emails or phone numbers.

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
Notice types: o = Solicitation, p = Presolicitation, k = Combined Synopsis, r = Sources Sought, s = Special Notice, a = Award. Set-asides: SBA, SBP, 8A, 8AN, HZC, HZS, SDVOSBC, SDVOSBS, WOSB, WOSBSS, EDWOSB, EDWOSBSS, VSA, VSS.

**Output fields include:** `noticeId`, `title`, `solicitationNumber`, `noticeType`, `postedDate`, `responseDeadline`, `department`, `naicsCode`, `setAside`, `popState`, `samUrl`, `changeType` (new or changed).

---

## GovCon Recompete Radar: expiring federal contracts (recompete tracker)
Finds **federal contracts whose period of performance ends in 3–18 months** (you choose the window) from official USAspending.gov data. Each row has the incumbent, UEI, award value, end date, agency, NAICS/PSC, and whether a related **SAM.gov notice** (sources sought, presolicitation, solicitation, bridge J&A) is already posted, with a match confidence:
- `high`: the contract number is referenced in the notice, or a same-agency, same-NAICS notice names the incumbent
- `medium`: the parent vehicle (IDV) is referenced, or the sub-agency, NAICS, PSC and state all match
- `low`: only the sub-agency and NAICS match (often a different requirement)

Most contracts aren't on SAM.gov yet 3–18 months out; that's the point. When a notice appears or the end date moves, the contract comes back as `changed`.

**Example: VA IT recompetes over $1M**
```json
{
  "naicsCodes": ["541512"],
  "awardingAgencies": ["Department of Veterans Affairs"],
  "minAwardAmount": 1000000,
  "monthsFrom": 3,
  "monthsTo": 18,
  "stateKey": "va-it"
}
```
**Example: watch competitors' expiring contracts (by name or UEI)**
```json
{ "incumbents": ["D84JK2RNL7V4", "Peraton"], "monthsFrom": 3, "monthsTo": 24, "stateKey": "competitors" }
```
**Current vs potential end date:** `endDate` is the current period-of-performance end. Unexercised options aren't visible, so a contract may run longer than shown.

---

## SEC Form D New Raises Feed: recently funded companies list
A daily list of **companies that just raised money**, from **SEC Form D filings** (Regulation D private placements) on official EDGAR. Each row has the issuer, CIK, state, industry group, total offering amount, amount sold and remaining, date of first sale, and links to the filing. Only new filings come back, investment funds can be excluded, and **no personal names** (executives, promoters, signers) are ever output.

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
**Example: one-month backfill of biotech raises**
```json
{ "dateFrom": "2026-09-01", "dateTo": "2026-09-30", "industryGroups": ["Biotechnology"], "maxFilings": 1000 }
```
A Form D filing isn't proof a round closed, and this isn't investment advice.

---

## Ready-made automations
- **n8n:** SAM.gov → Slack (in review), Recompete → Google Sheets, Form D → Google Sheets, SAM.gov → Google Sheets *(links added once each template is published)*
- **Make:** SAM.gov → Slack, Form D → Google Sheets *(links added once published)*
- **Any tool:** call `POST https://api.apify.com/v2/acts/cce_partners~<actor-name>/run-sync-get-dataset-items` with the JSON input as the body and your Apify token in the `Authorization: Bearer` header.

## FAQ
**Do I need SAM.gov, USAspending or SEC API keys?** No. A SAM.gov key is optional and gives real-time SAM.gov data.

**Will I get duplicates?** No. Keep the same `monitorName` (SAM.gov) or `stateKey` (Recompete, Form D) between runs, and only new or changed records come back.

**Is personal data included?** No. Contact names, emails and phones are never output, and individual-looking recipients are removed.

**Are these official government products?** No. They're independent tools using public data, not affiliated with GSA, SAM.gov, USAspending.gov or the SEC.

---
Built by [CCE Partners](https://apify.com/cce_partners). Questions or bugs: open an issue in this repo.
