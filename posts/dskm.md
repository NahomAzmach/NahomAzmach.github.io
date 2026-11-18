Debre Selam Kidus Michael is an Ethiopian Orthodox Tewahedo church community in Edmonds, Washington. The old site had years of real community history in it: services, Sunday school, events, photo archives. But it was hard to navigate and showing its age.

The job was a rebuild that looks credible, calm, welcoming and current, while keeping the church's Ethiopian Orthodox identity front and center, not buried under a generic template.

![The rebuilt home page, with the Amharic / English toggle up top.](assets/img/dskm.webp)

## Requirements before pixels

Before writing any code I wrote a full requirements doc: who the site is for, what visitors need to find fast (service times, address, contact, donations, events), the information architecture, and just as important, what's **out of scope** for a first pass (member portals, user accounts, rewriting donation processing). For a volunteer-run organization, a site that's easy to update matters more than a site with lots of features.

## Rescuing the old content

The old site was content the community had built up over years, and losing any of it wasn't an option. So I wrote a scraper that walks the old site's content API and pulls everything down, pages and media both, into a local export:

```python
BASE_URL = "https://eotcdskm.org"
API_BASE = f"{BASE_URL}/api/content"

def safe_name(value: str) -> str:
    # turn any URL into a filesystem-safe filename for the archive
    value = value.strip().replace("://", "_").replace("/", "_")
    value = re.sub(r"[^A-Za-z0-9_.\-]+", "_", value)
    return re.sub(r"_+", "_", value).strip("._") or "index"
```

Every legacy image lives on in a Media & Gallery archive, including things like the community health day, where the church hosted blood-pressure screenings, health education and care-package distribution.

## Two languages, one site

Both **Amharic and English** page content are maintained in the source, with a toggle in the header. One deliberate call: the navigation labels stay in English no matter which language you pick. People switch languages mid-visit, and a menu that rearranges itself under you is disorienting. The content is what changes.

## Organizing a big site

There's a lot here: member services, services, Sunday school, media and gallery, about and reach-us, each with sub-pages. To keep it maintainable, content lives in one file (`content.js`), image libraries and gallery helpers in another, and route rendering in the app. Adding an event or a page means editing content, not components.

**Stack:** React, React Router, Vite, Python (scraper), Vercel.
