# Cloture

A browser extension for Chrome, Microsoft Edge and Firefox that live-tracks how long you spend on the sites you've flagged as distracting, nudges you as you approach your daily limit, and blocks the site once you blow past it. All data stays in your browser.

## Formerly FocusGuard

Version 1.8 renamed the extension from FocusGuard to Cloture — the store already held some
fifty FocusGuards. Cloture is the procedure that ends a debate, which is what the block screen
does to the one about five more minutes.

**Updating from FocusGuard keeps your data, on two conditions:**

1. Extract the new files into **the same folder you already use**, whatever it is called. An
   unpacked extension's identity is its folder path; a new folder is a new, empty extension.
2. Press **Reload** on the extensions page afterwards. Restarting the browser is not enough: it
   keeps running the previous background worker, and blocking will not work until you reload.
   The popup shows a "Finish the update" banner when it detects this.

Your limits, history and API key carry over. The old background timer is removed automatically.

## Two builds, one source tree

`build.sh` produces both packages from the same code:

```
./build.sh
  -> dist/cloture-chrome-edge.zip
  -> dist/cloture-firefox.zip
```

Only two things differ. Firefox has no MV3 service worker, so its manifest adds a
`background.scripts` entry pointing at a bundled classic script (the Chrome build keeps the
module service worker, which Firefox ignores); and the Firefox manifest carries the
`browser_specific_settings.gecko` block Mozilla requires. Everything else — all the JavaScript,
every page, the icons — is byte-identical between the two.

The code calls `api.*`, defined once per file as `globalThis.browser ?? globalThis.chrome`,
because Firefox's promise-based namespace is `browser.*` and Chrome's is `chrome.*`.

## Install (about 60 seconds)

1. Unzip `cloture.zip` somewhere permanent — if you delete the folder, the extension stops working.
2. Open your browser's extensions page: `chrome://extensions` in Chrome, **`edge://extensions` in Microsoft Edge**.
3. Turn on **Developer mode** (top right).
4. Click **Load unpacked** and select the unzipped `cloture` folder.
5. Pin the icon: click the puzzle-piece in the toolbar, then the pin next to Cloture.

The settings page opens automatically on first install.

### Microsoft Edge

The same folder — there is no separate Edge build and nothing to convert. Edge is Chromium underneath, reads the same Manifest V3, and implements every API this extension uses. Open `edge://extensions`, turn on **Developer mode** in the left sidebar, click **Load unpacked**, and choose the same unzipped folder. Pin it from the puzzle-piece icon in the toolbar.

Three Edge-specific things worth knowing:

- **Notifications go through Windows.** If Focus Assist / Do Not Disturb is on, the 60% nudge is silent. The badge still counts and the block still appears. Windows Settings → System → Notifications controls it.
- **Efficiency mode can delay a block.** Edge throttles background work on battery, so the 30-second tick may fire late and a block can arrive a few seconds behind. Tracking totals are unaffected.
- **Sleeping tabs are irrelevant.** Edge suspends background tabs, and background tabs were never counted anyway.

Brave, Arc, Opera and any other Chromium browser load it the same way.

### Firefox

Use `dist/cloture-firefox.zip`, not the Chrome one. Unzip it, then:

1. Open `about:debugging` → **This Firefox**.
2. Click **Load Temporary Add-on…**
3. Select **`manifest.json` inside the unzipped folder** — the file, not the folder. This is the
   step people get wrong.

**Temporary add-ons are removed when Firefox restarts.** That is Firefox's rule for unsigned
extensions, not a defect. For a permanent install you either sign the package through
addons.mozilla.org (`web-ext sign`) or use Firefox Developer Edition with
`xpinstall.signatures.required` set to `false` in `about:config`.

Four Firefox-specific behaviours:

- **Host access is optional and revocable.** Firefox treats `<all_urls>` as a permission you can
  decline, and without it the block screen can never appear. The popup shows an amber
  "Blocking is off" banner with a button to grant access when this happens. On Chrome and Edge
  the permission comes with the install and the banner never shows.
