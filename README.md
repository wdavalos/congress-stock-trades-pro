# Congress Stock Trades Pro (House PTRs)

Every STOCK Act Periodic Transaction Report (PTR) filed with the **US House Clerk** (disclosures-clerk.house.gov), parsed into clean transaction rows: member, state/district, asset with ticker, buy/sell, transaction date, notification date, official amount range **plus numeric min/max/midpoint**, description, filing ID, and a link to the original PDF.

No proxy. No API key. Reads the Clerk's public filing index directly - the primary source, not a repackaged dataset.

**Run it on the Apify Store:** https://apify.com/red.cars/congress-stock-trades-pro
**ScraperFleet data page:** https://scraperfleet.com/actors/congress-stock-trades-pro/

## Better than the incumbents

[johnvc/us-congress-financial-disclosures-and-stock-trading-data](https://apify.com/johnvc/us-congress-financial-disclosures-and-stock-trading-data) is the demand leader (38 monthly active users, 5.0 across 5 reviews, $1.63 per 1,000): strong product, but amounts stay as text ranges and no numeric midpoint. This actor reads the same official filings and adds: numeric `amountMin` / `amountMax` / `amountMidpoint` per row, office and district parsed out of the index row, per-transaction `description` (option strikes, share counts, partial-sale notes), and a hard `maxItems` cap so you never pay for rows you didn't ask for. Version 0.0 covers House PTRs; Senate eFD parsing is on the roadmap.

[scrapers_lat/us-house-financial-disclosures-scraper](https://apify.com/scrapers_lat/us-house-financial-disclosures-scraper) charges from $8.32 per 1,000 results for the same Clerk index (0.0 rating, 3 users) and makes transaction extraction opt-in. This actor parses transactions by default at roughly a fifth of that price.

## What one row looks like

```json
{
  "filingYear": 2026,
  "lastName": "Pelosi",
  "fullName": "Nancy Pelosi",
  "state": "CA",
  "district": "11",
  "docId": "20033725",
  "pdfUrl": "https://disclosures-clerk.house.gov/public_disc/ptr-pdfs/2026/20033725.pdf",
  "transaction": {
    "owner": "SP",
    "asset": "Alphabet Inc. - Class A Common Stock (GOOGL) [ST]",
    "ticker": "GOOGL",
    "assetTypeCode": "ST",
    "transactionType": "P",
    "partial": false,
    "transactionDate": "01/16/2026",
    "notificationDate": "01/16/2026",
    "amountMin": 500001,
    "amountMax": 1000000,
    "amountMidpoint": 750000.5,
    "description": "Exercised 50 call options purchased 1/14/25 (5,000 shares) at a strike price of $150 with an expiration date of 1/16/26."
  }
}
```

- `owner`: SP (self), JT (joint), DC (dependent child), SC/SU/JU (spouse/child variants as filed)
- `transactionType`: P (purchase), S (sale, `partial: true` for partial sales), E (exchange)
- `assetTypeCode`: ST stock, OP option, AB, GS, MF, FS, RE, IN, VL, CX, WN, TR and other Clerk codes

## Input

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| `filingYear` | integer | current year | Which Clerk filing-index year to read |
| `lastName` | string | empty | Case-insensitive member surname filter, e.g. `Pelosi` |
| `startDate` / `endDate` | string | empty | Optional YYYY-MM-DD filter on the transaction date |
| `maxItems` | integer | 1000 | Hard cap; the run stops at the cap so you don't pay for unrequested rows |
| `includeDetails` | boolean | true | Include description and filing status per row |

## Pricing

**$0.005 per run + $1.50 per 1,000 transaction rows.** No proxy, no residential fallback, no API key. Billed per row actually returned.

## How it works

1. Reads the House Clerk's public filing index for the chosen year (one POST, no login).
2. Filters PTR Original / PTR Amendment filings (annual disclosure reports are excluded).
3. Downloads each PTR PDF and extracts the transaction table, including multi-line asset names and split amount ranges.
4. Pushes one row per transaction, deduplicated within a filing.

A PDF that fails to parse is returned as a single filing-level row with a `parseError` note instead of being silently dropped, so you always know what you did and did not get.


## Pair it with

**[FEC Campaign Finance Pro](https://apify.com/red.cars/fec-campaign-finance-pro)** covers the money side of the same story: donors, committees, itemized contributions (Schedule A) and independent expenditures (Schedule E) straight from the FEC's official OpenFEC API, $0.005 per run + $2.00 per 1,000 rows.

## FAQ

**Is this the same data Capitol Trades or QuiverQuant shows?** The underlying source is the same: members' STOCK Act PTR filings. This actor reads the Clerk's own archive directly, so it can never lag behind a repackager's ingestion pipeline.

**Does it cover the Senate?** Not yet in version 0.0. The Senate eFD portal requires a separate search flow; Senate PTR parsing is on the roadmap. The House has the higher-profile traders and the largest PTR volume.

**Are amounts exact?** No - the STOCK Act requires reporting in ranges. `amountMidpoint` gives the range midpoint for sorting and aggregation; treat it as an estimate, not a fact.

**How fresh is the data?** As fresh as the Clerk's site. Every row links to the exact PDF it was parsed from (filing ID and URL included), so you can verify any row in one click.

**Is this legal advice or investment advice?** No. PTRs are delayed, self-reported filings. Use for research and transparency monitoring only.
