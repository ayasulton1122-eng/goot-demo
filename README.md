# GOOT Resorts — Premium Cinematic Build ($1M edition)

## Front end
- **Cinematic hero**: continuous Kling video background (battery-aware: pauses when tab hidden, resumes on interaction if autoplay is blocked).
- **Premium design system**: obsidian/gold palette, Playfair Display + Amiri (Arabic headings), custom lerped cursor, scroll progress bar, luxury easing curves.
- **Sections**: hero → manifesto marquee → intro story with stats → **interactive villa selector** (Sawsan/Jouri/Orchid/Fayrouz/Lazurd with live image crossfade + localized prices) → spa rituals → dining → location → **gallery with lightbox** → guest testimonials → FAQ accordion → booking.
- **Bilingual AR/EN** with full RTL mirroring (logical CSS properties where it matters), persisted preference.
- **Accessibility**: skip link, focus management in lightbox (Escape/arrows + focus restore), `prefers-reduced-motion` respected, aria labels/roles.

## Back end
- Express 5 + PostgreSQL, Helmet CSP headers, strict 20 KB JSON, API rate limiting.
- **Security hardening added**: request/response timeouts (slow-loris resistant), same-origin enforcement on all API writes (CSRF), dotfile blocking (`.env`/`.git` never served), malformed-JSON 400s, graceful shutdown, reduced server fingerprinting.
- PostgreSQL **exclusion constraint** prevents overlapping active reservations per villa.
- Parameterized SQL everywhere, strict input validation.

## Run
1. Install Node.js 20+ and PostgreSQL.
2. Create database `goot`, run `db/schema.sql` against it.
3. Copy `.env.example` to `.env` (set `DATABASE_URL`).
4. `cd server && npm install && npm start`
5. Open `http://localhost:3000`

## Tests (no database required)
```bash
cd server && npm install
node --test tests/server.test.mjs
```
Covers: security headers, dotfile blocking, CSRF origin guard, payload validation, date-range validation, malformed JSON handling.

## Production notes
- Serve behind HTTPS/reverse proxy; set `NODE_ENV=production`.
- Use a dedicated PostgreSQL user with least privileges; store secrets in env vars only.
- Add authenticated admin endpoints before exposing booking management.
- Back up PostgreSQL and log booking/service events without storing unnecessary personal data.

## Verified GOOT contact/location data
- Phone: 920007811 · Email: info@gootresorts.com
- Address: Al Munsiyah District, Al Sahabah Street, Riyadh, Saudi Arabia
- Instagram: https://instagram.com/gootresorts · Social hub: https://linktr.ee/gootresort
