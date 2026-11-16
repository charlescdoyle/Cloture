# Cloture — Privacy Policy

**Last updated:** 15 November 2026

Cloture is a browser extension that measures how long you spend on websites you have
chosen to limit, and blocks those sites once you pass the limit you set.

This policy describes everything the extension does with data. It is short because the
extension does very little.

## The short version

- Your browsing time is stored **on your own computer**, in your browser's local extension
  storage. It is not sent to us, because there is no "us" — Cloture has no server, no
  account system and no analytics.
- Nothing is sold, rented, shared, or used for advertising. Ever.
- One optional feature sends a list of **hostnames** (for example `instagram.com`) to an AI
  provider **you choose and pay for with your own API key**. It is off until you turn it on,
  and it never sends page addresses, page content, or anything you typed.

## What Cloture stores on your computer

| Data | Why | Where |
|---|---|---|
| Hostnames of sites you add to your limits list, and the daily seconds spent on each | To count your time and decide when to block | Local extension storage |
| Counts of nudges, blocks, snoozes and overrides per day | The dashboard | Local extension storage |
| Your settings: limits, thresholds, override sentence | To work the way you configured it | Local extension storage |
| Your AI provider API key, if you use the optional feature | To make requests on your behalf | Local extension storage, never synced, excluded from exports |

History older than **120 days is deleted automatically**. You can export everything as a JSON
file, clear a single day, or delete all of it, from the Settings page at any time.

## What Cloture never collects

- The web addresses (URLs) of pages you visit — only the hostname of the tab in front of you
- The content of any page
- Anything you type, including into the override box, which runs inside the extension and not
  inside the website
- Your name, email address, location, or any account identifier
- Any data at all from sites that are not on your limits list, beyond a total count of
  "other browsing" seconds

## The optional AI feature ("Smart suggestions")

This feature is **off by default** and does nothing until you provide your own API key and
press a button.

When you do use it:

1. The extension asks your browser for permission to read your history. This is an **optional
   permission**: Cloture does not hold it until you press the button, and your browser asks
   you first. You can decline, and the rest of the extension works normally.
2. It builds a summary containing, for each of up to 60 hostnames: the hostname, a visit
   count, and a coarse timing shape (weekday working hours / off hours / late night).
3. **You can see the exact text that would be sent before anything is sent**, using the
   "Preview what would be sent" button.
4. The summary and your own description of your work are sent to the AI provider you selected,
   using your own API key, over HTTPS. Requests are pinned to zero-data-retention endpoints and
   are configured to fail rather than fall back to a provider that retains data.
5. The reply is a list of suggestions. **Nothing is applied automatically** — you tick what you
   accept.

Your relationship with that AI provider is governed by your own agreement with them. Cloture
is not a party to it and receives nothing from it.

Every run is logged locally — model, endpoint, provider options, number of hostnames sent — and
that log is included in your JSON export. Your API key is not.

## Permissions, and why each one exists

| Permission | Why it is needed |
|---|---|
| `storage` | To save your limits and your daily counts on your computer |
| `tabs` | To read the hostname of the tab currently in front of you. Cloture does not read tab contents |
| `alarms` | To bank elapsed time roughly every 30 seconds |
| `idle` | To stop counting when you walk away from the keyboard |
| `notifications` | To show the single warning at 60% of a limit |
| Access to websites (`<all_urls>`) | To display the block screen on a site you have limited. Cloture does not read, alter, or transmit page content |
| `history` *(optional)* | Only for Smart suggestions, only after you grant it, and only to count visits per hostname |

## Children

Cloture is not directed at children and collects no personal information from anyone.

## Changes

Material changes to this policy will be published here with an updated date, and will appear in
the extension's store listing before the version that implements them is released.

## Contact

Questions about this policy: **[your contact email]**

Source code: **[link to your repository, if public]**
