# iPhone offline maps for no-signal areas

A setup guide for three free iOS map apps configured to work with no cell signal and no
Wi-Fi: useful for hunting, backpacking, or any trip into an area with no coverage. Written up
so it can be followed by hand, or handed to an agent driving the phone over Appium.

The three apps, what each is good for:

- **OsmAnd** — free offline vector maps (streets, terrain, POIs), the main navigation layer.
- **Organic Maps** — a second free offline vector map, good cross-check / backup to OsmAnd.
- **iTAK** — tactical-style map app; can show a satellite/hybrid overlay, but that overlay is
  online-only (see the trap below). Its own "Download Offline Maps" tileset is what actually
  works with no signal.

## Quick start

1. Install OsmAnd, Organic Maps, and iTAK from the App Store.
2. Follow `guides/osmand.md` and `guides/organic-maps.md` to download your region in each app
   (pick your own region by name — do not reuse the `<your region>` placeholder literally).
3. Follow `guides/itak.md` for iTAK's offline tileset, then `guides/verify-offline.md` to prove
   all three actually work with the radio off before you need them in the field.

## Guides

- [`guides/osmand.md`](guides/osmand.md) — OsmAnd setup
- [`guides/organic-maps.md`](guides/organic-maps.md) — Organic Maps setup
- [`guides/itak.md`](guides/itak.md) — iTAK setup, including the overlay trap and the offline
  tileset size trap
- [`guides/verify-offline.md`](guides/verify-offline.md) — airplane-mode test, what a pass looks
  like per app
- [`agent/appium-recipe.md`](agent/appium-recipe.md) — how to drive an iPhone from a Mac with
  Appium/XCUITest, for anyone automating this setup instead of tapping it by hand

## License

MIT. See `LICENSE`.
