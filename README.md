# Antenna — a public-signal browser for iptv-org/iptv

Antenna is a single-file, open-source web app for browsing and watching the free,
publicly listed live TV streams collected by the [iptv-org/iptv](https://github.com/iptv-org/iptv)
project. Pick a country, a category, or a region, search by name, and press watch —
your browser connects straight to the stream. Nothing is proxied, hosted, or stored
by this app.

## Features

- Browse **~180 countries**, **29 categories**, and **40+ regions** pulled live from
  `iptv-org.github.io/iptv`
- Full-text search within whatever's loaded, or load the entire global index
  (thousands of channels) to search across everything at once
- HLS playback via [hls.js](https://github.com/video-dev/hls.js), with automatic
  fallback to native playback (Safari/iOS) and one-shot error recovery
- Every channel pre-buffers a few seconds before playback starts (shown as
  "Buffering…" with a live progress readout), instead of playing immediately
  and stuttering while it catches up — see **Playback buffering** below
- Favorites, saved between sessions
- Keyboard channel-surfing (`↑` / `↓` while a channel is open, `Esc` to close)
- Zero build step, zero dependencies to install — it's one HTML file

## Running it

Antenna has no backend and no build process. Options, easiest first:

1. **Open it directly.** Double-click `antenna.html`. Most browsers will fetch the
   playlists fine from a `file://` page, but a couple of browsers restrict
   cross-origin `fetch()` from local files — if channels won't load, use option 2.
2. **Serve it locally** (recommended):
   ```bash
   npx serve .
   # or
   python3 -m http.server 8080
   ```
   then open the printed URL.
3. **Deploy it anywhere that serves static files** — GitHub Pages, Netlify,
   Vercel, Cloudflare Pages, an S3 bucket, whatever you've got. It's just HTML/CSS/JS.

## OR

Go to the github pages version in deployments!

## How it works

- Channel and stream data comes from the public playlists at
  `https://iptv-org.github.io/iptv/*.m3u` (the same files the iptv-org project's
  own site uses), fetched directly from the browser and parsed client-side.
- `hls.js` is loaded from jsDelivr (`cdn.jsdelivr.net/npm/hls.js@1`), matching the
  library's own documented usage.
- Favorites use the host page's `window.storage` API when available (e.g. inside
  Claude), falling back to an in-memory store otherwise. **If you deploy this
  standalone and want favorites to survive a page reload, swap the `Store` object
  near the top of the script for a small `localStorage` wrapper** — that's
  intentionally left out of the shipped version since it isn't safe inside an
  embedded/sandboxed preview, but it's a five-line change for your own deployment.

## Playback buffering

When you open a channel, Antenna doesn't call `play()` right away. It attaches
the stream, lets it load paused, and polls how far ahead the buffer is. Once
about **4 seconds** are buffered (or **9 seconds** have passed, whichever comes
first), it starts playback. hls.js is also configured with a larger-than-default
buffer target (`maxBufferLength: 30`, `liveSyncDurationCount: 4`, etc.) so it
keeps loading well ahead of the playhead during viewing, not just at the start.
This trades a few seconds of startup delay for noticeably less stalling and
re-buffering once a channel is playing.

If you want more or less of a head start, both numbers are constants near the
top of the player code:

```js
const PREBUFFER_SECONDS = 4;       // how far ahead to buffer before playing
const PREBUFFER_MAX_WAIT_MS = 9000; // give up waiting and play anyway after this
```

## Known limitations

- **Some streams won't play.** These are third-party public streams the project
  doesn't control — a given one may be offline, geo-restricted, or served without
  the CORS headers a browser needs to play it cross-origin. This is normal for
  free public IPTV and not specific to this app. If a channel fails, try the next
  one — most of the catalog is checked automatically by iptv-org and works fine.
- Language-based browsing (iptv-org also groups channels by ~200 languages) isn't
  included, to keep the UI focused — the URL pattern is
  `https://iptv-org.github.io/iptv/languages/<code>.m3u` if you want to add it.
- Search only covers whatever's currently loaded. Use **Load all channels** in the
  header to search the entire catalog at once (it's a larger download).

## Credits & license

- **This app's code**: MIT-licensed, see `LICENSE`.
- **Channel list, logos, and stream links**: from
  [iptv-org/iptv](https://github.com/iptv-org/iptv) and the wider
  [iptv-org](https://github.com/iptv-org) project, released under the
  [Unlicense](https://github.com/iptv-org/iptv/blob/master/LICENSE) (public domain).
  No video is hosted by iptv-org or by this app — see their
  [legal note](https://github.com/iptv-org/iptv#legal) for how takedowns are handled.
- **Playback**: [hls.js](https://github.com/video-dev/hls.js) (Apache-2.0).
