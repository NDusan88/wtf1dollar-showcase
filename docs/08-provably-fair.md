# 8. Provably Fair — Verification

Every game and jackpot outcome is **server-authoritative** but **independently
verifiable**. The client never influences results; it only submits bet parameters.

## The core primitive

All game randomness derives from a keyed hash:

```
HMAC-SHA256( server_seed, "<client_seed>:<nonce>" )
```

- **server_seed** — a 32-byte secret generated per player per game seed chain. Only its
  `SHA256(server_seed)` hash is shown until you **rotate**.
- **client_seed** — player-settable; defaults to a random value you can change any time.
- **nonce** — a counter that increments on each bet, so every bet in a chain is unique.

Because you receive `SHA256(server_seed)` **before** betting, the operator cannot change
the server seed after the fact without breaking the hash — and once you rotate, the old
`server_seed` is revealed so you can recompute every past bet in that chain.

## Verifying a bet

1. Set (or note) your **client seed**.
2. Play some bets (each increments the nonce).
3. **Rotate** the server seed: `POST /api/dice/seed { "rotate": true }`. Seeds are a single
   shared per-user chain, so one rotation reveals the previous `server_seed` and starts a
   fresh chain for every game.
4. For any past bet, call `GET /api/<game>/verify/:id`. It returns the revealed
   `server_seed`, your `client_seed`, the `nonce`, and the recorded outcome.
5. Recompute locally and confirm it matches.

### Dice (worked example)

With revealed `server_seed`, `client_seed`, and `nonce`:

```js
const crypto = require('crypto');
const h = crypto.createHmac('sha256', server_seed)
                .update(`${client_seed}:${nonce}`)
                .digest('hex');
const result = (parseInt(h.slice(0, 8), 16) % 10000) / 100;   // 0.00–99.99
const win = direction === 'over' ? result > target : result < target;
```

The other games consume the same HMAC bytes and map them to their own result space:

- **Plinko** — the byte stream chooses left/right at each row to land the ball in a slot.
- **Mines** — the stream deterministically shuffles which cells are mines.
- **Crash** — the stream yields the bust multiplier for the round.
- **Coin Flip** — the first bits select heads/tails.
- **The WTF Button** — the stream draws a prize tier by cumulative weight.
- **WTF Scratcher** — the stream draws the card's symbols (and thus any 3-match prize) by weight.

Each game's `verify/:id` returns exactly the inputs needed to reproduce its result.

> **Dice odds note.** `result` is uniform over the 10,000 outcomes `0.00–99.99`. "Under `t`"
> wins on strictly-lower outcomes (exactly `t`%); "Over `t`" wins on strictly-higher outcomes,
> which is `(99.99 − t)%` — the boundary value `t` itself is a loss. The payout multiplier is
> priced off that true win chance so the RTP is exactly `(100 − edge)%` in both directions. The
> win/loss rule above (`result > target` / `result < target`) is unchanged, so verification is
> unaffected.

## Verifying a Crash Royale round

Crash Royale (the Arena) is one **shared** curve for the whole room, so — unlike the
per-player games — it uses **no client seed**. Each round has its own `server_seed`; the
round publishes `SHA256(server_seed)` while it is open for joins and reveals `server_seed`
at settlement. The crash point is:

```js
const crypto = require('crypto');
const hex = crypto.createHmac('sha256', server_seed).update(`arena:${round_id}`).digest('hex');
const u = parseInt(hex.slice(0, 13), 16) / Math.pow(2, 52);  // 52 bits -> uniform [0,1)
const crash = Math.min(max_multiplier, Math.max(1, Math.floor((1 / (1 - u)) * 100) / 100));
```

The curve `multiplier(t) = exp(growth_rate · seconds)` and every cash-out multiplier are
computed from the **server** clock, so the outcome is fully determined by `server_seed` and
`round_id`. Fetch it with `GET /api/arena/verify/:id`. The curve carries **no house edge** —
players compete against each other and the house takes a fixed rake from the pot.

## Verifying a jackpot drop

Each jackpot **round** has its own pot server seed. The pot publishes
`SHA256(pot_server_seed)` at the start of a round and reveals `pot_server_seed` when the
pot drops. The hidden **must-win target** is:

```
frac   = parseInt( HMAC_SHA256(pot_server_seed, "<round_number>:target").slice(0, 13), 16 ) / 2^52
target = round_start_usd + (threshold_usd - round_start_usd) * frac
```

- `round_start_usd` — the pot's value at the start of the round (the seed value).
- `threshold_usd` — the round cap (hard safety net).

A round **drops** on the bet whose contribution takes the running pot total from below
`target` to at or above it; the pot always drops at or before the cap. That crossing bet
only **triggers** the draw — the **winner is selected deterministically from the round's
eligible players, weighted by each player's cumulative contribution** (see
`draw.pickRoundWinner`, seeded by the same `server_seed`). Eligibility requires clearing the
tier's `min_entry_usd`; if no one has, the pool falls back to all contributors so the
must-drop guarantee always holds. Decoupling "who triggers" from "who wins" removes the
expected-value edge of sniping a nearly-full pot. The drop pays the whole accumulated pot
minus the house fee, then the pot **reseeds** to `seed_usd` from its reserve.

Verify a specific drop with `GET /api/draws/:id/verify` — it returns the revealed
`server_seed`, the round's `start` and `cap`, and the recomputed target. The winner is
reproducible from the revealed seed and the round's recorded contributions via the same
weighted-selection function.

## Why this is trustworthy

- **Pre-commitment:** the hash of the secret is published before play; the secret is
  revealed after — the operator can't retro-fit outcomes.
- **Determinism:** given the revealed inputs, anyone can recompute the exact result.
- **Separation:** per-player game seeds and per-round pot seeds are independent, so one
  reveal can't be used to predict another chain.
