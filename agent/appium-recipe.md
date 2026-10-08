# Driving an iPhone from a Mac with Appium/XCUITest

This is the recipe for automating the setup in the other guides — scripting an agent (or your
own tooling) to drive a real iPhone over USB, from a Mac, instead of tapping through it by hand.
It assumes: a Mac with Xcode installed, a free (or paid) Apple Developer account, and a physical
iPhone connected to that Mac.

## Why a LaunchAgent, not a plain SSH session

If you start the Appium server as a normal SSH-launched background process, codesigning the
WebDriverAgent runner can fail with `errSecInternalComponent`. This is a known class of macOS
codesigning issue tied to processes that aren't attached to a real GUI session/keychain context.

**Fix:** run Appium as a `launchd` LaunchAgent, bootstrapped into the logged-in GUI session, not
as a bare SSH-spawned process:

```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/<your-label>.plist
# to restart it later:
launchctl kickstart -k gui/$(id -u)/<your-label>
# to check it's alive:
launchctl print gui/$(id -u)/<your-label> | grep -E "pid|state"
```

This is the same pattern used for GitLab Runner on macOS, for the same underlying reason.

## The free Apple ID 7-day re-sign problem

If you're signing the WebDriverAgent build with a free (non-paid) Apple Developer account, the
resulting on-device trust expires after **7 days** and you'll need to rebuild/re-sign. A paid
account's signing lasts a year. Budget for this if you're setting up a long-running automation
rig on a free account.

## Phone-side trust gate

The first time a build from a given developer identity runs on the device, iOS blocks it until
you manually trust it: **Settings → General → VPN & Device Management → Developer App → Trust.**
This is a one-time human tap per developer identity per device — no way around it
programmatically. If you see `"Developer App Certificate is not trusted"` in the Appium/Xcode
log, this is why.

## Finding bundle IDs

Don't guess bundle IDs. Use `devicectl` on the Mac, with `--include-all-apps` (the default filter
only shows apps built with your own developer account):

```bash
/Applications/Xcode.app/Contents/Developer/usr/bin/devicectl device info apps \
  --device <UDID> --include-all-apps | grep -iE "itak|osmand|organic"
```

If `xcode-select -p` points at CommandLineTools rather than a full Xcode install, invoke
`devicectl` by its full path as above rather than relying on `$PATH`.

## Minimal session capabilities

```json
{
  "capabilities": {
    "firstMatch": [{
      "platformName": "iOS",
      "appium:automationName": "XCUITest",
      "appium:udid": "<UDID>",
      "appium:xcodeOrgId": "<YOUR_TEAM_ID>",
      "appium:xcodeSigningId": "Apple Development",
      "appium:updatedWDABundleId": "<your-reverse-dns>.WebDriverAgentRunner",
      "appium:bundleId": "<target-app-bundle-id>",
      "appium:showXcodeLog": true,
      "appium:wdaLaunchTimeout": 180000,
      "appium:allowProvisioningDeviceRegistration": true,
      "appium:newCommandTimeout": 600
    }]
  }
}
```

Notes:
- Set `newCommandTimeout` generously (600+ seconds). The default (60s) will kill your session if
  there's any gap between tool calls on your side, even though the device itself is fine — this
  cost one build session a mid-flow `invalid session id` failure.
- Leave `usePrebuiltWDA` off on the first run; once a derived-data build exists, Appium reuses it
  and subsequent session starts are fast (seconds instead of minutes).
- If the first `POST /session` 500s with a new error signature (one seen in testing:
  `RemoteXPC upstream connect error` / `Timed out while enabling automation mode`), retry once
  before treating it as a real blocker — it can be transient. Stop and investigate after two
  identical failures in a row.

## The basic driving loop

