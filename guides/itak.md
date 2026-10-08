# iTAK setup (offline tileset + the online-only overlay trap)

iTAK is a free tactical-map app. On its own it only has Apple's built-in map sources, which
aren't great for offline use. Two things make it useful here:

1. A custom "Google Hybrid" map-source overlay (satellite + labels), imported from a small XML
   file. **This overlay is online-only** — it points at a live Google tile server and renders
   solid black with no connection. It is NOT an offline source. Don't rely on it in the field.
2. iTAK's own **Download Offline Maps** feature, which caches real tiles to local storage for
   whatever source is currently active and whatever area you box-select. This is what actually
   works offline, and it works with *any* active source, including the Google Hybrid overlay
   above (the tiles get cached at download time, while you still had a connection) or iTAK's
   built-in sources.

## Part 1 — import the Google Hybrid overlay (optional, for the satellite look)

1. You need a small XML file describing the tile source. A known-working one is published in the
   open-source [ATAK-Maps](https://github.com/joshuafuller/ATAK-Maps) project releases, as
   `Google/google_hybrid.xml` inside their release zip. It uses a standard x/y/z tile URL
   (`lyrs=y`). **Do not use the equivalent Bing file from the same project** — it uses a quadkey
   URL scheme (`{$q}`) that iTAK doesn't support and will fail silently.
2. Get that XML file onto the phone, into iTAK's own `overlays` folder (lowercase — iTAK's
   container may also have an empty, unrelated `Overlays` folder with a capital O; use the
   lowercase one). By hand, this means AirDrop or the Files app, saving into iTAK's folder under
   "On My iPhone."
3. In iTAK: gear icon → tap the current map-source row → **Map Sources** → **+ (Add)** → **Import
   Saved Maps**. This opens the iOS file picker already scoped to iTAK's own container. Pick the
   XML file you placed in step 2 → **Open**.
4. A new entry appears under **CUSTOM MAP SOURCES** (e.g. "Google - Hybrid"). Tap it to make it
   the active source.

## Part 2 — download an actual offline tileset (the part that works with no signal)

1. With a custom or built-in source active, go back into **Map Sources → + → Download Offline
   Maps**. (This option is greyed out until some map source is active — if you don't see it,
   activate any source first.)
2. This opens a download screen with a map and a bounding box, plus a **"Search for a location by
   address"** field — a reliable way to center the box without needing a live GPS fix. Type a
   city or area name and it will geocode and center there.
3. **Trap: the box starts far too large.** A single search result can default to a viewport
   spanning an entire metro area — in testing this came up as an estimated **65 GB** download
   before any zooming. Pinch to zoom the box *in* (smaller geographic area = smaller download)
   and watch the estimated size shrink as you go — two zoom-in steps took one test case from
   65 GB down to 2.6 GB down to a reasonable 123 MB for a small neighborhood-scale box. Pick
   whatever area size you actually need; don't download at the metro-wide zoom level.
4. Give the tileset a name in the **Name** field.
   - **Trap:** if you tap **Download** before the typed name has visually "committed," iTAK may
     throw an in-app error ("Please provide a Name for this tileset") even though the field shows
     your text. Dismiss the error and tap **Download** again — the second tap goes through.
5. Watch the progress indicator under **OFFLINE MAPS** go to 100%. You'll see your named tileset
   with a thumbnail once it's done.
6. **Set the offline tileset as your active map source** before going into the field (Map
   Sources → tap your downloaded tileset). If you leave an online-only source like Google Hybrid
   active instead, the live map will render black with no signal even though the offline tiles
   are sitting right there on the device, downloaded and ready — because the app is still trying
   to pull from the wrong (online) source.

## Bundle ID note

iTAK's bundle identifier is `com.tpc.itak` — not `com.atakmap.app.iTAK`, which is a reasonable
but wrong guess if you're scripting this and haven't checked.

## Verify

See [`verify-offline.md`](verify-offline.md) for the airplane-mode test and exactly what a pass
looks like for both the overlay (expected to fail/render black) and the offline tileset (expected
to render from disk).
