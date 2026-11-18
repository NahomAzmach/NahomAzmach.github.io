Conservation and natural-resources jobs are scattered across a bunch of boards that don't talk to each other. A posting goes up on one site, a state agency lists something on another, and if you're not refreshing all of them constantly, you find out about the good one after it closes.

So I built a watcher that does the refreshing for you. It's free and it's always on.

![The dashboard, "Field Notes: Job Watch." Every tracked job, filterable by state and source, with new ones tagged.](assets/img/envsci.webp)

## What it watches

Twice a day it checks three sources:

- **Conservation Job Board**: Washington, Oregon, California
- **TAMU Natural Resources Job Board**: WA, OR, CA
- **Washington State government jobs**: Ecology, Fish & Wildlife, Natural Resources, Transportation

Anything new gets emailed as a digest, and every tracked job shows up on a filterable dashboard. When I grabbed that screenshot it was tracking 888 jobs, 56 of them new in the last 48 hours.

## The whole thing is a diff

The core is simple on purpose. Scrape everything, compare against what we already know, and only the difference matters:

```python
for jid, job in scraped.items():
    if jid in known:
        job["first_seen"] = known[jid].get("first_seen", now_iso)
    else:
        job["first_seen"] = now_iso
        new_jobs.append(job)          # only these get emailed
    job["last_seen"] = now_iso
    known[jid] = job

# prune jobs that disappeared from the sites a while ago
cutoff = now - timedelta(days=PRUNE_AFTER_DAYS)
```

Each source is its own little module, and each one fails independently. If one board changes its HTML and breaks, the other two still run, and the digest tells you which source failed instead of the whole thing silently dying.

## $0 infrastructure

This was a fun constraint: no servers, no database, no bill.

- **GitHub Actions** runs the scraper around 7 AM and 5 PM Pacific and commits the updated data back to the repo.
- **The data is just a JSON file** that lives in `docs/`.
- **GitHub Pages** serves the dashboard from that same folder, so the dashboard reads the JSON the scraper just committed.
- **Gmail** sends the digest with an app password stored as a repo secret.

Funny thing: in [Commute Watch](#commute-watch) I had to move *off* GitHub Actions because its scheduler kept firing hours late on a 10-minute schedule. Here, for something that runs twice a day, an hour of drift doesn't matter at all. Same tool, totally different fit. The requirement decides.

## Built to hand off

I wanted someone non-technical to be able to own this. Changing who gets the emails is editing one secret on GitHub. The README walks through setup from zero: push the folder, make an app password, add three secrets, turn on Pages, hit "Run workflow" to test.

**Stack:** Python (requests + BeautifulSoup), GitHub Actions, GitHub Pages, vanilla HTML/JS dashboard, Gmail SMTP.
