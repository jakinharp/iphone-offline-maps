# OsmAnd setup (offline vector maps)

OsmAnd is free, offline-first, and will suggest a region to download based on your current
location the first time it launches.

## Steps

1. Install OsmAnd (free tier, from the App Store) and open it.
2. On first launch it shows a "Download map" screen: *"Let OsmAnd determine your location and
   suggest maps to download for that region."* It will already have a region picked (named after
   wherever you are standing) with a download-size button, e.g. "Download 138.9 MB".
3. Tap that download button. OsmAnd downloads the region in two pieces — a base map file plus an
   auxiliary file — which together land around 150–300 MB depending on the region. In testing
   (region: Austin, TX — a dense mid-size US city — used only because that's where the test
   device happened to be; your own region will differ) this came to 250.9 MB across 2 files, done
   in under a minute on a fast connection.
4. When it finishes, the button changes to "Go to the map." Tap it.
5. **Trap:** OsmAnd can crash in the background during a long gap (for example if you switch away
   and come back much later). This is not a failed download — if it crashed, you'll see a "Share
   crash log" dialog over a default world view on relaunch. Tap **Cancel** on that dialog (no need
   to send the log) and check your region is still there (next step).
6. Confirm the download persisted: **Menu → Maps & Resources → Local**. You should see your
   region listed with its name and size, e.g. "2 map(s) — 250.9 MB."

## What you get, and what you don't

This flow downloads OsmAnd's **vector map** (streets, terrain contours, POIs) for your region.
It does **not** set up a satellite imagery underlay — that's a separate, manual step (long-press
on the map → Actions → Download map, choosing a satellite/Earth source as an "Underlay" layer,
at whatever max zoom you want). That's out of scope for this guide; the vector map alone is
enough to navigate and find your position offline.

## Verify

See [`verify-offline.md`](verify-offline.md) — put the phone in airplane mode and confirm search,
map render, and the Local-maps list all still work.
