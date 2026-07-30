# Scan App

Arena Club box and card capture. Single file — `scan-app.html` is the whole app.

## Deploy to Vercel

```bash
mkdir scan-app && cd scan-app
# drop scan-app.html and vercel.json in here
git init && git add -A && git commit -m "scan app"
vercel --prod
```

`vercel.json` rewrites `/` to `scan-app.html`, so the root URL serves the app without
renaming anything. If you'd rather not keep the config file, rename the app to `index.html`
and delete `vercel.json` — either works.

No build step, no framework. HTTPS is required for camera access, which Vercel gives you
automatically. Open the URL in Safari on the iPhone and use **Add to Home Screen** — it
launches full-screen with no browser chrome, which matters on a fixed stand. It installs under
the name **Scan App**.

## The four config blocks

All at the top of the `<script>`, marked `▶ CONFIG`.

**1 · Logo** — set `LOGO_SRC` to a URL or base64 data URI. Until then a wordmark stands in.

**2 · Box lookup** — `BOX_REGISTRY` is a stub holding ids 8170–8172. Swap `lookupBox()` for a
fetch against admin so the box name comes from the real record instead of a hard-coded map.
This is the main thing still faked.

**3 · Object storage** — `Storage.put()` currently queues in memory. Set `mode:'presigned'`
and point `endpoint` at a serverless route that returns `{url}` for a PUT. Works the same
for S3 or GCS. Keys are `boxes/{box}/{seq}_{cardId}_{side}.jpg`, so a box is one prefix.

**4 · Capture tuning** — countdown length, stillness thresholds, JPEG quality, guide aspects.

## Orientation

The phone is inverted on the stand, so every frame is rotated 180° in `grab()` *before* it's
written. The live preview is also flipped via CSS, so what you see and what gets saved both
come out upright. Nothing downstream needs to know about the mount.

## Hands-free capture

Auto mode watches for a settled frame, arms (brackets turn amber, ring fills), then fires.
After a shot it waits for movement before re-arming — flipping the card is the trigger, so you
can work straight through a stack without touching the screen. `MANUAL` disables it; the
shutter always works.

## Bluetooth clicker

Yes, and it needs no special API. A Bluetooth shutter remote, page turner or footswitch
pairs at the **OS level** and presents itself as a Bluetooth keyboard — so it sends ordinary
key presses and the app just listens for them. Pair it in iOS Bluetooth settings, not in the app.

Web Bluetooth is not the route: Safari doesn't implement it on any platform, and it cannot
reach HID devices even where it exists. WebHID is Chrome-only. Keyboard events are the path
that actually works on an iPhone.

**The one real trap.** Cheap camera shutter remotes send Volume Up/Down, and iOS keeps those
keys for itself — a web app never sees them. If you press your remote and nothing registers,
that's why. Remotes that send **Enter, Space, arrows or page keys** do come through; they're
usually sold as *e-reader page turners* or *Kindle remotes* rather than camera shutters.
Bluetooth footswitches for musicians and transcriptionists work well too, and leave both
hands on the cards.

**One button is enough.** The primary key advances whichever screen you're on, so a single
click walks the entire box:

```
click 1  SCAN BOX        click 6  CONFIRM CARD
click 2  (box scanned)   click 7  SHUTTER  front
click 3  CONFIRM COUNT   click 8  SHUTTER  back
click 4  SHUTTER  front  click 9  CONFIRM CARD
click 5  SHUTTER  back   click 10 COMPLETE BOX
```

**Pairing and testing.** The start screen has a `CLICKER` row with a `PAIR` button. It asks you
to press the button you want for capture, shows you exactly which key code arrived, then
optionally a second button for confirm. The key code readout is the important part — it tells
you in one press whether your remote's keys reach the browser at all. The mapping is saved to
the device.

Out of the box it already listens for Space, Enter, arrows, page keys and play/pause, so most
remotes work before you pair anything.

## Camera on the deployed app

The prototype falls back to a simulated feed so the flow is clickable anywhere. On the real
Vercel deployment the camera is live, with no code change — but four conditions have to hold,
and all four are easy to get wrong:

1. **HTTPS.** `getUserMedia` is blocked on plain HTTP. Vercel serves HTTPS by default, so this
   is only a problem if you test from a `file://` copy or a LAN IP.
