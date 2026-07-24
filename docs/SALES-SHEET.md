# WTF $1 — Provably-Fair Crypto Casino · Sales Sheet

**A complete, self-hosted crypto casino in a single Node.js app.** Eight instant-play
provably-fair games, three progressive must-win jackpots, an on-site crypto wallet with
automated deposits & payouts, and a full operator admin panel — no third-party game
vendor, no per-spin licensing fees, no revenue share.

**Live demo:** <https://wtf1dollar.com>

![WTF $1 lobby](./screenshots/lobby.png)

---

## Why buy

- **You own the code.** No game-provider revenue share, no SaaS seat fees, no lock-in.
- **Tiny ops footprint.** One Node process + one file-based SQLite DB — runs on a $5 VPS, Docker, or any managed Node host. No DB server, Redis, or build step.
- **Provably fair & auditable.** Every outcome is server-authoritative and independently verifiable (`HMAC-SHA256`); clean service/route/config layering.
- **Ready to sell/operate.** Docker + systemd + nginx deploy assets, one-command deploy script, full docs, SEO/PWA, and admin economics + integrity checks included.

## What's inside

| | |
|---|---|
| **Games** | Dice · Plinko · Mines · Crash · Coin Flip · The WTF Button (mystery multiplier to 1000×) · WTF Scratcher (3×3 scratch card) · Crash Royale (real-time multiplayer PvP crash) |
| **Jackpots** | Mini / Mega / Grand — Mega Moolah-style must-win drops, opt-in from any game, self-reseeding |
| **Wallet** | USD-denominated on-site balance; automated NOWPayments deposits (hosted invoice + signed IPN) and Mass Payout withdrawals (USDT BEP-20 / BSC, ~$0.02 fee) |
| **Fairness** | Per-game provably-fair engine + per-bet idempotency (a dropped connection mid-bet never double-charges) |
| **Retention** | Lucky Wheel free spins, referrals, login streaks, leaderboards, scheduled pot-boost events |
| **Admin** | Grouped dashboard: per-game stats & live settings, house P&L, integrity checks, **live activity** (who's online, which page), payouts, withdrawals, users, audit log, CSV export |
| **Real-time** | Socket.io jackpot count-ups, cross-game activity ticker, canvas game animations, winner confetti |
| **Growth** | Per-page SEO meta + JSON-LD, sitemap, PWA manifest, social share image |

## Tech stack

Node.js ≥ 22.5 · Express 4 · Socket.io 4 · built-in `node:sqlite` (WAL, **no native build**)
· `bcryptjs` + `express-session` · `otplib` TOTP payouts · vanilla-JS frontend (no framework, no bundler)
· NOWPayments (payments) · CoinGecko (pricing).

## Ships with

Full source (server, all game logic, admin, frontend) · all `config/*.json` (games, jackpots,
wheel, coins) · schema + auto-migrations & seed · `Dockerfile`, `docker-compose.yml` (auto-HTTPS),
systemd unit, nginx config, PowerShell deploy script · audit/RTP & smoke-test scripts · full
[documentation set](./README.md).

---

![The WTF Button — flagship mystery-multiplier game](./screenshots/button.png)
![Admin dashboard](./screenshots/admin-dashboard.png)

*More: the full [screenshot gallery](./screenshots/README.md) and [product overview](./01-product-overview.md).*

> ⚠️ **Compliance.** Online gambling is regulated in most jurisdictions. Obtain legal
> review and any required licensing before operating with real money. 18+ only. License
> terms are agreed explicitly with the seller.
