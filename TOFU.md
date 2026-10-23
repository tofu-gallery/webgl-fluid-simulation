# Tofu gallery copy

This repository is Tofu's copy of **PavelDoGreat/WebGL-Fluid-Simulation** (licence: MIT), listed in Tofu's app gallery as **WebGL Fluid Simulation**.

- `master` mirrors the upstream repository and is never committed to by Tofu.
- `tofu` carries Tofu's own changes; tags named `pin/<date>` mark the versions the gallery points to.
- A visitor who chooses this app gets a private snapshot of the pinned version in their own Tofu account. Their copy does not follow upstream updates.

Pins: `pin/2026-10-23` is the first pin and still contains the upstream author's Google Analytics. `pin/2026-10-23.1` removes it and is the one to use.

## What the app is

An interactive fluid simulation that runs in the browser on WebGL (WebGL 2, with a WebGL 1 fallback). The visitor drags a pointer or a finger across a black canvas and coloured dye swirls, glows and fades. A control panel (dat.GUI) changes quality, vorticity, bloom, sunrays and the background colour, and can save a screenshot. It is plain HTML and JavaScript: no build step, no server, no account, nothing to configure.

## How Tofu runs it

Plain static site: served as it is, on Tofu's free static lane (a prebuilt site: `index.html` at the root folder and no `package.json`).

| Setting | Value |
|---|---|
| Framework preset | `static` |
| Root folder | the repository root (`index.html` is at the top) |
| Install command | none (there is no `package.json`) |
| Build command | none |
| Output directory | the root folder itself |
| Environment variables | none |
| Database | none |

## What Tofu changes

- **2026-10-23: removed the upstream author's Google Analytics.** Upstream's `index.html` loaded `https://www.google-analytics.com/analytics.js` and reported a page view to the author's own Google Analytics property (`UA-105392568-1`). Upstream's `script.js` then sent seven more events to it through `ga('send', 'event', ...)`: which WebGL version the visitor's device supports (on every start) and clicks on the author's links. A copy in someone's own Tofu account would have kept reporting to a third party, so all of it is gone:
  - `index.html`: the two analytics `<script>` tags, replaced by a one-line comment.
  - `script.js`: the seven `ga('send', 'event', ...)` calls (each click handler keeps its `window.open(...)` line), plus a one-line comment under the licence header. The calls had to go as well as the snippet: without the snippet `ga` is not defined, the call inside `getWebGLContext` runs on every start, and the app stops with `Uncaught ReferenceError: ga is not defined` before it draws anything or builds its control panel (checked in a browser, see Verification).
  - Kept on purpose, as the app's own links and credits: the Github, Twitter, Discord and "Check out mobile app" buttons in the control panel, and the mobile promo popup (phones only, 20 seconds after the page opens) with the App Store and Google Play badges. They point at the author's own pages and apps and only open when a visitor clicks; nothing is sent anywhere before that.
- No other file is changed. `LICENSE`, the licence header of `script.js` and the notice at the top of `dat.gui.min.js` are exactly as upstream.

## Upstream

- Repository: https://github.com/PavelDoGreat/WebGL-Fluid-Simulation
- Mirrored commit: `a2d292931f19d9b3b9f564e23e6c32729d2121c3` ("Merge pull request #91 from pbrubaker/main", 2024-11-12), the head of `master` when this copy was pinned.

## Licence

MIT, Copyright (c) 2017 Pavel Dobryakov. Licence files and copyright notices stay as they are; the two files Tofu changed (`index.html` and `script.js`) carry a one-line note saying so.

Caveats:

- `dat.gui.min.js` is a copy of dat.GUI (Copyright 2011 Data Arts Team, Google Creative Lab), licensed under the Apache License 2.0. Its header is untouched and Tofu has not modified it.
- `app_badge.png` and `gp_badge.png` are the "Download on the App Store" and "Get it on Google Play" badges. The badges and those names are Apple's and Google's trademarks. They only appear inside the mobile promo popup.
- The repository does not say where `iconfont.ttf` (the icons on the control panel's link buttons), `LDR_LLL1_0.png` (the dither texture the display shader samples), `logo.png` or `promo_back.png` come from. Its only licence file is the MIT `LICENSE`. They are shipped exactly as upstream ships them.

## Verification

Run on 2026-10-23 against the `tofu` branch. Nothing was deployed and no Docker was used.

- Static lane: there is no `package.json`, so nothing is installed or built. `node --check script.js` parses cleanly.
- Served a copy of the app folder with the importer's dropped paths removed (`.git`, `.DS_Store`) using `python3 -I -m http.server <port> --bind 127.0.0.1 --directory <copy>`. `curl` returned HTTP 200 for `/` and for every file the page loads: `script.js`, `dat.gui.min.js`, `iconfont.ttf`, `LDR_LLL1_0.png`, `logo.png`, `app_badge.png`, `gp_badge.png`, `promo_back.png`.
- Browser check (Chrome 153 on macOS, own tab, 127.0.0.1 only, 1440x900 at 2x, WebGL 2): the page loads with no `console.error` or `console.warn` call and no uncaught exception or rejection; `typeof ga`, `typeof gtag` and `typeof dataLayer` are all `undefined`; the page loads 6 resources (7 by the end of the test), every one from 127.0.0.1, adds no element that points at another site, and makes no `fetch`, XHR or beacon call. The control panel lists all of its controls, including Github, Twitter, Discord and Check out mobile app.
- Core action: before any input the canvas already shows swirling colour (mean brightness, brightest colour channel 0 to 255, 27.9 across the canvas), and a counter on the app's `splat` function (installed after start-up) read 0. A real mouse drag (button down, 30 moves across the canvas, button up) took it to 30, left a bright plume along the path and raised the mean brightness of the whole canvas to 87.7; the band along the drag path went from 26.2 to 85.5, and a band elsewhere on the screen from 28.5 to 45.5 as the glow and sunrays spread. Pixels were read from the canvas's own drawing buffer in the frame after the app drew it.
- The author's own links: clicking the four control-panel buttons and the two promo badges runs their handlers with no error and calls `window.open` with the author's own addresses. The browser's `window.open` was replaced by a recorder for this test, so no external page was opened.
- The network check can see a request: the same hooks, run against a script tag and a `fetch` to the reserved host `www.google-analytics.com.invalid`, reported both.
- Why `script.js` changed too: upstream's `script.js` with only the snippet removed from `index.html` stops at `Uncaught ReferenceError: ga is not defined` (`script.js:156`) and never builds the control panel.
- Import filter: the pushed `tofu` branch's file list, read through the same path filter Tofu's GitHub import applies (root folder = the repository root, prebuilt site), keeps 14 files and drops 1 (`.DS_Store`, upstream's macOS folder file, which nothing refers to). Every file `index.html` links to is kept.