- **Blocks can arrive up to a minute late.** Firefox doesn't document a minimum alarm interval,
  so the Firefox build ticks once a minute instead of every 30 seconds.
- **The background is an event page, not a service worker.** It gets suspended the same way, and
  the tracker was already written to survive that — elapsed time is banked to storage on events
  rather than held in a running timer.
- **The block screen may fall back.** It is normally an extension iframe, which a page cannot
  read keystrokes from. If Firefox refuses the frame under a page's CSP, the extension falls
  back to an in-page overlay that intercepts keys and focus at `window` capture. Both paths
  work; the iframe is the better one.

## How the escalation works

| Where you are | What happens |
|---|---|
| Under 60% of the limit | Green badge quietly counting minutes on the toolbar icon |
| 60–99% | Badge turns amber, one desktop notification: "18m on instagram.com today" |
| At the limit | Badge turns red, notification, and a full-page overlay with a **Snooze 5 min** button |
| Snoozes used up (2/day by default) | Hard block — the overlay won't go away until you type a sentence out in full |

Every nudge, block and override is counted and shown in the dashboard, which is the part that actually changes behavior.

## Dashboard

Click the toolbar icon:

- Total time on distracting sites today, compared against your 7-day average
- Per-site bars showing time against each limit (over-limit sites show the percentage)
- 14-day trend chart
- Counters for nudges, blocks, overrides, and a "clean days" streak — consecutive days where nothing hit its limit

## Smart suggestions (bring your own key)

Guessing which sites are your problem is the boring part, and the answer is personal: Instagram is a job for a marketer and a sinkhole for a litigator. So the extension can ask a model to sort your browsing for you.

You type two things — what your actual work is, and what you think you waste time on — and Cloture summarizes your history into a table of hostnames, visit counts, and a coarse timing shape (weekday working hours / off hours / late night), plus the minutes it has already measured for sites you track. The model returns a verdict per host — **distraction**, **work**, or **neutral** — with a one-line reason and a suggested daily limit. You tick the ones you accept; nothing is applied on its own, and only "distraction" rows are tickable.

Five things make this defensible rather than alarming:

1. **History access is an optional permission.** The extension doesn't hold it until you press the button, and your browser asks you first. Never use the feature and it never has it.
2. **Only hostnames leave the machine.** No URLs, no paths, no query strings, no page titles — so the searches themselves, which is the sensitive part, are never in the payload. The summary is built from `new URL(x).hostname` and nothing else.
3. **You see the payload before it is sent.** "Preview what would be sent" prints the exact request body, and sends nothing.
4. **The request is pinned to zero-data-retention endpoints.** Every call sets `provider.zdr: true`, `provider.data_collection: "deny"`, and `allow_fallbacks: false` — so if no qualifying provider is available the request *fails* rather than quietly routing to one that retains your data. "US providers only" additionally pins `provider.only` to US-headquartered providers with ZDR endpoints. OpenRouter also offers a true US-only regional endpoint (`us.openrouter.ai`), but that requires a Business or Enterprise plan; the toggle is there and tells you so.
5. **Every run is logged.** Model, endpoint, provider options, which provider actually served it, hostnames sent, token usage. The log is in your JSON export. Your API key is not — it's stripped from exports and never synced.

The key is yours: create one at openrouter.ai and paste it into Settings. Any model slug works; runs cost a fraction of a cent.

## Insights (the full dashboard)

Click **View insights** in the popup. It opens a full page covering 7, 30, or 90 days:

- **The period at a glance** — total time, change vs the previous period of equal length, average per recorded day, and your **override rate** (overrides ÷ blocks): when the wall goes up, how often do you actually stop?
- **Daily minutes**, as bars with a **7-day rolling average** over the top. The line is the signal; one bad Sunday is noise.
- **What happened when you hit a limit** — nudges, blocks, snoozes, overrides, clean days, longest clean streak.
- **By site** — time, change against the previous period, days over limit, and the share of days you stayed under. Sortable by biggest change (new habits first) or most time.
- **Share of tracked browsing** — distracting sites against everything else.
- **Export CSV** — one row per site per day, for checking the arithmetic in Excel.

