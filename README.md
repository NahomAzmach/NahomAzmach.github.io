# nahomazmach.xyz

My portfolio and blog. Plain HTML/CSS/JS with no build step.

## Run it locally

The posts load as Markdown files, so open the site through a small server instead of double-clicking the file:

```bash
python -m http.server 8000
# then visit http://localhost:8000
```

## Where things live

| What | Where |
|---|---|
| Name, intro, links, about page, project list, thought list | `content.js` |
| Project write-ups | `posts/<slug>.md` |
| Thoughts | `thoughts/<slug>.md` |
| Images | `assets/img/` |
| Layout, styles, routing | `index.html` |

## Post a new thought

1. Write `thoughts/my-new-idea.md` in Markdown.
2. Add it to the top of `THOUGHTS` in `content.js`:
   ```js
   { slug: "my-new-idea", title: "My new idea", date: "2026-12-01", summary: "One line." },
   ```
3. Deploy. It shows up on the home page and under Writing at `#note-my-new-idea`.

## Add a project

Add an entry to `PROJECTS` in `content.js`. Set `cover: null` to get a generated cover, or point it at an image in `assets/img/`. Then write `posts/<slug>.md`.

Markdown extras: image alt text becomes the caption. You can also use `<div class="stats">` (big numbers), `<div class="flow">` (numbered steps) and `<div class="pair">` (two images side by side). See the existing posts for examples.

## Deploy

Any static host works, including GitHub Pages, Vercel and Netlify. Upload the folder as is.
