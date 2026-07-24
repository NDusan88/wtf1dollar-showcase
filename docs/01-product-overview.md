# 1. Product Overview

## What it is

**WTF $1** is a complete, self-hosted **provably-fair crypto casino** delivered as a
single Node.js application. It ships eight instant-play games, three progressive
"must-win" jackpots, an on-site crypto wallet with automated deposits and payouts, a
full operator admin panel, and a polished real-time frontend — all with no third-party
game vendor and no per-spin licensing fees.

The entire platform runs as **one Node process** backed by a **file-based SQLite
database**, so it can be deployed on a $5 VPS, a Docker host, or any managed Node
platform without external database or message-broker infrastructure.

- **Live reference deployment:** <https://wtf1dollar.com>

![WTF $1 lobby — biggest jackpot banner and game grid](./screenshots/lobby.png)

> More screenshots — every game, the Lucky Wheel, and the admin panel — in the
> [screenshot gallery](./screenshots/README.md).

## Who it's for

- Operators launching a **crypto-first casino / social-casino** brand quickly.
- Entrepreneurs who want to **own the code** (no revenue share, no SaaS lock-in).
- Developers who need a **well-structured, auditable** base to extend with new games.

## The games

All eight games share one fairness engine, one wallet, a configurable house edge
(1% default), optional jackpot opt-in, manual + auto-bet, and per-bet idempotency
(a dropped connection mid-bet never double-charges).

| Game | Mechanic | Configurable in |
|------|----------|-----------------|
| **Dice** | Pick a target (2–98), roll Under/Over; result `0.00–99.99` | `config/dice.json` |
| **Plinko** | Drop a ball through 8/12/16 rows at low/med/high risk; payout tables auto-scaled so the binomial EV always holds the house edge | `config/plinko.json` |
| **Mines** | Reveal safe tiles on a 5×5 grid with 1–24 mines; multipliers from survival probability; cash out anytime | `config/mines.json` |
| **Crash** | Cash out (or auto-cashout) before the rising multiplier busts; `1.01×`–`100000×` | `config/crash.json` |
| **Coin Flip** | Heads or tails, ~`1.98×` on a win | `config/flip.json` |
| **The WTF Button** | One smash draws a weighted prize tier from a dud (`0×`) up to a legendary `1000×`, weights tuned to a 1% edge | `config/button.json` |
| **WTF Scratcher** | Scratch a 3×3 meme card; match three symbols for `2×`–`100×`, weighted to a configurable edge | `config/scratcher.json` |
| **Crash Royale** | Real-time **multiplayer PvP** crash: equal antes form a shared pot, one provably-fair curve, highest cash-out wins (bust = `0`); house takes a rake, the curve carries no edge | `config/arena.json` |

![The WTF Button — mounted arcade button with live jackpots, odds and prize table](./screenshots/button.png)

![WTF Scratcher — 3×3 provably-fair scratch card with live prize table](./screenshots/scratcher.png)

## Progressive jackpots (Mega Moolah-style)

- Three tiers — **Mini / Mega / Grand** — players opt into any subset from any game.
- Every game (including **Dice**) contributes a flat per-tier **ante** on top of the stake,
  independent of bet size (auto-refunded if a pot is temporarily unavailable). Any qualifying
  stake enters the pot.
- Each round has a hidden **must-win target** between the pot's seed and its **cap**.
  The bet whose contribution pushes the running total past the target **triggers the
  drop** (the pot always drops at or before the cap), but it does **not** auto-win —
  the winner is drawn **weighted by each eligible player's cumulative contribution**
  that round. This removes the expected-value edge of "sniping" a pot near its cap.
- **Anti advantage-play controls:** a minimum participation threshold per tier
  (`min_entry_usd`) gates who is eligible to win, and jackpot entry requires a credited
  real-money deposit (`jackpot_require_deposit`) to blunt Sybil/bonus farming.
- After a win the pot **reseeds** from an accrued **reserve** (funded by `reserve_pct`
  of every contribution) — so the house is never underwater.
