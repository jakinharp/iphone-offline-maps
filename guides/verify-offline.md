# Verify offline: the airplane-mode test

Don't trust "downloaded" as proof it works with no signal. Prove it by actually cutting the
radio and using each app.

## Steps

1. **Settings → Airplane Mode → on.** Confirm Wi-Fi and Cellular both show as off/disabled in
   Settings, and the airplane icon appears in the status bar. (If you're also disabling Wi-Fi
   separately via its own toggle, do that too — some setups leave Wi-Fi joined to a known network
   even in airplane mode until you toggle it off explicitly.)
2. Open each app in turn and check the items below.
3. When done, **Airplane Mode → off**, confirm Wi-Fi/Cellular come back.

## What a pass looks like, per app

**OsmAnd**
- Search for a place name inside your downloaded region. With no network, you should still get
  results pulled from local data (city/POI name, type, distance).
- Tap a result and confirm the map renders a full vector street view around it, not a blank/grey
  tile grid.
- Menu → Maps & Resources → Local should still list your region with its name and size. OsmAnd
  will also show its own "No internet available" banner if you try to browse for *new* regions —
  that banner appearing is itself confirmation the radio is actually off.

**Organic Maps**
- Search for a place name in your region. Local results (street-level, with addresses) should
  come back with no network.
- Tap a result and zoom in — confirm street-level detail renders, not blank tiles.
- Menu → Download Maps should still show your region as downloaded with its size.

**iTAK**
- Map Sources screen: your online overlay (e.g. Google Hybrid) should render as a solid black
  thumbnail with no connection — that's expected and correct, it's proof the overlay really is
  online-only, not a bug.
- Your downloaded offline tileset should render a real thumbnail from local storage (actual
  imagery, not black, not a placeholder) with the radio off. That's the proof that matters.
- Known gap: without a live GPS fix (e.g. testing indoors/at a desk), iTAK's main map view may
  stay parked at a default position and never visually pan to your downloaded area, even though
  the tileset itself is proven to render from disk via the thumbnail above. Getting the live
  viewport onto your tiles requires an actual GPS fix in the field, or a manual pan once you're
  there.

## A note on proof

A "downloaded" badge or a progress bar hitting 100% is a claim, not proof. The airplane-mode
test above is proof, because the behavior (search results, map rendering, thumbnails) is only
possible if the data is genuinely on the device and the app is reading it locally.
