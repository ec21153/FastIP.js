<div align="center">

<img src="./pics/intro.png" width="80%" />

**Multi-source public IP detection client-side library**

Resolves the client's public IPv4/IPv6 address via a waterfall of **85+ free endpoints** —  
no backend, no API keys, no dependencies. Works on any static site.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Vanilla JS](https://img.shields.io/badge/Vanilla-JS-f7df1e?logo=javascript&logoColor=000)](src/FastIP.js)
[![No Dependencies](https://img.shields.io/badge/Dependencies-0-brightgreen.svg)](#)
[![jsDelivr](https://img.shields.io/badge/cdn-jsDelivr-e84d3d?logo=jsdelivr&logoColor=white)](https://cdn.jsdelivr.net/gh/DosX-dev/FastIP.js@latest/src/FastIP.js)

**[🌍 Live Demo — ip.dosx.su](https://ip.dosx.su/)** &nbsp;·&nbsp; **[📄 Standalone Demo — demo/index.html](demo/index.html)**

</div>

---

## 💡 The Problem It Solves

Sometimes you need a user's IP address on the **frontend** — for geolocation hints, rate-limiting UI, personalization, or just displaying it — but you don't have a server. You can't call `request.headers` from a static HTML page.

**FastIP.js** was built exactly for this case: it queries a large pool of free public IP-echo APIs directly from the browser, picks the first valid response, and gives it back to you as a plain `Promise<string>`. No server. No proxy. No sign-up. Just drop in a script-tag and go.

---

## 🏆 Key Pillars

### 🔒 Reliability — 85+ endpoints, worldwide coverage

**FastIP.js** doesn't rely on a single service. It maintains a pool of **85+ free IP-detection endpoints** spread across different providers, regions, and formats. If one API is down, rate-limited, or blocked in a specific country, the library silently moves on to the next. The endpoint list is fully open and can be updated or forked at any time.

> Works correctly in restricted networks, different geographic regions, and behind VPNs.

### 🛡️ Security — minimal footprint, transparent operation

Each API receives **exactly one plain GET request** — no cookies sent, no ambient credentials forwarded, no POST body, no fingerprinting. **FastIP.js** never transmits any data about the user; it only reads the IP address that each public API returns. The full list of queried hosts is visible in [`src/FastIP.js`](src/FastIP.js) — nothing hidden, nothing phoning home.

### ⚡ Speed — adaptive ranking + multi-tier caching

**FastIP.js** learns from experience. Every successful response earns the winning endpoint a higher score; scores decay slightly on each page load. On repeat visits, the top-ranked source is tried alone first — if it answers fast, only **1 request** is made instead of 8. Results are also cached in memory for **30 seconds**, so multiple calls on the same page are instant.

### 🔍 Transparency — open API pool, fully customizable

The entire source list is a plain JS array at the top of [`src/FastIP.js`](src/FastIP.js). You can inspect every endpoint, remove ones you don't trust, add your own private mirror, or shuffle priorities. No opaque registry, no remote configuration, no surprise changes.

---

## 🚫 Ad Blocker Resilience

**FastIP.js** was tested with popular content blockers enabled. Out of **85+ endpoints**, the following detection rates were observed:

| Extension                                                                                                                | Detected / Blocked     | Notes                                                  |
| ------------------------------------------------------------------------------------------------------------------------ | ---------------------- | ------------------------------------------------------ |
| [**Adblock Plus**](https://chrome.google.com/webstore/detail/adblock-plus-free-ad-bloc/cfhdojbkjhnklbpkdaibdccddilifddb) | ~8–9 endpoints blocked | The rest pass through completely                       |
| [**Privacy Badger**](https://chrome.google.com/webstore/detail/privacy-badger/pkehgijcmpdhfbdbbnkijodmdjhbjlgp)          | ~5 endpoints affected  | Many were only cookie-restricted, not blocked outright |

With 85+ sources in the pool, blocking a handful has no practical effect — the library simply skips them and wins the race with a different endpoint. Blocked hosts are automatically blacklisted internally and won't be retried for 7 days.

---

## 📦 Installation

### Via jsDelivr CDN _(recommended)_

```html
<script src="https://cdn.jsdelivr.net/gh/DosX-dev/FastIP.js@latest/src/FastIP.js"></script>
```

> Always points to the latest release. No build step required — works on any static page, GitHub Pages, Netlify, Cloudflare Pages, etc.

### Download & self-host

Grab [`src/FastIP.js`](src/FastIP.js) and serve it from your own origin:

```html
<script src="/js/FastIP.js"></script>
```

---

## 🎯 Quick Demo

```html
<!doctype html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>My IP</title>
    </head>
    <body>
        <p>Your IP address: <strong id="ip">detecting…</strong></p>

        <script src="https://cdn.jsdelivr.net/gh/DosX-dev/FastIP.js@latest/src/FastIP.js"></script>
        <script>
            FastIP.getPublicIP()
                .then(function (ip) {
                    document.getElementById("ip").textContent = ip;
                })
                .catch(function () {
                    document.getElementById("ip").textContent = "unavailable";
                });
        </script>
    </body>
</html>
```

**[→ See it live at ip.dosx.su](https://ip.dosx.su/)**

> 📂 A fully self-contained standalone demo is available at [`demo/index.html`](demo/index.html).  
> Open it directly in a browser — no server needed. Demonstrates both `getPublicIP()` and `getDualStack()` with a live UI.

---

## 🔧 API

### `FastIP.getPublicIP()`

Returns a `Promise<string>` — the client's best public IP address (IPv4 preferred, falls back to IPv6).

```js
FastIP.getPublicIP().then(function (ip) {
    console.log("Your IP:", ip);
});

// async/await
const ip = await FastIP.getPublicIP();
console.log("Your IP:", ip);
```

### `FastIP.getDualStack()`

Returns `{ v4: Promise<string|null>, v6: Promise<string|null> }` — two **independent** promises. Renders IPv4 immediately; IPv6 upgrades the result when available.

```js
const { v4, v6 } = FastIP.getDualStack();

v4.then(function (ip) {
    console.log("IPv4:", ip);
});
v6.then(function (ip) {
    console.log("IPv6:", ip ?? "not available");
});
```

> `v6` resolves to `null` if the client has no IPv6 connectivity.

---

## ⚙️ How It Works

```
┌──────────────────────────────────────────────────────────┐
│                    FastIP.getPublicIP()                  │
├──────────────────────────────────────────────────────────┤
│  1. Load stored rankings from localStorage  (< 1 ms)     │
│  2. Sort 85+ sources by adaptive score  (best first)     │
│  3. Golden shortcut: top source score ≥ 30?              │
│     → try it alone  (800 ms timeout, 1 request)          │
│  4. Fast tier: race top-8 sources  (2.5 s timeout)       │
│  5. Full pool: remaining sources  (6 s timeout)          │
│  6. Winner → +2 score; all scores decay × 0.92/session   │
│  7. Network error → source blacklisted for 7 days        │
│  8. Result cached in memory for 30 s                     │
└──────────────────────────────────────────────────────────┘
```

### 🏗️ Architecture Details

-   **Waterfall strategy** — fast tier first, full pool only on fallback
-   **`Promise.any()`** — first valid response wins; the rest are aborted via `AbortController`
-   **Score system** — each source holds a score 0–99; winners gain points, scores decay every session
-   **Compressed persistence** — state is JSON-serialized → `deflate-raw`-compressed → stored in a single `localStorage` key (`FastIP2`)
-   **Private IP filter** — RFC 1918 / RFC 4193 / loopback addresses are always rejected

---

## 🌐 Sources

FastIP.js ships with **85+ free IP detection endpoints**, including:

| Provider             | Type            | Stack   |
| -------------------- | --------------- | ------- |
| Cloudflare CDN Trace | `trace`         | Dual    |
| icanhazip.com        | `text`          | v4 / v6 |
| ipify.org            | `json`          | v4 / v6 |
| Amazon AWS checkip   | `text`          | Dual    |
| Mullvad VPN          | `text` / `json` | Dual    |
| ipinfo.io            | `json`          | Dual    |
| ident.me             | `text`          | v4 / v6 |
| seeip.org            | `json` / `text` | v4 / v6 |
| … and 70+ more       |                 |         |

All sources are **free**, **keyless**, and **publicly accessible**. The full list is visible in [`src/FastIP.js`](src/FastIP.js).

---

## 🛡️ Privacy & Security

-   **Read-only** — each API receives a single plain GET request; no data is ever sent
-   **No tracking** — no cookies forwarded, no analytics, no fingerprinting
-   **localStorage only** — cached scores stay on the client, never transmitted
-   **Private IPs rejected** — RFC 1918 / RFC 4193 / loopback ranges are filtered out
-   **Zero dependencies** — no third-party code introduced into your build

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.
