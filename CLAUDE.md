# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A single-page static tracker for a Polish long-distance hiking trail (Główny Szlak Beskidzki 2026, Wołosate → Ustroń). The entire app is one self-contained `index.html` file — no build step, no package manager, no dependencies beyond CDN-loaded Leaflet and Google Fonts.

## Development

There is no build/lint/test tooling. To work on this site, edit `index.html` directly and open it in a browser (or serve it with any static file server, e.g. `python -m http.server`). Password-gated crypto (`crypto.subtle.digest`) requires a secure context, so `file://` works in most browsers but if it doesn't, serve over `http://localhost`.

## Architecture

`index.html` contains three inline sections: CSS (`<style>`), markup, and JS (`<script>`). Key pieces of the JS:

- **`CONFIG`** — GitHub owner/repo/branch and a SHA-256 hash of the site password (`passHash`). Visiting the page with `?genhash=your-password` prints the hash to paste back into `CONFIG.passHash`.
- **`DAYS`** — hardcoded array of the trip itinerary (21 stages: date, from/to, km, GOT points, overnight stop, lat/lng). This is the source of truth for the itinerary; edit this array to change stages/distances/points.
- **`progress.json`** — the only persisted state, `{ completed: [day numbers], updated: <timestamp string> }`. Fetched client-side on load (`loadProgress()`, polled every 5 min) to render progress; **not** derived from `DAYS`.
- **Password gate** — client-side only (hashes the entered password and compares to `CONFIG.passHash`), gates the `#app` view. This is obfuscation, not real security — the itinerary/JS is always in the page source.
- **Admin mode** — toggled via a link at the bottom; prompts for a GitHub fine-grained PAT (stored in `localStorage`, never sent anywhere but the GitHub API). In admin mode, clicking a day's checkbox toggles completion locally, and "Zapisz postęp" (`saveBtn` handler) writes the updated `progress.json` straight to the repo via the GitHub Contents API (`PUT /repos/{owner}/{repo}/contents/progress.json`), committing directly to `CONFIG.ghBranch`. This is how the tracker is updated in production — there's no server/backend.
- **Map** — Leaflet map (OpenStreetMap tiles) plotting `START` + each day's `lat/lng`, with a red solid polyline for completed segments and a dashed gray polyline for remaining ones, computed from a contiguous run of completed days starting at day 1 (`renderMapProgress()`).
- **Progress UI** — a trail-blaze-styled progress bar, three stat cards (days/km/GOT points), and a per-day list, all derived by `render()` from `DAYS` + the `completed` set.

When editing the itinerary (`DAYS`), keep `progress.json`'s `completed` array consistent with day numbers (`d` field), and note that map progress assumes completed days form an unbroken prefix from day 1.
