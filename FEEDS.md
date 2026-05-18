# RSS Feeds

21 feeds currently configured. Categorised by type and annotated with known fetch behaviour.

---

## Feed List

| # | Source | URL | Category | Notes |
|---|---|---|---|---|
| 1 | The Hacker News | `https://feeds.feedburner.com/TheHackersNews` | General | Reliable, high volume |
| 2 | Rapid7 Blog | `https://blog.rapid7.com/rss` | Vulnerability research | |
| 3 | Dark Reading | `https://www.darkreading.com/rss.xml` | Industry news | |
| 4 | BankInfoSecurity | `https://www.bankinfosecurity.com/rssFeeds.php?type=main` | Financial / CNI | **Bot-blocked** - requires HTTP fallback with browser User-Agent |
| 5 | Palo Alto Unit 42 | `https://unit42.paloaltonetworks.com/feed` | Vendor threat research | |
| 6 | Malwarebytes Blog | `https://blog.malwarebytes.com/feed/` | Malware analysis | |
| 7 | Microsoft Security Blog | `https://www.microsoft.com/en-us/security/blog/feed/` | Microsoft stack | |
| 8 | Krebs on Security | `https://krebsonsecurity.com/feed` | Investigative | |
| 9 | SANS ISC | `https://isc.sans.edu/rssfeed.xml` | Daily indicators | Atom format |
| 10 | Fortiguard IR | `https://www.fortiguard.com/rss/ir.xml` | Vendor IR | **Bot-blocked** - requires HTTP fallback with browser User-Agent |
| 11 | Recorded Future | `https://www.recordedfuture.com/feed` | Threat intelligence | |
| 12 | Cisco Talos | `https://feeds.feedburner.com/feedburner/Talos` | Vendor threat research | FeedBurner relay |
| 13 | Malware Traffic Analysis | `https://www.malware-traffic-analysis.net/blog-entries.rss` | PCAP / traffic analysis | Infrequent posts, high signal |
| 14 | CISA ICS Advisories | `https://www.cisa.gov/uscert/ics/advisories/advisories.xml` | OT/ICS | US government feed |
| 15 | Bleeping Computer | `https://www.bleepingcomputer.com/feed/` | Vulnerability / malware | High volume |
| 16 | The DFIR Report | `https://thedfirreport.com/feed/` | DFIR case studies | Low volume, very high signal |
| 17 | JPCERT/CC | `https://blogs.jpcert.or.jp/en/atom.xml` | National CERT | Atom format |
| 18 | Elastic Security Labs | `https://www.elastic.co/security-labs/rss/feed.xml` | Detection research | |
| 19 | CrowdStrike Blog | `https://www.crowdstrike.com/en-us/blog/feed` | Vendor threat research | |
| 20 | SentinelOne Labs | `https://www.sentinelone.com/labs/feed/` | Malware and threat research | |
| 21 | Red Canary | `https://redcanary.com/feed` | Detection engineering | |

---

## Adding a New Feed

1. Open the `RSS Feed List (EF)` node
2. Add a new entry to the `feeds` JSON array:
   ```json
   { "name": "Source Name", "url": "https://example.com/feed.rss" }
   ```
3. Update `totalFeeds` - the `Clear Accumulator (CJS)` node reads the feeds array length automatically, so no manual update is needed
4. Add a domain mapping to `resolveSource()` in `Filter pubDate 12h (CJS)`:
   ```javascript
   if (link.includes('example.com')) return 'Source Name';
   ```
5. Save and re-activate the workflow

---

## Planned Additions

- [ ] Telegram feed ingestion via RSS bridge or Bot API
- [ ] Onion-site crawling for dark web leak sites and underground forums (Tor + scraper)

