# Cosmic Frontier

![CI](../../actions/workflows/ci.yml/badge.svg) ![License: MIT](https://img.shields.io/badge/license-MIT-blue)

An astronomy learning hub with real user accounts, an interactive front end and a security-hardened,
**dependency-free** Node.js server (Node 18+; no `npm install` needed).

## Run it

```bash
node server.js          # http://127.0.0.1:3000
npm test                # 14 backend tests (node:test)
```

The site must be opened through the server (not by double-clicking the HTML files), because accounts,
sessions and page templating live on the server.

## Project layout

```
server.js            HTTP server, templating, API, sessions, security headers
views/layout.html    shared header / footer / head (edit nav & footer once)
views/pages/*.html   page content fragments (first line = title/description/scripts)
public/css/style.css design system (no inline styles anywhere)
public/js/           core.js (shared), starfield.js, orrery.js, stars.js, discover.js, assistant.js, account.js
test/                backend tests
data/db.json         created on first sign-up (users + hashed sessions) — back this up, never commit it
```

## What's new

**Visual:** animated parallax starfield, interactive solar-system orrery, life-cycle explorer for
Sun-like vs massive stars, star colour/temperature slider with spectral classes, light-travel-time scale,
CSS-drawn art for every card (no external image dependencies), scroll reveals, animated counters, timeline
progress line. Respects `prefers-reduced-motion`.

**Structure:** one shared layout instead of eight copy-pasted headers; clean URLs (`/missions`, old
`*.html` links redirect); per-page scripts; working filter chips + search on every card page; accessible
landmarks, skip link, `aria-current`, keyboard-operable widgets.

**Accounts:** register, log in/out, saved items (bookmark any mission/topic/news item), change password,
delete account.

## Security

- Passwords hashed with **scrypt** (N=32768, r=8, p=1, per-user random salt; parameters stored with the hash).
- Sessions are random 256-bit tokens; only their **SHA-256 hash** is stored. Cookie is `HttpOnly`,
  `SameSite=Lax`, `Secure` + `__Host-` prefix in production. Sessions rotate on login, expire after 7 days,
  and other sessions are revoked on password change.
- **CSRF:** Origin / Sec-Fetch-Site check, JSON-only content type, and a per-session token header on all
  authenticated writes.
- **Rate limiting:** per-IP and per-account login lockouts, sign-up limit, global API limit.
  Unknown e-mails cost the same time as wrong passwords (no timing oracle) and share one generic error.
- **CSP with no `unsafe-inline`:** no inline scripts, styles or handlers (a test enforces this), plus
  `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`, COOP/CORP, HSTS in production.
- All user-controlled text is rendered with `textContent`; request bodies capped at 8 KB; static serving
  is allow-listed by extension with a path-traversal guard; external links use `rel="noopener noreferrer"`.

## Deploying

```bash
NODE_ENV=production TRUST_PROXY=1 HOST=127.0.0.1 PORT=3000 node server.js
```

Put it behind a TLS-terminating reverse proxy (nginx, Caddy, …) that **sets** `X-Forwarded-For`.
Environment variables: `PORT`, `HOST`, `NODE_ENV`, `TRUST_PROXY`, `DATA_DIR`, `COOKIE_SECURE`.

## Known limits (be honest with your users)

- Storage is a JSON file: fine for one instance and modest traffic; move to Postgres/SQLite for more.
- **No email verification or password reset** (needs an email provider). Until then, a forgotten password
  cannot be recovered, and sign-up reveals whether an email is registered (rate-limited).
- No 2FA. The assistant is an offline keyword FAQ, not a live AI model.

## Content to double-check before publishing

I kept your original text and links. These items looked worth verifying against the source:

1. **Missions → Artemis IV** links to `.../artemis-ii-news-and-updates/` (an Artemis II URL).
2. **Research News** – three NASA items (Hubble lenses, LISA, Hubble oxygen) all point to the same generic
   exoplanet press-release index rather than their own articles.
3. **Missions → Roman** is labelled "Launched: 2026"; confirm that is still accurate.

## Docker

```bash
docker build -t cosmic-frontier .
docker run -p 3000:3000 -v cf-data:/data cosmic-frontier
```

> **GitHub Pages note:** this app has a Node backend (accounts, sessions), so it cannot be hosted on
> GitHub Pages, which serves static files only. Deploy to a Node/Docker host (Render, Railway, Fly.io,
> a VPS, etc.) and use GitHub for source and CI.