- Fully provably fair: the pot publishes `SHA256(server_seed)` before the round and
  reveals the seed on the drop, so anyone can recompute the target.

## Platform features

- **Provably fair everywhere** — every outcome derives from
  `HMAC-SHA256(server_seed, "client_seed:nonce")`. Players rotate their seed to reveal
  the server seed and re-verify any past bet via `GET /api/<game>/verify/:id`.
- **On-site wallet** — unified USD-denominated balance; bets and jackpot contributions
  debit it; players withdraw anytime.
- **Automated crypto deposits** — NOWPayments hosted invoices; balance credited on the
  signed IPN webhook.
- **Automated withdrawals** — NOWPayments Mass Payouts (USDT BEP-20 / BSC, ~$0.02 fee) via a
  retrying payout worker with duplicate-credit protection.
- **Lucky Wheel** — one free provably-fair spin every 12h granting house-funded balance
  bonuses (never injected into player-funded pots).
- **Retention** — referral program, daily login streaks, leaderboards, admin-scheduled
  pot-boost events.
- **Accounts** — email/password and Wallet Connect (sign-in-with-Ethereum).
- **Play-for-fun mode** — a virtual balance and separate fun seed chains, isolated from
  real-money data.
- **Admin panel** at `/admin` — grouped dashboard (Overview / Games / Jackpots /
  Finance / People / System) with per-game stats & settings, house economics, per-game
  integrity checks (payout-error / duplicate-credit detection), a **Live activity**
  view of who's on the site right now, jackpots, draws, payouts, withdrawals, balances,
  users, wheel config, events, audit log, and CSV export.
- **Real-time UI** — Socket.io jackpot count-ups, a cross-game activity ticker, per-game
  canvas animations, and winner confetti.
- **SEO & PWA** — per-page meta/canonical/OG tags, JSON-LD structured data, sitemap,
  web manifest, and social share image.

![Admin dashboard — grouped navigation, headline KPIs and per-game P&L cards](./screenshots/admin-dashboard.png)

## Tech stack

| Layer | Technology |
|-------|------------|
| Runtime | Node.js **≥ 22.5** |
| Web / API | Express 4 |
| Real-time | Socket.io 4 |
| Database | **`node:sqlite`** (built-in) — WAL mode, **no native build step** |
| Crypto (EVM) | `viem` |
| Auth | `bcryptjs` password hashing, `express-session` (SQLite-backed store) |
| 2FA (payouts) | `otplib` (TOTP for NOWPayments Mass Payouts) |
| Frontend | Static HTML + CSS + vanilla JS (no framework, no build) |
| Payments | NOWPayments (deposits + Mass Payouts); CoinGecko (pricing) |

No database server, no Redis, no bundler/transpiler, and no native modules to compile.

## What's included

- Full application source (server, all game logic, admin panel, frontend).
- All configuration files (`config/*.json`) for games, jackpots, wheel, coins, wallet.
- Database schema + automatic migrations & seed (`database/`).
- Deployment assets: `Dockerfile`, `docker-compose.yml` (auto-HTTPS), systemd unit,
  nginx reverse-proxy config, one-command PowerShell deploy script, release builder.
- Operational scripts: DB reset, admin-password hasher, RTP/house audits, smoke tests.
- This documentation set.

## Licensing

The repository currently declares the **MIT** license in `package.json`. If you are
buying this as a product, agree the license/transfer terms explicitly with the seller
(e.g. exclusive/non-exclusive, resale rights). MIT is permissive and can be changed by
the copyright holder for future distribution.

## Positioning / why buy

- **You own it.** No game-provider revenue share, no per-seat SaaS fees.
- **Low ops footprint.** One process + one file DB; trivial to host and back up.
- **Auditable & fair.** Server-authoritative outcomes with public verification math.
- **Extensible.** Every game follows the same service/route/config/animation pattern —
  adding a ninth game is a well-trodden path (full architecture docs included with purchase).

> ⚠️ **Compliance.** Online gambling is regulated in most jurisdictions. Obtain legal
> review and any required licensing before operating with real money. 18+ only.