```bash
# window size
curl -s http://127.0.0.1:4723/session/<id>/window/rect

# screenshot (base64 PNG in .value — decode it; verify the magic bytes \x89PNG\r\n\x1a\n
# if you don't have `file` available to check it for you)
curl -s http://127.0.0.1:4723/session/<id>/screenshot

# tap, via W3C actions (coordinates in points, not pixels)
curl -s -X POST http://127.0.0.1:4723/session/<id>/actions \
  -H "Content-Type: application/json" \
  -d '{"actions":[{"type":"pointer","id":"finger1","parameters":{"pointerType":"touch"},
       "actions":[{"type":"pointerMove","duration":0,"x":207,"y":448},
                   {"type":"pointerDown","button":0},
                   {"type":"pause","duration":100},
                   {"type":"pointerUp","button":0}]}]}'

# full accessibility tree (can be slow — see gotcha below)
curl -s http://127.0.0.1:4723/session/<id>/source

# clean up
curl -s -X DELETE http://127.0.0.1:4723/session/<id>
```

## Gotchas

- **System alerts vs. in-app dialogs.** iOS permission prompts (location, notifications, etc.)
  live outside the app's own accessibility scope. Looking them up by `accessibility id` the way
  you would an in-app element will 404. Use the alert-specific endpoints instead:
  `GET /session/<id>/alert/text`, `POST /session/<id>/alert/accept`,
  `POST /session/<id>/alert/dismiss`.
- **`/alert/accept` is not "pick the button I wanted."** It resolves to whatever Apple considers
  the alert's default/positive button. On a 3-choice location alert ("Allow Once" / "Allow While
  Using App" / "Don't Allow") it may land on a button you didn't intend. If the exact choice
  matters, address the button by its accessible label/coordinates instead of the generic accept
  endpoint.
- **Switches vs. their label text.** A settings toggle's row often has a static-text element with
  the same accessibility id as the switch itself. Clicking the label returns HTTP 200 and changes
  nothing. Address the actual `XCUIElementTypeSwitch` by its `name` predicate (for the Settings
  app's own toggles, these are standard reverse-DNS names like
  `com.apple.settings.airplaneMode`), and tap near the switch's own on-screen rectangle (the
  right edge of the row), not wherever the label happens to be.
- **`/source` can hang** on screens with large native table/list views (tens of seconds or more).
  This is not a device failure — `/status` and `/screenshot` will keep responding normally on the
  same session while a slow `/source` call is still in flight. If an element can be found another
  way (coordinates, a visible screenshot, a narrower predicate query), prefer that over a full
  page-source dump on these screens.
- **Large screenshots take longer.** A screen with a lot of imagery (e.g. a rendered satellite
  tile preview) can produce a multi-megabyte base64 payload and take well over the default HTTP
  timeout to return. Use a longer client-side timeout for `/screenshot` on image-heavy screens.
- **Simulated GPS location is not reliable.** `mobile: setSimulatedLocation` may return success
  without the app's own location readout actually updating. Don't assume it worked; check the
  app's own GPS/position display before building further automation on top of it.
- **Addressing a toggle by name, generally.** When an element has no reachable accessibility id,
  fall back to a `-ios predicate string` lookup (e.g. `name == '<exact.identifier>'`) rather than
  guessing at a generic accessibility id.
- **Push/pull files directly into an app's own container**, instead of relying on UI automation
  to get a file in front of the right picker:
  ```
  POST /session/<id>/appium/device/push_file   {"path": "@<bundle-id>:<relative/path>", "data": "<base64>"}
  GET  /session/<id>/appium/device/pull_file?path=@<bundle-id>:<relative/path>
  ```
  Verify a push by immediately pulling the same path back and comparing bytes — don't assume a
  200 response means the file landed correctly.
- **Transport note:** a USB/RemoteXPC-based Appium connection to the device survives the device's
  own Wi-Fi being turned off (e.g. during the airplane-mode verification test) — it does not
  depend on the device's Wi-Fi radio.

## General setup, abstracted

In testing this was run as: a cloud/VPS host opening an SSH tunnel to a Mac (`ssh -f -N -L
4723:127.0.0.1:4723 <your-mac>`), with Appium's HTTP server (default port 4723) running on the
Mac as the LaunchAgent described above, driving the iPhone over USB. Substitute "your Mac" for
whatever machine actually has Xcode and the cable-connected phone; the tunnel step is only needed
if you're driving the Mac from somewhere else.
