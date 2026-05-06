# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**給力盒子 (GetPower Box)** — a static, serverless lunch ordering system for group orders. No build step, no package manager. Open `index.html` directly in a browser or serve with any static file server.

## Development & Deployment

- **Run locally**: open `index.html` in a browser, or `npx serve .` / `python -m http.server`
- **Deploy**: `firebase deploy` — deploys to project `lunchbox-de9f9`. The app root is the repo root, not `public/`
- `public/index.html` is the unused Firebase Hosting default template (not the actual app)

## Architecture

Single-page application in `index.html` with three in-page views toggled by `switchView()`:

- `#view-home` — landing page with menu photo (`getpower.jpg`) and navigation buttons
- `#view-order` — order form with quantity selectors, rice preference, and custom requests
- `#view-summary` — order statistics dashboard (tabs: 訂購總表 / 個人訂單 / 餐點統計)

`order.html` is a legacy standalone version with no Firebase integration.

### Data Flow

1. User fills out `#view-order`
2. `submitOrder()` writes to **Firebase Realtime Database** (compat SDK v10.7.1, loaded via CDN)
3. A global `ordersCache` array is kept in sync via an `onValue()` real-time listener on `dbRef`
4. `#view-summary` calls `renderSummary()` which reads from `ordersCache`

Firebase DB path is hardcoded as `lunchbox_orders_2026_03`. Each order is pushed as a flat object with `{ id, name, rice, special, items, total, time }`.

### Menu Structure

Two sections in `#view-order`:

1. **選擇餐盒** — full bento boxes (with rice)
2. **單點主菜** — à la carte mains; `data-name` values carry a `(單點)` suffix (e.g. `"鹽烤鯖魚(單點)"`)

Both sections use `.menu-card` divs with `data-name` and `data-price` attributes. Categories use CSS border-left color coding:
- `category-fish` (blue) — 海鮮類
- `category-chicken` (green) — 雞肉類
- `category-pork` (yellow) — 豬肉類
- `category-beef` (red) — 牛肉類
- `category-veg` (emerald) — 蔬菜類

Rice options (checkboxes, `name="rice"`): 正常飯, 半飯, 無飯, 飯換地瓜, 全素, 蛋奶素

### Styling

Uses a local Tailwind CSS file (`tailwind 1.css` — note the space in filename). Custom styles are inline `<style>` blocks. Primary color: orange `#f97316`.

## Key Functions in index.html

- `switchView(to)` — core view switcher; `showOrder()` / `showSummary()` / `goBack()` wrap it with header UI updates
- `changeQty(btn, delta)` / `updateTotal()` — quantity controls and running total
- `submitOrder()` — validates, writes to Firebase, shows confirmation modal
- `renderSummary()` — renders all three summary tabs from `ordersCache`
- `switchTab(tab)` — toggles between 訂購總表 / 個人訂單 / 餐點統計 panels
- `deleteOrder(firebaseKey)` / `clearAllOrders()` — Firebase deletions; `onValue` listener auto-refreshes UI
- `toggleCustomReq()` / `updateCustomReqDisplay()` — dropdown for preset dietary customizations

## Adding Menu Items

Copy an existing `.menu-card` div and update `data-name`, `data-price`, and display text. Assign the appropriate `category-*` CSS class. No JavaScript changes needed — `updateTotal()` and `submitOrder()` iterate all `.menu-card` elements dynamically.

For à la carte items, append `(單點)` to the `data-name` value so the summary view groups them correctly.
