# Deploy Scan App

## Fastest — drag and drop, no CLI, no git

1. Go to **vercel.com/drop**
2. Drag this whole `scan-app` folder onto the page
3. Pick a team and a project name, then select **Deploy**

Vercel serves the files as-is — there's no framework to detect and no build step.

## From the terminal

```bash
npm i -g vercel
cd scan-app
vercel --prod
```

The first run asks which scope and project to use, then creates a production deployment.

## From git

Push this folder to a repo, then import it at vercel.com/new. Every push redeploys.

---

## After it's live

1. Open the URL in **Safari** on the iPhone. Not Chrome — iOS Chrome has less reliable
   camera access.
2. Allow the camera when prompted. If you deny it once, iOS remembers for that URL; clear it
   under Settings → Safari → Camera.
3. **Share → Add to Home Screen.** It installs as **Scan App** and launches full screen with
   no address bar, which matters on a fixed stand.

**How to tell it's really using the camera:** the amber `SIMULATED · NO CAMERA` badge in the
top bar disappears. If that badge is showing, everything you capture is a drawn placeholder,
not a photograph.

Camera access requires HTTPS. Vercel provides that automatically, so it works on the deployed
URL even though it won't from a `file://` copy on your desktop.

## What's in here

| file | |
|---|---|
| `index.html` | the entire app |
| `vercel.json` | cache and camera headers |
| `test-kit/` | printable box labels and test cards |

`vercel.json` sets `Cache-Control: no-store` on the HTML. The whole app is one file, so
without it a phone will keep serving a stale copy after you redeploy.

## Two things fetched at runtime

Google Fonts for the typefaces, and jsQR from cdnjs as a barcode fallback on browsers without
native `BarcodeDetector`. Everything else — the logo, the label, all the code — is embedded.
If the grading floor has unreliable wifi, both can be inlined so the app runs fully offline.
