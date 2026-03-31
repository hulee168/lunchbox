# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**給力盒子 (GetPower Box)** — a static, serverless lunch ordering system for group orders. No build step, no package manager. Open HTML files directly in a browser or serve with any static file server.

## Architecture

This is a **single-page application** built entirely in `index.html` with three in-page views toggled by JavaScript:

- `#view-home` — landing page showing the menu photo (`getpower.jpg`) and navigation buttons
- `#view-order` — order form with quantity selectors, rice preference, and custom requests
- `#view-summary` — order statistics dashboard showing all submitted orders

`order.html` is a legacy standalone version of the order page (no Firebase, local modal only).

### Data Flow

1. User fills out order form in `#view-order`
2. On submit, `submitOrder()` writes to **Firebase Realtime Database** (compat SDK v10.7.1, loaded via CDN)
3. `#view-summary` reads all orders from Firebase in real time using `onValue()`
4. Firebase config is embedded directly in `index.html` in the `<script>` block

### Menu Data

Menu items are hardcoded as HTML elements with `data-name` and `data-price` attributes on `.menu-card` divs. Categories use CSS border-left color coding:
- `category-fish` (blue) — 海鮮類
- `category-chicken` (green) — 雞肉類
- `category-pork` (yellow) — 豬肉類
- `category-beef` (red) — 牛肉類
- `category-veg` (emerald) — 蔬菜類

### Styling

Uses a local Tailwind CSS file (`tailwind 1.css` — note the space in filename). Custom styles are written inline in `<style>` blocks within each HTML file. The design uses an orange (`#f97316`) primary color scheme.

## Key Functions in index.html

- `showOrder()` / `showSummary()` / `goBack()` — view switching
- `changeQty(btn, delta)` — quantity +/- buttons for menu cards
- `updateTotal()` — recalculates running total and highlights selected cards
- `submitOrder()` — validates form, writes order to Firebase, shows confirmation modal
- `loadSummary()` — fetches all orders from Firebase and renders statistics
- `toggleCustomReq()` / `updateCustomReqDisplay()` — dropdown for dietary customizations

## Firebase

The app requires a Firebase project with Realtime Database enabled. The config object (`apiKey`, `databaseURL`, etc.) must be present in `index.html`. Orders are stored under a single path (e.g., `/orders`) as flat objects keyed by `push()` IDs.

## Adding Menu Items

To add a dish, copy an existing `.menu-card` div and update `data-name`, `data-price`, and the display text. Assign the appropriate `category-*` CSS class for the left border color. No JavaScript changes needed — `updateTotal()` and `submitOrder()` iterate over all `.menu-card` elements dynamically.