### What the Insights page refuses to do

A day with no record means the browser was closed or tracking was paused. Those days are **excluded from averages, never counted as zero**, and any comparison is suppressed entirely when either period has under 40% coverage — so a week where your laptop was shut doesn't masquerade as an improvement. The page states its own limits at the bottom: that the clock stops when you're idle (so video and long reads undercount), that "days over limit" is judged against your *current* limit because limit history isn't recorded yet, and that falling minutes are a measurement, not proof of causation.

## Settings

Right-click the icon → Options, or click **Settings** in the popup.

- Add or remove tracked sites and set a per-site daily limit (12 common ones ship by default)
- Move the first-nudge threshold, snooze count and snooze length
- Turn the hard block on or off, and set the override sentence — make it long and irritating
- Set how much idle time counts as "walked away" (default 60s)
- Export all history as JSON, clear today, or delete everything

## What counts as time

Only the tab you're actually looking at, in a focused browser window, while you're at the keyboard. Background tabs, minimized windows and idle time don't count. A single slice is capped at 10 minutes so a sleeping laptop can't record hours.

## Privacy

Everything is in `chrome.storage.local` on this machine. No account, no server, no analytics. The `<all_urls>` permission is what lets the extension read the hostname of your active tab and draw the overlay; it never reads page content. The one and only outbound request the extension can make is the Smart suggestions call described above — it happens when you press the button, to the endpoint you configured, with your own key, and you can see the payload first. Leave that feature alone and Cloture makes no network requests at all.

## Files

| File | What it does |
|---|---|
| `manifest.json` | MV3 manifest, permissions, entry points |
| `smart.js` | Smart suggestions: history summary, prompt, OpenRouter call, response parsing |
| `background.js` | Service worker: the tracking loop, thresholds, badge, notifications |
| `common.js` | Shared storage schema, date math, site matching, level logic |
| `content.js` / `content.css` | Decides when to block; mounts the block screen and the in-page fallback |
| `block.html` / `block.js` | The block screen itself, run as an extension page inside an iframe |
| `popup.html` / `popup.js` | The dashboard |
| `options.html` / `options.js` | Settings |

## Why the block screen is an iframe

An overlay injected into the page shares the page's event path and its focus. A site with a capture-phase `keydown` handler — which most feeds have, for their single-key shortcuts — sees every character you type into the override box and can cancel it, and a site with a focus trap pulls the caret straight back out. The result is a box you can't type into while the page quietly receives your keystrokes.

The block screen therefore runs as an extension page inside a `chrome-extension://` iframe. That frame is a separate document and a separate origin: the page cannot read its key events, cancel them, or focus anything inside it. Your override sentence is typed into the extension, never into the site.

If a page ever refuses the frame, the extension falls back to an in-page overlay that intercepts key and focus events at `window` capture — ahead of any listener the page has on `document` — and takes focus back the instant a page tries to steal it. Chrome and Edge both exempt web-accessible extension frames from page CSP, so the fallback is really there for a future Firefox build.

## Known limits

- Chromium browsers only (Chrome, Edge, Brave, Arc). Firefox needs a day's work — the promise-based namespace is `browser.*`, not `chrome.*` — and Safari needs a native wrapper built in Xcode.
- It sees browser tabs, not native desktop apps (Slack, Steam) and not your phone.
- Time is banked every 30 seconds, so the last half-minute before a block can be slightly stale.
- The hard block is a speed bump, not a security control — anyone can disable an extension from `chrome://extensions` or `edge://extensions`. That's by design: friction is the mechanism, not enforcement.

## Phones

iOS and Android don't let a third-party app monitor other apps this way. On iPhone, Screen Time → App Limits does the daily-limit-plus-block flow natively; on Android it's Digital Wellbeing → App Timers. Both are worth setting to match the limits you pick here.
