If you went to Western Washington University, you know the Bellingham rental hunt. Listings are split across different property management companies' sites, every one with its own layout, and the good places go fast. (We're the Vikings, hence the name.)

Viking Territory pulls rentals from those sites into one place you can actually search.

## Scraping sites that don't want to be scraped

Property manager sites are mostly built for humans clicking around, with listings loaded by JavaScript and inconsistent formats everywhere. So the scrapers use **Selenium** to load pages like a real browser, then BeautifulSoup to pull out each listing. Hammer Properties and a ShowMojo-powered listing map each get their own scraper, and a combined runner merges them.

## Addresses are messier than you think

To put listings on a map you need coordinates, and to get coordinates you need an address a geocoder understands. Real listing addresses look like `1234 Elm St #B - Unit 4`, or they're not addresses at all ("General Application," "Lease Takeover"). So there's a cleaning step before geocoding:

```python
def clean_address(address):
    """Remove apartment numbers and clean up address for geocoding."""
    if should_skip_address(address):          # "General Application", etc.
        return None

    base_address = address.split('-')[0].strip()
    base_address = base_address.split('#')[0].strip()

    words, cleaned, skip_next = base_address.split(), [], False
    for word in words:
        if skip_next:
            skip_next = False
            continue
        if any(p in word.lower() for p in ['apt', 'unit', 'suite', '#']):
            skip_next = True                   # drop "Unit" AND the "4" after it
            continue
        cleaned.append(word)
    return ' '.join(cleaned)
```

Then Nominatim geocodes it, with retries for timeouts, and the listing goes into **DynamoDB**.

## The rest of the pipe

An **Express** API serves listings from DynamoDB, and a **React** front end lets you filter by address and min / max price, normalizing prices like "$1,450/mo" into numbers so the filters actually work. The scraper and API were set up to run on AWS EC2.

This one's marked ongoing, and the next pieces are the obvious ones: more property managers, a proper map view, and alerts when something new in your price range shows up. (Which, now that I've built [Job Watch](#job-watch), is basically a diff and an email.)

**Stack:** Python, Selenium, BeautifulSoup, geopy / Nominatim, AWS DynamoDB + EC2, Node.js, Express, React.
