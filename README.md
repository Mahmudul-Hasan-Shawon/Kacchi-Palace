<div align="center">

# 👑 Kacchi Palace

**The Royal Taste of Dhaka: a storefront for authentic dum-style kacchi biryani, delivered hot across Dhaka.**

A dependency-free, multi-page restaurant website with a live menu, shopping bag,
mobile-wallet checkout, order tracking and printable invoices. The entire menu,
pricing and store configuration are managed from a Google Sheet: no server to run,
no database to host.

[GitHub](https://github.com/Mahmudul-Hasan-Shawon/kacchi-palace-website) ·
[Report an issue](https://github.com/Mahmudul-Hasan-Shawon/kacchi-palace-website/issues)

</div>

![HTML5](https://img.shields.io/badge/HTML5-static%20pages-E34F26?logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-4.8k%20lines-1572B6?logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-vanilla%20ES5%20safe-F7DF1E?logo=javascript&logoColor=black)
![Google Apps Script](https://img.shields.io/badge/Backend-Google%20Apps%20Script-4285F4?logo=google&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Database-Google%20Sheets-34A853?logo=googlesheets&logoColor=white)
![Font Awesome](https://img.shields.io/badge/Icons-Font%20Awesome%206.5.2-528DD7?logo=fontawesome&logoColor=white)
![Lenis](https://img.shields.io/badge/Smooth%20scroll-Lenis%201.3.13-111111)

---

## ✨ Features

### 🍛 Ordering (index.html)

| Feature | Details |
| --- | --- |
| Live menu | Products and store config fetched from a Google Apps Script endpoint (`getAll`), with loading, error + retry, and empty states |
| Search and filters | Free-text search across title, brand, category, SKU and tags; category pills; brand/category side panels on mobile; one-click "Clear Filter" |
| Product modal | Image, pricing with discount badge, stock indicator, quantity stepper (1 to 99), and auto-parsed "Package includes" lists for party dala packages |
| Cart drawer | Persisted in `localStorage` (`kp_cart`), quantity edit, animated row removal, fly-to-cart dot animation, badge on desktop and mobile navs |
| Cart reconciliation | On every load the saved cart is checked against the live menu: sold-out, deleted or repriced items are corrected or dropped with a toast |
| Checkout | Name, phone, address, delivery zone (Inside Dhaka / Outskirts) and payment method; delivery charge applies live to the total |
| Payments | Cash on Delivery, bKash and Nagad. Wallet orders require account number and transaction ID, with the merchant account number shown from config |
| Order confirmation | Success screen with Order ID and Tracking Code, confetti burst, and a warning note if the confirmation email failed server-side |
| Order tracking | Look up by Order ID or Tracking Code (`trackOrder`); six-step timeline (Processing, Confirmed, Packed, Shipped, Out for Delivery, Delivered) with a dedicated Cancelled state |
| Invoice | On-page invoice that mirrors the server-emailed PDF field for field, with isolated print (only the invoice goes to the printer) |

### 📖 Content pages

| Page | What's inside |
| --- | --- |
| [about.html](about.html) | Brand story ("A Feast Fit for Royalty, Delivered"), value cards and a stat strip, since 2018 |
| [gallery.html](gallery.html) | Filterable photo gallery: All, Kacchi & Rice, Roast & Kabab, Bhuna & Curry, Party Dala, Dessert, Drinks, with captions |
| [faq.html](faq.html) | Single-open accordion FAQ with a call-to-action block |
| [contact.html](contact.html) | Contact info cards, map note, and a validated contact form (name, phone, message) with a success state |

### 🛍️ Shared storefront shell

Every page ships the same nav, cart drawer, checkout, tracking and success overlays
driven by one shared [assets/site.js](assets/site.js), so customers can start an
order on any page and finish it there. A mobile bottom nav (Brand, Categories,
Cart, Track Order, WhatsApp Chat) mirrors the desktop actions.

### 🎨 Experience

| Feature | Details |
| --- | --- |
| Smooth scrolling | Lenis smooth scroll with a scroll progress bar and sticky navbar state |
| Motion | Scroll-reveal via IntersectionObserver, hero image marquee, floating sparks and petals, count-up stats, staggered card entries |
| Reduced motion | Every animation honors `prefers-reduced-motion`: reveals, confetti, sparks and fly-to-cart all disable cleanly |
| Accessibility | ARIA labels on icon buttons, keyboard-operable logo, Escape closes only the topmost overlay, `aria-live` toast |
| Bilingual | English UI with Bangla content support: dedicated Bangla webfonts (Hind Siliguri, Anek Bangla), Bangla delivery notice and Bangla package contents from the sheet |

### ⚙️ Sheet-driven configuration

Store config arrives with the menu payload and flows through the whole site:
site title, WhatsApp number (normalized to `wa.me` links in nav, mobile nav and
footer), currency symbol (defaults to ৳), delivery charges (defaults: ৳60 inside
Dhaka, ৳120 outskirts), and bKash / Nagad account numbers shown at checkout.

---

## 🏗️ Architecture

```
 ┌─────────────────────────────┐  GET ?action=getAll          ┌─────────────────────────┐
 │   Browser (static site)     │ ───────────────────────────► │  Google Apps Script     │
 │                             │  GET ?action=getProducts     │  Web App (Code.gs)      │
 │  index.html    storefront   │ ───────────────────────────► │                         │
 │  about / gallery /          │  GET ?action=trackOrder      │  validates orders,      │
 │  faq / contact              │ ───────────────────────────► │  emails confirmation,   │
 │  assets/site.js   engine    │                              │  returns order +        │
 │  assets/styles.css  theme   │  POST {action:"submitOrder"} │  tracking code          │
 └──────────┬──────────────────┘ ◄─────────────────────────── └───────────┬─────────────┘
            │                                                             │
            │  cart cached in localStorage ("kp_cart")                    │ reads / writes
            │  wa.me / tel: deep links for calls and chat                 ▼
            │                                                 ┌─────────────────────┐
            └────────────────────────────────────────────────►│   Google Sheet      │
                                                             │  menu · config ·    │
                                                             │  orders · status    │
                                                             └─────────────────────┘
```

One `API_URL` constant at the top of [assets/site.js](assets/site.js) is the only
wiring. Notable engineering choices documented in the code:

- **No CORS preflight**: the order `POST` is sent without a `Content-Type` header
  so it stays a "simple" request, sidestepping the preflight that Apps Script web
  apps cannot answer.
- **XSS hardening**: product data, category names and customer-entered addresses
  (rendered on the tracking screen) all pass through `esc()` / `jsStr()` before
  touching `innerHTML`.
- **Locale-safe money**: amounts are grouped manually instead of
  `toLocaleString()`, so a phone set to Bangla never renders Bengali numerals in
  the cart while the emailed invoice uses Latin ones.
- **Defensive storage**: every `localStorage` access is wrapped; private-mode
  browsers degrade instead of facing a blank page.
- **Overlay discipline**: a shared scroll lock syncs page freezing across all
  seven drawers/modals, and Escape peels them off topmost-first.

---

## 📁 Project structure

```
Kacchi Palace/
├── index.html          # Storefront: hero, live menu, cart, checkout, tracking, invoice
├── about.html          # Brand story, values, stats strip
├── gallery.html        # Filterable photo gallery (7 categories)
├── faq.html            # Accordion FAQ + CTA
├── contact.html        # Contact cards + validated form
└── assets/
    ├── site.js         # Storefront engine: API client, cart, checkout, tracking (1k lines)
    └── styles.css      # Full design system on CSS custom properties (4.8k lines)
```

No build step, no bundler, no `node_modules`. The site is plain files served as-is.

---

## 🧰 Tech stack

| Layer | Technology |
| --- | --- |
| Markup | Static HTML5, five pages |
| Styling | Hand-written CSS3 (custom properties, grid/flex, keyframe animation, print styles) |
| Logic | Vanilla JavaScript (ES5-compatible), no framework |
| Backend | Google Apps Script Web App (deployed as "Execute as: Me, access: Anyone") |
| Database | Google Sheet (menu, store config, orders, shipping status) |
| Fonts | Google Fonts: Cormorant Garamond, Manrope, Cinzel, Bodoni Moda, Hind Siliguri, Anek Bangla |
| Icons | Font Awesome 6.5.2 via cdnjs |
| Smooth scroll | Lenis 1.3.13 via unpkg |

---

## 🚀 Getting started

### Prerequisites

- Any modern browser
- A way to serve static files (VS Code Live Server, or one of the commands below)
- Optional, for backend work: a Google account for Sheets + Apps Script

### Run locally

```bash
# option 1: Python
python -m http.server 8000

# option 2: Node
npx serve .
```

Then open <http://localhost:8000>. Opening `index.html` directly from disk is
discouraged: the menu, tracking and checkout all depend on `fetch` to the Apps
Script endpoint.

There are no install, typecheck or test commands: the project has zero build-time
dependencies. Serving the files is the whole setup.

### Services and URLs

| Service | URL | Notes |
| --- | --- | --- |
| Storefront | `http://localhost:8000` | All five pages |
| Menu API (read) | `{API_URL}?action=getAll` | Products + store config |
| Tracking API | `{API_URL}?action=trackOrder&id=...` | Order lookup by ID or tracking code |
| Order API (write) | `POST {API_URL}` | Placed at checkout |

`API_URL` is set at the top of [assets/site.js](assets/site.js#L20); a live
deployment is already configured, so the site works out of the box.

### Connecting your own Google Sheet backend

The connection recipe lives in the header comment of
[assets/site.js](assets/site.js#L1): import the `Kacchi_Palace_Google_Sheet.xlsx`
workbook into Google Drive, open it as a Google Sheet, paste the `Code.gs` script
under Extensions → Apps Script, deploy as a Web App (Execute as: Me, Who has
access: Anyone), and paste the resulting `/exec` URL into `API_URL`. The workbook
and `Code.gs` are referenced from there but are not part of this repository.
When updating an existing deployment, use Manage deployments → Edit → New version
so the URL stays the same.

---

## 📦 Deployment

The site is pure static files, so any static host works.

**GitHub Pages** (this repository's natural home):

1. Push the repository to GitHub.
2. Settings → Pages → Deploy from a branch → `main` / `(root)` → Save.
3. The site goes live at `https://<username>.github.io/kacchi-palace-website/`.

No build command, no output directory, nothing to configure. Netlify and Vercel
work the same way with an empty build config (drag-and-drop the folder onto
Netlify also works).

**Backend note:** the Apps Script deployment is independent of the frontend host.
If you redeploy the script, keep the same `/exec` URL and the site needs no
changes; if you create a new deployment, update `API_URL` in
[assets/site.js](assets/site.js#L20).

---

## 🔌 API overview

All traffic goes to a single Apps Script endpoint. The frontend calls three
`GET` actions and one `POST`:

| Action | Method | Purpose | Returns |
| --- | --- | --- | --- |
| `getAll` | GET | Full menu + store config for the storefront | `{ products: [...], config: {...} }` |
| `getProducts` | GET | Silent refresh after an order, so stock levels catch up | `[...]` |
| `trackOrder` | GET | Order lookup by Order ID or Tracking Code | `{ found, orderNumber, shippingStatus, items, subtotal, deliveryCharge, total, ... }` |
| `submitOrder` | POST | Place an order (customer info, items, totals, payment details) | `{ orderNumber, trackingCode, emailSent }` |

<details>
<summary><strong>Product and config fields consumed by the frontend</strong></summary>

<br />

Each product in the `getAll` payload may carry:

| Field | Used for |
| --- | --- |
| `id`, `title`, `brand`, `category`, `sku` | Grid, filters, search, modal |
| `size`, `unit`, `tags` | Size line, image fallback matching, package contents parsed after an `Includes:` marker |
| `imageUrl` | Product image, falling back to a keyword-matched category image |
| `oldPrice`, `offerPrice`, `displayPrice`, `hasDiscount`, `discountPct` | Pricing with strikethrough and sale badge |
| `inStock` | Stock tag, disabled add buttons, cart reconciliation |

Store `config` fields:

| Field | Used for |
| --- | --- |
| `siteTitle` | Document title and nav logo |
| `whatsAppNumber` (or `whatsappNumber`) | All `wa.me` deep links |
| `currencySymbol` | Every money render (default ৳) |
| `insideDhakaCharge`, `outsideDhakaCharge` | Checkout delivery totals (defaults 60 / 120) |
| `bKashAccount`, `nagadAccount` | Wallet payment instructions |
| `totalProducts`, `happyOrders` | Order payload sizing, hero stats |

</details>

<details>
<summary><strong>Order POST payload (key fields)</strong></summary>

<br />

`action`, `firstname`, `lastname`, `fullname`, `contactnumber`, `address`,
`delivery_location` (`inside` / `outside`), `services` (payment method),
`account_number`, `transaction_id`, `subtotal` / `delivery_charge` / `total`
(both formatted and numeric), `products` (rendered list), `quantities`,
`quantitiesArray` (per-product quantity vector, grown past the sheet's
`totalProducts` so late-added items are never truncated), `totalItems`.

</details>

The source of truth for all of the above is [assets/site.js](assets/site.js):
see `loadData()`, `placeOrder()` and `doTrack()`.

---

## 🍔 Why "Kacchi Palace"

Kacchi biryani is Dhaka's celebration dish: mutton and fragrant polao rice sealed
together and slow-cooked dum style, made for weddings and feasts from 2 guests to
2,000. This site exists so a family-run kitchen can take those orders directly:
the menu lives in a spreadsheet the owners already know how to edit, orders land
in that same sheet with email confirmations, customers get a tracking code like
any big delivery app, and the whole thing costs nothing to host.

<p align="center"><em>Slow-cooked in true dum style · Open daily 9AM to 10PM · <a href="https://wa.me/8801772241694">+880 1772-241694</a></em></p>
