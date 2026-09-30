# Leather Wallet — Scroll Animation

A scroll-driven product animation: 300 frames of a leather wallet being made,
scrubbed frame-by-frame as the page scrolls. The technique premium product
pages use, in a single self-contained HTML file.

**[View it](https://turbocode99.github.io/wallet-web-app/)** once GitHub Pages is enabled (Settings → Pages → deploy from `main`, root).

## Running it

No build step, no dependencies, no server required:

```bash
open index.html
```

Every frame is embedded as a base64 `data:` URI, so the file works offline and
from the filesystem — nothing is fetched at runtime.

## How it works

**Sticky stage over a tall scroll space.** `#scroll-space` is `400vh` tall while
`#stage` is `position:sticky; height:100vh`. That gives four screens of scroll
distance to drive the animation while the canvas stays pinned in the viewport.

**Scroll maps to a frame index.** `onScroll` converts scroll position into a
0‑to‑1 percentage and multiplies it by the frame count, so the sequence tracks
the scrollbar exactly rather than playing on a timer.

**Smoothed, cross-faded scrubbing.** Rather than snapping to the nearest frame,
a floating-point index is eased toward the scroll target every
`requestAnimationFrame` (`SMOOTHING = 0.12`), and the two nearest frames are
composited with `globalAlpha`. Without the blend, fast scrolling reads as
discrete jumps; with it, motion stays continuous even between frames.

**Preload before interaction.** All 300 images load behind an overlay with a
progress bar; scrubbing is disabled until every frame is decoded, so the
animation never stutters on a frame that has not arrived. `img.onerror`
counts toward completion too, so a bad frame cannot hang the loader forever.

**Canvas sized from the source image.** The canvas adopts each frame's
intrinsic dimensions, and CSS (`max-width/max-height:100%`) handles the fit —
so the sequence stays sharp without hardcoding a resolution.

## Working on this from another machine

The repository is **private**, so the other device needs to be signed in to
GitHub first — either the CLI:

```bash
gh auth login -h github.com -w
```

or an HTTPS clone using a personal access token with `repo` scope.

```bash
git clone https://github.com/turbocode99/wallet-web-app.git
```

That is the whole setup. There is no install step, no dependencies and no
build — open `index.html` and it runs.

Normal round trip:

```bash
git pull --ff-only && git add -A && git commit -m "..." && git push
```

**Pull before you edit, and edit on one machine at a time.** `index.html` is a
single ~9 MB file that is almost entirely one long array of base64 frames, so
if the same file is changed on two devices, git cannot produce a meaningful
three-way merge — the conflict arrives as megabytes of interleaved base64 with
no sane way to resolve it by hand. `--ff-only` makes git refuse to
auto-create a merge in that situation rather than handing you that mess.

## Structure

| | |
|---|---|
| `index.html` | Everything — markup, styles, 300 embedded frames, and the scrubber |

The file is ~9 MB, almost entirely base64 image data. The JavaScript itself is
about 100 lines.

## Notes

The frame count is declared once in `FRAME_COUNT` and the frames live in the
`FRAMES` array immediately below it. Swapping in a different sequence means
replacing that array and updating the count to match — nothing else in the
script is sequence-specific.