2. **A user gesture.** iOS Safari will not open the camera without one. `SCAN BOX` is that
   gesture — the camera is requested on tap, never on page load.
3. **Permission granted once per origin.** If it's ever denied for the deployment URL, iOS
   remembers. Clear it under Settings → Safari → Camera, or in the AA menu in the address bar.
4. **`playsinline`.** Without it iOS hijacks the video into a fullscreen player. It's set.

The app requests the rear camera at up to 2560 x 1440 and falls back to whatever the device
offers. When it's running simulated instead of live, an amber **SIMULATED · NO CAMERA** badge
sits in the app bar for the whole session, so a demo run can never be mistaken for real capture.

## Confirming the scan

The image on the confirm screen is a **crop of the code that was actually decoded**, not a wide
shot of the box. Both decoders report where they found the code, and the crop is expanded with
extra room above it, since the printed id sits over the QR on an Arena label — so the
confirmation shows the code and the `8170 / Kind / Box` text together. Tap it to enlarge.

If that image ever shows a different box than the one in front of you, the scan grabbed a
neighbouring label, and `RESCAN` is right there.

## Testing it with no camera

Open it on a desktop and deny or ignore the camera. The scan screen renders a simulated box —
corrugated, taped, shipping label facing the lens — and that label carries a **real, decodable
QR for box 8170**, embedded in the file. The app decodes it through the same
`BarcodeDetector`/jsQR path a real camera uses and advances on its own. So the no-camera path
isn't a mock of scanning; it's actual scanning of a drawn box. If it advances, decoding works.

`SKIP · USE 8170` bypasses decoding if you just want to reach the capture flow, and
`ENTER CODE` accepts any of the payload formats by hand.

The simulated feed is deliberately drawn **inverted**, exactly as the phone on the stand sees
the world, so the CSS flip and the `grab()` rotation both get exercised. If a preview ever
looks upside down, that's a real bug and not a demo artifact.

## The production label

Reference image kept alongside this file as `reference-production-label.png`. The decoded
payload is:

```
admin.arenaclub.com/b/8170
```

An admin URL, not a bare code. The app takes the **last path segment** as the box id, so `8170`
is what reaches the top bar, the storage keys and the manifest. Scheme, trailing slash and query
string are all tolerated, and a deeper path like `/boxes/2026/8170` resolves the same way.
Because the payload is a live URL, the box block shows **OPEN IN ADMIN** as a link straight to
the record.

Printed layout, top to bottom: id, kind with its mark, sub line, then a large QR. That format is
mirrored in `LABEL` at the top of the script and in the simulated feed, so the demo now looks
like what the camera will actually see.

Legacy forms still parse — bare codes, `id|name`, and JSON — so nothing breaks if an older label
turns up in a bin.

## Test kit

**`box-labels.pdf`** — four labels in the production format on one letter sheet, ids 8170–8173,
each carrying a real `admin.arenaclub.com/b/<id>` payload. Cut on the dashed lines. Verified
decodable off a 100 dpi render, so a normal print is comfortable.

**`box-label-<id>.png`** — the same four as large single images. No printer: open one on your
monitor and scan the screen.

**`test-cards.pdf`** — three pages. Page 1 is six card fronts at true 2.5 x 3.5, page 2 the six
matching backs, page 3 a slab-sized pair at 3.25 x 5.25 for the `SLAB` guide. Print
single-sided, cut, pair by the big number. Each card carries three things worth having:

- a **number on both sides**, so a front/back mismatch in the stored images is obvious at a glance
- **white registration squares in all four corners** — if a corner is missing from the saved
  image, the crop guide is clipping and `CFG.pad` needs raising
- a **focus target**: type stepping down 7pt to 3pt, plus line-pair bars. If the 4.5pt line is
  legible in the stored file, the capture is sharp enough to review. If only 7pt reads, the
  phone is focusing on the table instead of the card.

## Known gaps for the real build

- Images are in memory only. For offline resilience, queue blobs in IndexedDB and drain on
  reconnect. A dropped connection mid-box currently loses the box.
- Barcode decoding uses `BarcodeDetector` where available and falls back to jsQR from CDN.
  For a production install, vendor jsQR locally so it works on a dead network.
- No dedupe. Scanning the same box twice creates a second set of keys under the same prefix.
- Desktop shortcuts for demos: `Space` captures, `Enter` confirms.
