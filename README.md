# WTF $1 — Provably-Fair Crypto Casino (Full Platform For Sale)

**A complete, self-hosted crypto casino in a single Node.js app.** Seven instant-play
provably-fair games plus a real-time multiplayer Crash Royale, three progressive must-win
jackpots, an on-site crypto wallet with
automated deposits & payouts, and a full operator admin panel — no third-party game
vendor, no per-spin licensing fees, no revenue share.

> This is the public showcase. **The full source is private** and delivered to the buyer
> after purchase (see [How to buy](#how-to-buy)). This is proprietary software, not
> open source — see [LICENSE](LICENSE) and [EULA.md](EULA.md).

**Live demo:** https://wtf1dollar.com

![WTF $1 lobby](docs/screenshots/lobby.png)

---

## Why buy

- **You own the code.** No game-provider revenue share, no SaaS seat fees, no lock-in.
- **Tiny ops footprint.** One Node process + one file-based SQLite DB — runs on a $5 VPS,
  Docker, or any managed Node host. No DB server, Redis, or build step.
- **Provably fair & auditable.** Every outcome is server-authoritative and independently
  verifiable (`HMAC-SHA256`); clean service/route/config layering.
- **Ready to operate.** Docker + systemd + nginx deploy assets, one-command deploy script,
  full docs, SEO/PWA, and admin economics + integrity checks included.

## What's inside

| Area | Details |
|---|---|
| **Games** | Dice · Plinko · Mines · Crash · Coin Flip · The WTF Button (mystery multiplier to 1000x) · WTF Scratcher (3×3 scratch card) · Crash Royale (real-time multiplayer PvP) |
| **Jackpots** | Mini / Mega / Grand — Mega Moolah-style must-win drops, opt-in from any game, self-reseeding |
| **Wallet** | USD-denominated on-site balance; automated NOWPayments deposits (hosted invoice + signed IPN) and Mass Payout withdrawals (USDT BEP-20 / BSC) |
| **Fairness** | Per-game provably-fair engine + per-bet idempotency (a dropped connection mid-bet never double-charges) |
| **Retention** | Lucky Wheel free spins, referrals, login streaks, leaderboards, scheduled pot-boost events |
| **Admin** | Grouped dashboard: per-game stats & live settings, house P&L, integrity checks, live activity, payouts, withdrawals, users, audit log, CSV export |
| **Real-time** | Socket.io jackpot count-ups, cross-game activity ticker, canvas game animations, winner confetti |
| **Growth** | Per-page SEO meta + JSON-LD, sitemap, PWA manifest, social share image |

## Tech stack

Node.js >= 22.5 · Express 4 · Socket.io 4 · built-in `node:sqlite` (WAL, no native build)
· `bcryptjs` + `express-session` · `otplib` TOTP payouts · vanilla-JS frontend (no framework,
no bundler) · NOWPayments (payments) · CoinGecko (pricing).

## Screenshots

![The WTF Button — flagship mystery-multiplier game](docs/screenshots/button.png)
![Crash Royale — real-time multiplayer PvP](docs/screenshots/arena.png)
![Admin dashboard](docs/screenshots/admin-dashboard.png)

## Pricing

| Tier | Price | What you get |
|------|-------|--------------|
| **Non-exclusive license** | Contact us | Full source; deploy your own brand. Reselling/redistribution not permitted. |
| **Exclusive sale (full transfer)** | Contact us | Full source + IP transfer; the seller stops selling. |
| **Deployment + 30-day support** | Contact us | Get live this week; setup help and fixes. |

## How to buy

1. Open an issue or contact the seller (see profile) with your preferred tier.
2. We agree terms and use **escrow** (Escrow.com / Flippa / crypto escrow) — no direct
   pre-payment required.
3. On payment release you receive the private repo (read collaborator invite for
   non-exclusive, or full repository transfer for exclusive) plus deploy docs.

> **Compliance.** Online gambling is regulated in most jurisdictions. Obtain legal review
> and any required licensing before operating with real money. 18+ only.
