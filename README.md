# Trades card editor

Turns a raw chart screenshot into a clean 1920 × 1080 trade card: session tabs,
symbol badge, timeframe pill and wordmark over a `#f5f5f5` background.

The editor is a single HTML file. No build step, no dependencies, no server.
Open it and it works. `guide.html` is an illustrated guide that ships alongside
it; the **Guide** button in the header opens it as a panel inside the editor,
so it works the same embedded in an iframe as it does standalone.

## Use it

1. **Load a screenshot** — button, drag and drop, or paste with `Ctrl/⌘ + V`.
2. **Frame it** — drag to move, scroll or pinch to zoom, arrow keys nudge 1 px
   (hold `Shift` for 10 px). Use **Trim edges** to shave exact source pixels off
   a side, which is how you cut a half-drawn candle without shifting the frame.
3. **Set the labels** — symbol, session, timeframe.
4. **Sign it** — pick the X or Instagram mark and type your own username.
5. **Set the ground** — keep `#f5f5f5` or pick any colour; the overlays follow it.
6. **Export PNG** — 1920 × 1080 composition, output up to 7680 × 4320.

The preview canvas *is* the export canvas, so the downloaded file is always
pixel-identical to what you see. There is no second render path that can drift.

## Controls

| Control | What it does |
| --- | --- |
| Symbol | NQ, ES, YM, GC, SI (futures) and EURUSD, GBPUSD (forex). GC carries the gold mark, SI the silver one |
| Session | London, New York, Asia. The active session is always centred; the others rotate around it and fade outward. A slider sets how far that fade goes, 0 leaving them flat grey |
| Timeframe | Presets from M1 to W1, or type your own label |
| Wordmark | Logo is X (filled) or Instagram (outlined, so the chart shows through it). Username is free text up to 20 characters, case preserved — leave it empty and the logo centres on its own. Toggle the whole thing off with **Show on card** |
| Zoom | Slider plus ±0.1% and ±0.5% steps for fine framing |
| Position | 1 px and 10 px nudges, or recentre |
| Trim edges | Per-side crop in source pixels |
| Background | Six presets, a colour picker and a hex field. Default `#f5f5f5`. Every overlay colour is derived from whatever you pick — glass, type, session pill, hairlines and shadows all follow, so the card stays readable on a black ground as readily as a white one |
| Overlay size | Scales all the furniture 80–120% |
| Export quality | 1080p, 4K, 6K or 8K. Overlays are redrawn at the output resolution, not upscaled |
| Overlay tone | Centre blends the glass panels into the background, right lifts them toward white, left sinks them to a soft grey so they stay visible |

## Publish it

### GitHub Pages (recommended)

```bash
git init
git add .
git commit -m "Trades card editor"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
git push -u origin main
```

Then in the repository: **Settings → Pages → Source → GitHub Actions**. The
included workflow publishes on every push to `main`. Your editor will be live at
`https://YOUR-USERNAME.github.io/YOUR-REPO/`.

If you prefer not to use Actions, set **Source → Deploy from a branch → main /
(root)** instead — `index.html` at the repository root is all Pages needs.

### Embedding in Whop

The editor is designed to run inside an iframe.

1. Deploy it first, using GitHub Pages above or any static host. You need a
   public `https://` URL.
2. In the Whop dashboard, create an app and add an **App view**.
3. Set the view's URL to your deployed address.
4. Install the app into your whop and open it.

**Getting the file out of an embedded frame.** Some hosts sandbox their iframes
in a way that blocks automatic downloads. The editor does not rely on that
working: after every export it shows a bar with a thumbnail and three ways to
take the file — **Download**, **Share** (the native share sheet on phones) and
**Open in tab**. At least one works in every environment, so the export is never
trapped inside the frame.

When it detects that it is embedded it also slims its own header to give the
canvas more room, and posts its height to the parent frame for hosts that size
their iframes dynamically.

**The guide opens in place.** The **Guide** button opens the guide as a panel
inside the editor rather than a new tab, because a tab is either blocked by the
host's frame sandbox or throws the reader out of the app view. `Esc` or
**Close** returns to the canvas with the framing and labels untouched, and
**Open in tab** is there for anyone who wants a window of their own. Upload
`guide.html` alongside `index.html`; if it is missing the button says so rather
than showing a 404.

There is nothing to configure and no server component. It stores nothing, sets
no cookies and sends no data anywhere; all rendering happens in the browser.

### Anywhere else

Upload `index.html` to any static host (Netlify, Vercel, Cloudflare Pages, an S3
bucket, your own server). It is one file with no server-side requirements.

## Customise

Everything lives in the `<script>` block of `index.html`.

- **Add a symbol** — append to `SYMBOLS`, then add a matching branch in
  `drawSymbolIcon()`. Badges are drawn with `numberBadge()`, `metalBadge()` or
  the flag helpers, so a new one is usually a single line.
- **Add another logo** — the X and Instagram marks are panel controls, so
  switching between them needs no source edit. For a third mark, append to
  `LOGOS` and add a branch to `drawMark()`. Draw it inside a 24 × 24 space the
  way `drawXMark()` and `drawInstaMark()` do and it scales with everything else.
- **Change the preset grounds** — the background is a panel control, so changing
  it needs no source edit. To change which swatches are offered, edit
  `BACKDROPS`. Everything downstream comes out of `buildPalette()`, so a new
  colour needs no other change.
- **Change the export size** — the `W` and `H` constants, and the `width` and
  `height` attributes on the `<canvas>` element.
- **Change the fonts** — the `POPPINS` and `INTER` constants and the Google
  Fonts `<link>` in `<head>`.
- **Adjust the layout** — the overlay geometry is all inside `draw()`, expressed
  in 1920 × 1080 coordinates and multiplied by the overlay-size scale `u`.

## Notes

- Fonts load from Google Fonts. Fully offline, the canvas falls back to your
  system sans-serif and re-renders automatically once webfonts arrive, so text
  is never drawn in the wrong face.
- HEIC images cannot be decoded by browsers. On iPhone, use a screenshot or set
  Settings → Camera → Formats → Most Compatible.
- The editor works from a copy capped at 3840 px on the long edge so panning
  stays smooth, but the untouched original is kept and used at export time, so
  no detail from the source screenshot is lost.
- Every overlay is drawn as vectors and text, never as bitmaps, so exporting at
  4K or 8K rasterises them natively at that size rather than scaling them up.
  Every size is also rendered larger than it is saved and resolved down, which
  is what smooths the curve of a badge and the edge of the type — 1080p and 4K
  get a full 2× pass, 6K a partial one, and 8K is already at the largest canvas
  the export will allocate.
- That supersampling cleans up the overlay, not the chart. A 1080p screenshot
  exported at 8K gives crisp badges and type over a chart with no more detail
  than it started with; the source screenshot is the ceiling.
- If a device cannot allocate the canvas for a large export it falls back
  automatically and tells you to pick a smaller size, rather than failing
  silently.

## Licence

MIT. See [LICENSE](LICENSE).
