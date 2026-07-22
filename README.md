# Green Street Garage — gsg.rentals

Static site. No build step, no dependencies.

```
index.html      the entire site (markup, styles, listings, routing)
photos/         all imagery
hero.mp4        cover video, H.264, no audio track
hero.webm       cover video, VP9 (smaller; browsers pick this first)
poster.jpg      first frame, shown while the video loads
favicon.svg
vercel.json     caching + security headers
robots.txt
sitemap.xml
```

## Replacing the cover video

Re-encode with audio stripped and faststart enabled, then overwrite
`hero.mp4` / `hero.webm` / `poster.jpg`:

```bash
ffmpeg -i source.mov -an -c:v libx264 -crf 26 -preset slow \
  -pix_fmt yuv420p -movflags +faststart hero.mp4
ffmpeg -i source.mov -an -c:v libvpx-vp9 -crf 36 -b:v 0 hero.webm
ffmpeg -i source.mov -frames:v 1 -q:v 6 poster.jpg
```

A landscape (16:9) source will fill the cover edge to edge and let you
delete the blurred backdrop layer. A vertical source keeps the blur fill.

## Deploy to Vercel

**Option A — drag and drop (fastest)**
1. Zip this folder's *contents* (not the folder itself).
2. Go to vercel.com → Add New → Project → deploy without Git.
3. Framework preset: **Other**. Leave build command and output directory empty.

**Option B — Git (recommended, gives you version history)**
1. Push this folder to a GitHub repo.
2. Vercel → Add New → Project → import the repo.
3. Framework preset: **Other**. No build command. Output directory: `./`

## Custom domain

Vercel → Project → Settings → Domains → add `gsg.rentals`.
Vercel shows the exact records; at your registrar you'll set:

- `A` record on the apex (`@`) pointing to Vercel's IP
- `CNAME` on `www` pointing to `cname.vercel-dns.com`

DNS can take up to a few hours. HTTPS provisions automatically once it resolves.

Note: Vercel's Hobby plan is licensed for personal, non-commercial use.
A business site needs Pro ($20/mo).

## Email

The inquiry buttons point to `hello@gsg.rentals`. That address needs to exist.
Cheapest route is free forwarding (Cloudflare Email Routing, or your
registrar's forwarding) into an inbox you already read. To *send* as that
address, Google Workspace or Fastmail run about $6–8/month.

---

## Editing the site

Everything lives in `index.html`.

### Adding or changing a listing

Find the `ITEMS` array in the script block. Each entry:

```js
{
  id:"chevrolet-c10-1971",     // becomes the URL: /#/chevrolet-c10-1971
  type:"vehicle",              // "vehicle" or "location"
  code:"A-01",                 // slate number on the card
  category:"Truck",
  title:"1971 Chevrolet C10 Longbed",
  line:"Short summary for the index card",
  desc:["Paragraph one.","Paragraph two."],
  specs:[["Year","1971"],["Condition","Running and driving"]],
  stills:[ {photo:"c10-hero"}, {photo:"c10-01"} ],
  work:[ /* optional, see below */ ]
}
```

The first entry in `stills` is the card image on the index and the hero on
the detail page. Use `{ph:"Photos coming soon"}` for a placeholder block.

### Adding photos

1. Drop the file in `photos/` (JPEG, roughly 1200px on the long edge, quality ~70).
2. Add it to the `PHOTOS` map: `"c10-07": "photos/c10-07.jpg"`
3. Reference it: `{photo:"c10-07"}`

### Selected work credits

Optional per listing. Renders below the gallery; the images link out.

```js
work:[{
  client:"Wahine",
  project:"Novelty '22",
  type:"Lookbook / Apparel campaign",
  photographer:"Ricardo Gomes",
  year:"2022",
  url:"https://wahinehonolulu.com/pages/1-1",
  stills:[ {photo:"wahine-01"}, {photo:"wahine-02"} ]
}]
```

Leave `stills` empty to show a neutral "Images courtesy of —" block instead.
