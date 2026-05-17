# Sample Output

Real output from a live pipeline run. All screenshots are from actual digest executions — no mock data.

---

## Digest — Discord

Twice-daily digest delivered to a Discord channel. Items sorted by relevance score descending, each with source, category, AI-generated summary, and suggested action.

![Discord Digest](discord_digest.png)

---

## Digest — Email (Gmail)

Same digest delivered as an HTML email via Gmail OAuth2.

![Email Digest](email_digest.png)

---

## Error Notifications

When a feed fails to fetch (connection timeout, bot block, HTTP error), the error notification branch fires immediately — separate from the digest run. Alerts go to both Discord and Gmail with feed name, URL, status code, and error detail.

**Discord:**

![Discord Crawl Failure](discord_crawl_failure.png)

**Gmail:**

![Email Crawl Failure](email_crawl_failure.png)

---

## Audit Log — Feed Noise Analysis

Google Sheets audit log after several weeks of runs. Every item processed is logged with verdict (`PASSED` / `REJECTED`), triage score, category, and reason. This view is a summary chart built on top of the raw audit data — used to calibrate triage thresholds and identify noisy feeds.

![Feed Noise Analysis](gsheet_feed_noise_analysis_chart.png)
