# Organic Maps setup (offline vector maps, backup to OsmAnd)

Organic Maps is a second free, offline-first vector map app. Good as a cross-check against
OsmAnd, or as a backup if one app has a bug on a given device. It has no satellite layer at all
(vector only), which is fine for a backup app.

## Steps

1. Install Organic Maps (free, from the App Store) and open it. It launches straight to a map.
2. **Location permission alert.** iOS will ask *"Allow 'Organic Maps' to use your location?"*
   with three choices (something like "Allow Once" / "Allow While Using App" / "Don't Allow").
   This is a system-level alert, not an in-app one. Pick **"Allow While Using App"** if you want
   live position; "Don't Allow" is fine too — it only disables live location, not offline map
   browsing or downloading.
   - **Trap:** if you decline location, Organic Maps will raise its own in-app "Location services
     are disabled" dialog with a link to Settings the next time it needs a fix (e.g. when you
     zoom to your current area). You can dismiss this with **Cancel** — it's not required for
     downloading or browsing offline maps by name. To enable it later: **Settings app → Organic
     Maps → Location → While Using the App**.
3. Open the menu (hamburger icon, bottom-right) → **Download Maps**.
4. Tap **Find map** (search field) and type your region name. Searching by state/country name
   alone (e.g. "Texas") returns a very large bundle (in testing, "Texas" alone was 1 GB across 11
   maps) — search by city or metro name instead to get a right-sized download (in testing,
   searching "Austin" surfaced a ~116 MB metro-area file).
5. Tap the result, then **"Download map (_ MB)"** in the action sheet that appears.
6. Back out to the Download Maps root screen and confirm: it should show **"DOWNLOADED"** with a
   total size, and your country/region listed with a checkmark.

## Trap: CarPlay dialog

If this device has ever been connected to CarPlay (even once), Organic Maps may carry a
CarPlay-activation UI element in its accessibility tree that is not actually on the phone screen.
If you're driving this app with an automation tool and taps land on the wrong element (e.g. a tap
meant for the search button opens the route planner instead), that's this — not a bug in your
script. Not an issue for manual/by-hand use.

## Verify

See [`verify-offline.md`](verify-offline.md).
