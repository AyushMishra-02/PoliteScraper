# Polite Scraper

A polite scraper for Books to Scrape that downloads the first three catalogue pages, extracts all 60 book detail URLs, fetches each book page, normalizes the fields, validates the records, and writes a honest run report without crashing on a broken page.

## Target classification

- Site: Books to Scrape
- Why this is appropriate: it is a public practice sandbox built specifically for learning and testing scraping.
- Scope: first 3 catalogue pages only
- Data collected: book title, product URL, price text, stock text, star rating, description, source page, fetch timestamp, and the normalized numeric GBP price
- Rule: I will not reuse this code on another site without checking its rules and terms first.

I checked the live site and the target is a public sandbox intended for scraping practice. The robots check for https://books.toscrape.com/robots.txt returned HTTP 404, so the result is: `no robots file found`.

## Setup

```bash
cd scraper
python -m pip install -r requirements.txt
```

## Run

```bash
cd scraper
python src/main.py
```

Optional failure test:

```bash
cd scraper
python src/main.py --inject-fake-url
```

This adds one deliberately broken product URL to the crawl list so the scraper can prove it skips the broken page and still keeps the valid 60 records.

## Politeness rules followed

- Identifying user-agent: `FlyRankInternshipA9/1.0 (https://github.com/AyushMishra-02/PoliteScraper)`
- Request timeout: 10 seconds
- Delay between requests: 500 ms minimum
- Cache used during development: HTML pages are saved locally and reused instead of re-requesting the site repeatedly
- Status check before use: only HTTP 200 is accepted for a real page fetch
- No duplicate URLs allowed in the final catalogue

## Record schema

The scraper validates each record before writing it to JSON.

```json
{
  "title": "A Light in the Attic",
  "product_url": "https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html",
  "price_text": "£51.77",
  "availability_text": "In stock (22 available)",
  "rating_text": "Three",
  "description": "...",
  "source_page": "https://books.toscrape.com/catalogue/page-1.html",
  "fetched_at": "2026-08-06T10:00:00Z",
  "price_gbp": 51.77
}
```

The pipeline keeps the raw scraped text and the normalized numeric value side by side, and the canonical `product_url` is treated as the stable identity for deduplication.

## Output

The script writes:

- `output/books.json` — validated records
- `output/errors.json` — invalid or failed records
- `output/run-report.json` — the final batch summary

## Example run report

This is the real output from a run of the scraper:

```json
{
  "start_time": "2026-10-05T09:26:17Z",
  "end_time": "2026-10-05T09:26:17Z",
  "duration_seconds": 7.53,
  "pages_fetched": 60,
  "cache_hits": 0,
  "valid_records": 60,
  "invalid_records": 0,
  "failed_pages": 0
}
```

The failure-resilience check with one fake detail URL produced:

```json
{
  "start_time": "2026-10-05T09:18:47.803651Z",
  "end_time": "2026-10-05T09:18:55.334468Z",
  "duration_seconds": 7.53,
  "pages_fetched": 61,
  "cache_hits": 0,
  "valid_records": 60,
  "invalid_records": 1,
  "failed_pages": 1
}
```

This proves one broken page is logged and skipped while the valid 60 records survive.

## Honest limitation

This scraper is intentionally narrow: it only processes the first three catalogue pages of the sandbox, collects the fields needed for this assignment, and does not attempt to crawl the whole site or reuse the code on another target without a fresh rules check.

## Ethics note

I use official public endpoints and a public sandbox designed for learning. I do not bypass logins, paywalls, or restrictions, and I only collect the fields required for this assignment.

## Why no browser was needed

The data is already present in the HTML returned by the server, so loading a browser would add cost and complexity without changing the underlying data source.
